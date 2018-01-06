---
layout: post
title: Grafana Update - IBM x3650 M2 on IPMI
date: 2018-01-05 11:19:40 +0000
tags:
- homelab
- dashboard
- IBM
- '3650'
- IPMI
categories:
- homelab
---
Minor update following on from my previous post ([here](https://blog.misaunders.com/2018/homelab-dashboard-with-grafana-influxdb-telegraf/ "Grafana")), I've changed the IBM input to use IPMI over LAN, and unlocked a few more details.

Install a small tool from your linux distribution called IPMITool. On ubuntu, that's:

    sudo apt-get install ipmitool

Then you can use this to raw query the IPMI status from your devices. For the IBM, this looks like:

    ipmitool -H 192.168.1.4 -U USERID -P PASSW0RD -I lanplus sdr

Note the default username and password, change those to match the credentials in your IBM IMM. From what I can tell, both LAN and LANPLUS work here, however the Dell iDrac only supports LANPLUS. Doesn't seem to make any difference here. 

What you'll get back, is an output of the available readings. This will show what you'll be able to work with in Grafana later. If you can see a reading here, you'll be able to work with it. 

My server outputs the following:

    Ambient Temp     | 30 degrees C      | ok
    Altitude         | 280 feet          | ok
    Avg Power        | 90 Watts          | ok
    Planar 3.3V      | 3.29 Volts        | ok
    Planar 5V        | 4.90 Volts        | ok
    Planar 12V       | 11.99 Volts       | ok
    Planar VBAT      | 3.01 Volts        | ok
    Fan 1A Tach      | 3248 RPM          | ok
    Fan 1B Tach      | 2375 RPM          | ok
    Fan 2A Tach      | 2697 RPM          | ok
    Fan 2B Tach      | 1850 RPM          | ok
    Fan 3A Tach      | 3567 RPM          | ok
    Fan 3B Tach      | 2700 RPM          | ok
    Fan 1            | 0x00              | ok
    Fan 2            | 0x00              | ok
    Fan 3            | 0x00              | ok
    Front Panel      | 0x00              | ok
    Video USB        | 0x00              | ok
    DASD Backplane 1 | 0x00              | ok
    SAS Riser        | 0x00              | ok
    PCI Riser 1      | 0x00              | ok
    PCI Riser 2      | 0x00              | ok
    CPU 1            | 0x00              | ok
    CPU 2            | 0x00              | ok
    All CPUs         | 0x00              | ok
    One of The CPUs  | 0x00              | ok
    IOH Temp Status  | 0x00              | ok
    CPU 1 OverTemp   | 0x00              | ok
    CPU 2 OverTemp   | 0x00              | ok
    CPU Fault Reboot | 0x00              | ok
    Aux Log          | 0x00              | ok
    NMI State        | 0x00              | ok
    ABR Status       | 0x00              | ok
    Firmware Error   | 0x00              | ok
    PCIs             | 0x00              | ok
    CPUs             | 0x00              | ok
    DIMMs            | 0x00              | ok
    Sys Board Fault  | 0x00              | ok
    Power Supply 1   | 0x00              | ok
    Power Supply 2   | 0x00              | ok
    PS 1 Fan Fault   | 0x00              | ok
    PS 2 Fan Fault   | 0x00              | ok
    VT Fault         | 0x00              | ok
    Pwr Rail A Fault | 0x00              | ok
    Pwr Rail B Fault | 0x00              | ok
    Pwr Rail C Fault | 0x00              | ok
    Pwr Rail D Fault | 0x00              | ok
    Pwr Rail E Fault | 0x00              | ok
    PS 1 Therm Fault | 0x00              | ok
    PS 2 Therm Fault | 0x00              | ok
    PS1 12V OV Fault | 0x00              | ok
    PS2 12V OV Fault | 0x00              | ok
    PS1 12V UV Fault | 0x00              | ok
    PS2 12V UV Fault | 0x00              | ok
    PS1 12V OC Fault | 0x00              | ok
    PS2 12V OC Fault | 0x00              | ok
    PS 1 VCO Fault   | 0x00              | ok
    PS 2 VCO Fault   | 0x00              | ok
    Power Unit       | 0x00              | ok
    Cooling Zone 1   | 0x00              | ok
    Cooling Zone 2   | 0x00              | ok
    Cooling Zone 3   | 0x00              | ok
    Drive 0          | 0x00              | ok
    Drive 1          | 0x00              | ok
    Drive 2          | 0x00              | ok
    Drive 3          | 0x00              | ok
    Drive 4          | 0x00              | ok
    Drive 5          | 0x00              | ok
    Drive 6          | 0x00              | ok
    Drive 7          | 0x00              | ok
    Drive 8          | 0x00              | ok
    Drive 9          | 0x00              | ok
    Drive 10         | 0x00              | ok
    Drive 11         | 0x00              | ok
    All DIMMS        | 0x00              | ok
    One of the DIMMs | 0x00              | ok
    DIMM 1           | 0x00              | ok
    DIMM 2           | 0x00              | ok
    DIMM 3           | 0x00              | ok
    DIMM 4           | 0x00              | ok
    DIMM 5           | 0x00              | ok
    DIMM 6           | 0x00              | ok
    DIMM 7           | 0x00              | ok
    DIMM 8           | 0x00              | ok
    DIMM 9           | 0x00              | ok
    DIMM 10          | 0x00              | ok
    DIMM 11          | 0x00              | ok
    DIMM 12          | 0x00              | ok
    DIMM 13          | 0x00              | ok
    DIMM 14          | 0x00              | ok
    DIMM 15          | 0x00              | ok
    DIMM 16          | 0x00              | ok
    DIMM 1 Temp      | 0x00              | ok
    DIMM 2 Temp      | 0x00              | ok
    DIMM 3 Temp      | 0x00              | ok
    DIMM 4 Temp      | 0x00              | ok
    DIMM 5 Temp      | 0x00              | ok
    DIMM 6 Temp      | 0x00              | ok
    DIMM 7 Temp      | 0x00              | ok
    DIMM 8 Temp      | 0x00              | ok
    DIMM 9 Temp      | 0x00              | ok
    DIMM 10 Temp     | 0x00              | ok
    DIMM 11 Temp     | 0x00              | ok
    DIMM 12 Temp     | 0x00              | ok
    DIMM 13 Temp     | 0x00              | ok
    DIMM 14 Temp     | 0x00              | ok
    DIMM 15 Temp     | 0x00              | ok
    DIMM 16 Temp     | 0x00              | ok
    PCI 1            | 0x00              | ok
    PCI 2            | 0x00              | ok
    PCI 3            | 0x00              | ok
    PCI 4            | 0x00              | ok
    All PCI Error    | 0x00              | ok
    One of PCI Error | 0x00              | ok
    IPMI Watchdog    | 0x00              | ok
    Host Power       | 0x00              | ok
    DASD Backplane 2 | 0x00              | ok
    DASD Backplane 3 | 0x00              | ok
    Backup Memory    | 0x00              | ok
    Progress         | 0x00              | ok
    Planar Fault     | 0x00              | ok
    SEL Fullness     | 0x00              | ok
    PCI 5            | 0x00              | ok
    OS RealTime Mod  | 0x00              | ok
    

You can see here there are a lot of disabled, or unreadable sensors. This means this specific server and module doesn't support this feature. 

On the IBM IMM, a lot of others are reported as just "ok" state, with a 0x00 reading. This means that you could potentially make a SingleStat panel to report a red/green status for that sensor if you wanted, but you wouldn't have anything graphable. 

We can see however 2 extra things I couldn't do with the raw SNMP data. We now get usable RPM data from the fans, but also interestingly, power level! 

So how do we get that into Influx via Telegraf? Add this server to the conf file. Telegraf's configuration lives in /etc/telegraf/telegraf.conf

I already have my R710's iDrac in there from the previous post, so adding the IBM to it looks like:

    [[inputs.ipmi_sensor]]
      servers = ["root:calvin@lanplus(192.168.1.10)","USERID:PASSW0RD@lanplus(192.168.1.4)"]
      interval = "30s"
      timeout = "20s"

Multiple entries are quoted, and separated by commas. Each one has it's own set of credentials and IP address. You can monitor as many servers as needed this way, and each can be its own set of details and protocol. 

So from the previous post, I already had voltage and ambient temperature graphed. Switching these to use the IPMI data was easy. Then I was able to add 2 new graphs, for fan speed, and power consumption in watts. 

![](/uploads/2018/01/06/Screen Shot 2018-01-06 at 11.30.42 am.png)

The fans are easy to map multiple into a single graph, as we can select them by their unit type.

![](/uploads/2018/01/06/Screen Shot 2018-01-06 at 11.32.12 am.png)

By setting the FROM clause to specify the server (192.168.1.4 is the IBM server), and unit = rpm, we will get data from all fans at once in a single query. Set group by to include Tag(name) to make them show as separate lines. Due to the way IPMI data works, set fill(previous) to get connected lines, and not just points.

You can use $tag_name in the alias section to make cleaner names for the lines & legends. 

I've chosen to turn off the legend completely on these graphs, since they're fairly self explanatory, and individual fan names are really needed for specific reporting. 

I hope this helps out anyone that's working with an IBM server. In this case it's related to the x3650 M2, but my understanding is that the IMM systems that IBM used didn't change much in concept across their series, so this should be applicable to a wider range of servers.
