---
layout: post
title: Homelab Dashboard with Grafana, InfluxDB & Telefrag
date: 2018-01-04 18:39:28 +0000
---
So it's been a while since I've had a chance to write anything here, but since it's the holiday break, and I'm off work for a while, it's been time to add new hardware to the rack, take care of things that have been needing to be done for a while, and really clean and tune things up. 

One of the key things I wanted to do, was build up a proper graphing/monitoring setup for my setup. I built one for work a few years back, so I had the basics down, but wanted to see what was new too. 

So I won't go into detail on installing the packages, the 3 main components here are [Grafana](http://docs.grafana.org/installation/ "Grafana Installation"), [InfluxDB](https://docs.influxdata.com/influxdb/v1.4/introduction/installation/ "InfluxDB Installation") & [telegraf](https://github.com/influxdata/telegraf "Telegraf"). All of these are free, and open source, and easily available, and easy to setup. I'm running mine on a Ubuntu VM, setup solely for this purpose. So just a basic Ubuntu 16 server install on a VMWare VM, from ISO, and basic network setup, nothing fancy, then install the above 3 packages. 

So, what does the dashboard look like?

![](/uploads/2018/01/04/Screen Shot 2018-01-04 at 7.56.34 pm.png)So, the hardware we're working with here consists of:

Dell R710, with iDrac 6, running ESXi.

IBM x3650 M2, running FreeNAS

Cisco 2921 Router

We're showing:

WAN Input/Output, live data, and historical graph data. 

We're gathering all interface data from the Cisco router (2921 in this case, but that's to the way Cisco's IOS works, this can be applied to pretty much any router, or switch)

We use Telegraph to get this from the Cisco via SNMP. 

    [[inputs.snmp]]
      agents = [ "192.168.1.254:161" ]
      version = 2
      community = "public"
      name = "snmp"
    
     [[inputs.snmp.field]]
        name = "hostname"
        oid = "RFC1213-MIB::sysName.0"
        is_tag = true
    
      [[inputs.snmp.table]]
        name = "snmp"
        inherit_tags = [ "hostname" ]
        oid = "IF-MIB::ifXTable"
    
        [[inputs.snmp.table.field]]
          name = "ifName"
          oid = "IF-MIB::ifName"
          is_tag = true
    

So this stores the hostname of the router as the tag "name", and puts it in the default Telegraf database in Influx. Because the Cisco puts out effectively tabular data for the interfaces, importing it is very easy; Telegraf adapts it directly to tables in Influx to make gathering a lot of data with only a single poll straight-forward. 

    SELECT derivative(mean("ifHCInOctets"), 1s) *8 FROM "snmp" WHERE ("agent_host" = '192.168.1.254' AND "ifName" = 'Gi0/0') AND $timeFilter GROUP BY time($__interval) fill(null)

This is the query needed in Grafana to display that data, so the upper left Single Stat panel showing the WAN inbound. We use a math(\*8) to take the data from bits to bytes. ifHCInOctets is the value/reading that we have, and we're using the WHERE clause to filter it to a specific interface on the router. In this case, it's Gi0/0. In some cases this may be a specifically named interface, depending on what your broadband uplink actually consists of. 

Repeat the above for the OutOctets. (You can duplicate panels in Grafana once you've got one setup nicely, then just edit the query to the different value to save some time)

We use an identical query for the WAN historical graphs. I actually made 2 queries, so that IN and OUT can be considered separate graphs, then you can push the OUT traffic to the right axis if you wish, and also apply a negative transform. Some people I've seen use a negative value in the math (\* -8 instead of \* 8), however this makes your values in the legend negative as well, which bothers me so using negative transform you get it to do up/down graphs, but still show proper values in the table.  

![](/uploads/2018/01/04/Screen Shot 2018-01-04 at 8.09.04 pm.png)![](/uploads/2018/01/04/Screen Shot 2018-01-04 at 8.08.31 pm.png)![](/uploads/2018/01/04/Screen Shot 2018-01-04 at 8.08.49 pm.png)![](/uploads/2018/01/04/Screen Shot 2018-01-04 at 8.08.57 pm.png)