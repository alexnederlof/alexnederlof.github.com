---
layout: single
title: "Installing SSD in my 27\" iMac"
date: 2011-08-28 13:16
comments: true
categories: [ Hardware, Tech]
---
I finally decided to install a SSD in my 27″ iMac. The iMac is a late 2009 model and it’s very fast. However, most of the time it’s just waiting for the hard disk and not using it’s full potential. After experiencing the speed of a SSD on my MacBook Air I was convinced my iMac should have that power as well.

I’m typing this with the new SSD installed and everything is working blazingly fast! Boot-up time went from 92 seconds to 21. All apps start so fast you can’t even see the loading screen. I love this speed!
<!--more-->
## So how did I do it?
First of all I found these three great articles: [one from Tobias Müller](http://www.twam.info/hardware/apple/installing-additional-ssd-in-mid-2010-27-imac), [one from ChargedPc](http://blog.chargedpc.com/2011/05/2011-imac-ssd-install-guide.html) and [one from ssd.youds.com](http://ssd.youds.com/). These articles were a great start.

My choice of SSD was [the Vertex 3 240GB](http://www.ocztechnology.com/ocz-agility-3-sata-iii-2-5-ssd.html). It doesn’t matter much which one you choose, as long as you pick a Sandforce controller. I found out the my iMac has a SATA 2 controller, which offers a 3 GB/s throughput, and the iMacs after my model all have SATA 3, offering 6 GB/s throughput. I couldn’t find any benchmarks telling me what the performance loss of running a SATA 3 disk on a SATA 2 cable was so I decided to just go with a SATA-3 disk, since that offered a 550 MB/s read/write  speed which is still less then 3 GB/s.

For the 2009 iMac model, you need a SATA Power Y-Splitter and a Left Angled SATA cable.

Before starting the disassembly I plugged the SSD into my Ubuntu server to check if the SSD’s firmware was up-to-date. This is impossible to do on your mac. The firmware shipped was up-to-date so I had nothing to worry about.

For the disassembly I followed the guide from ssd.youds.com. I noticed some differences because he is using the 2010 model. Some connectors are different and the not all screws are in the same place. Tip for disassembling your iMac: find a teardown blog for your specific model! I’m not going to place pictures here of my disassembly because there are enough tear-down galleries as it is.

When the mac was disassembled it turned out my pre-investigation wasn’t as thorough as it should have been. The late 2009 model only has two SATA ports: one for the optical drive and one for the hard drive. I expected three. Since SATA doesn’t support bus-sharing I had to choose to either replace my harddisk or my superdrive. I chose to remove my superdrive since I never use it anyway and my iMac can boot from USB.

![SSD Holder](/assets/images/ssd/SSD-holder-back.jpeg)Removing the optical drive isn’t trivial. The drive itself also functions as the cover of a shaft that transports air from the fan on side to the cooler on the other side. This is the reason why iMacs are so quiet. Their construction is extremely smart. Because the SSD is soo much smaller I had to construct a placeholder for my SSD. Here’s how it looks:



You have to remove the optical driver and it’s cable. Then you split the power from the hard drive to use for your SSD and you use the SATA port of the optical drive for the SSD. The SATA cable I had was way too long unfortunately so I suggest getting a short one if you plan on doing it yourself.

After re-assembling my iMac I found out my microphone wasn’t working so I had to open it up again, plug in the mic and assemble it yet again. Remind yourself to make pictures of every wire you unplug if you don’t want this to happen to you! Also remember that every socket on the motherboard needs a cable, and all sockets are labeled. After booting your mac check that the camera works, microphone works, fan sensors work, etc.

When I started up I wanted to use the Lion’s harware test by holding the ‘D’ key. However, this didn’t work for some reason, the Mac just didn’t respond. When I released the ‘D’ key it booted up normally. I could see the SSD configured in the Disk utility and formatted it to Max OS extended journaled. According to the sensors in the iMac the temperature of the SDD was 128 degrees Celsius. I read this on other fora. It has something to do with the way Apple wires it’s hard drives. It doesn’t really seem to matter so I ignored it. If I figure out what I’m going to do with it I will update this post.

## Moving the data.

My SSD is still significantly smaller then the HD I had. This means I had to move everything except from some big media folders from the HD to the SSD. As always make sure you have a back-up before going any further! I used [Carbon copy](http://www.bombich.com/) to move all data excluding my Pictures, Movies and Downloads folder. After that I changed the start-up disk (in System preferences) to the SSD and rebooted the computer. After the blazingly fast boot I checked if everything was still working. Then I renamed the “Users” folder on my hard drive to “Extended Homedirs”. I removed everything but the Pictures, Movies and Downloads folders from old home folder. Then I deleted everything else from that hard drive except for the “Extended Homedirs” folder. In my new home folder on the SSD I saw that Lion had already created a new Movies, Downloads and Pictures folder. I deleted those and replaced them with Aliases to the “Extended Homedirs” subfolders.

That’s it!

![SSD installed](/assets/images/ssd/SSD-installed.jpeg)