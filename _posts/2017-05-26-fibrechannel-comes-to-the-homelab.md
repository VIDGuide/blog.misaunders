---
layout: post
title: FibreChannel Comes to the HomeLab!
date: 2017-05-26 00:00:00 +0000
categories:
- homelab
tags:
- homelab
- storage
- FibreChannel
- FreeNAS
---


So, I've decided it was time to upgrade my home-lab storage. I previously had just a single host, a Dell R710, with 4x 2TB drive on-board via a H700 RAID card, running ESXi. I had a FreeNAS setup to provide media storage, but it just had single volumes sitting on a VMDK. Not ideal, not a good idea, but it worked. Until I started to run out of space.

So, I could have gotten bigger disks. But where is the fun in that? The H700 can support large disks, I could put them into ESXi, but then I've still got a FreeNAS setup doing virtually nothing, and only a single host. Where's the fun in that?

Since my home-lab isn't just about media storage, it's also a learning tool, to help keep me sharp in my day job, I've been wanting to work towards a multi-host setup, with shared storage. HA and fail-over are important things to learn about.

So I also decided to learn about some older technologies too. FibreChannel was already superseded by the time my career progressed to the point of needing TB of shareable storage, I jumped right into SAS territory.

I found a NetApp DS14MK4 online fairly cheaply, which came with 7x 300GB FC disks. These are 10,000rpm disks, but in a ZFS ZRAID2, this gave me about 1.2TB usable space, so not nothing. But I needed more.

My first plan was to pick up 7 more caddies with SATA inter-posers. Found some online fairly cheaply, then my plan was to re-use the 2TB disks I already had in the R710, and migrate those plus a couple of extra's into the NetApp shelf. Problem was, when they turned up, they were for a MK1 shelf, and not compatible with the MK4. Shame!

Then I found a second shelf! These things can stack 6-7 together into a single storage chain, so I figured why not, power bill be damned! This other one I was able to bargain out a decent price, and it included 14x 450GB disks. Now this was a little more like it. This gave me 3.5TB usable space on top of the 1.2TB I already had. Then I picked up 7 more 300GB disks, adding another 1.2TB and filling the shelves. So roughly 6TB of ZRAID2 protected disks, made up of 2 separate 7x 300 disk sets (one for ESXi, one for my coming-soon security cameras), and a 2x7x450GB pool for my media.

But wait, I wanted shared storage. Can't (easily) do FC from inside the R710 for that purpose.. (Yes, I realize I could do PCI pass-through of the FC card, but not ideal) -- So I picked up a cheap IBM x3650 M2 server. 32GB RAM, single CPU, no disks. So now I have a dedicated storage server.

Fitted this with a QLogic QLE2462 for multi-path support and installed FreeNAS. Aside from this IBM being *horridly slow* through the BIOS, this turned out to be a breeze and nothing special really. The only catch/trick was to do with getting FreeNAS to boot properly. The trick: Set the first boot device to be "Legacy Only" -- Seems FreeNAS has some issues with IBM's EFI implementation on this server.

<iframe width="100%" height="auto" src="https://www.youtube.com/embed/KdNx6wFoBbA" frameborder="0" allowfullscreen="" async="" preload=""></iframe>

Doesn't everybody want to see the lights blink? Here's a few seconds of the formatting in progress!

Yes, the fans sound very loud in that video! One of the PSU's in the second shelf popped almost as soon as I fired it up. The seller sent a new one, but it took a few days to arrive, and sadly, while it was on just one PSU, max fans ahoy!

**So my net result is:**

R710 running ESXi now with no local disks. 14 300GB FC disks, and 14 450GB FC disks. All disks with multi-path redundancy, all shelves with dual PSU's. Backed by Dual UPS's & rails. IBM x3650 M2 server with Dual PSU's running the FreeNAS entirely, with Sonarr, CouchPotato and Transmission in jails.

Storage presented to the ESXi box over NFS, and all VM's are running on there now.

**Future plans**: properly VLAN out the network, to move storage traffic to it's own network. I also want to move to iSCSI for the ESXi data stores. Eventually one day I'd like to add a small 1RU server to use as a second ESXi host to get to play with the shared storage and HA features, but at least I've got a foundation now.

What does this cost to run? Well, the R710, UPS's and switch previously  sat at around 400W according to both a Wattmeter, and the UPS's LCD readouts. Now with the NetApp shelves added, and the IBM server, the total consumption is around 900W. The R710 still has the 2TB disks spinning, which will be consuming a little power, but probably not much in comparison with the 28 spindles in the NetApps.

Will I keep it forever? Probably not, but it's been fun to play with, and I do love putting old enterprise gear to work!

And finally some pictures of the setup rack.

![](/uploads/2017/05/27/IMG_2446-1.JPG)

![](/uploads/2017/05/27/IMG_2447.JPG)

![](/uploads/2017/05/27/IMG_2451.JPG)

![](/uploads/2017/05/27/IMG_2445.JPG)

![](/uploads/2017/05/27/IMG_2450.JPG)

![](/uploads/2017/05/27/IMG_2449.JPG)

![](/uploads/2017/05/27/IMG_2448.JPG)

(The TippingPoint x505 and few other patch panels aren't used for anything, they was just some free pickups so they sits in the rack :)

