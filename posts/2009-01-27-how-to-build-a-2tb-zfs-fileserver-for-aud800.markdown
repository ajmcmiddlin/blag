---
author: Andrew
comments: true
date: 2009-01-27 12:07:35+00:00
layout: post
link: http://ajmccluskey.com/2009/01/how-to-build-a-2tb-zfs-fileserver-for-aud800/
slug: how-to-build-a-2tb-zfs-fileserver-for-aud800
title: How to Build a 2TB ZFS Fileserver for AUD800
wordpress_id: 121
categories: Guides
tags: fileserver solaris unix zfs
---

My aspirations to build a [Solaris](http://www.sun.com/software/solaris/) fileserver have finally been realised and I now have 2TB of networked storage sitting beneath my desk as I type this.  Now that the server is built and working, I figured it was time to share my experiences with the world.



## The Requirements


Put simply, I wanted a fileserver to host all of my music, movies and other files in a central location that was networked with the rest of my house.  Not only did I want to store my files, but I wanted around 2TB of actual storage so that it would last me a while, as well as some sort of protection against an inevitable drive failure.  Unless you want to run some sort of backup software, which I don't, then RAID is generally how you protect against drive failures.  So RAID it was.

After some research, I decided that a dedicated box with RAID was definitely going to meet my needs the best.  I also decided that RAID 1 (a simple mirror) was inefficient as it sacrifices too much drive space.  As cheap as storage is these days, I'm still a poor uni student.  So, something like RAID 5 is what I wanted.  Once this was decided, it was time to look at filesystems.  It didn't take long to see that Sun's ZFS with raidz was probably the way to go.  This made the choice of operating system simple, as ZFS means Solaris unless you want to run your filesystem in userspace.  To be honest, I'm not sure how much this would impact performance, but a chance to get acquainted with a new OS was too good to pass up, so Solaris it was.
<!-- more -->



## The Hardware


Now that I knew what OS I was running, I then had to decide on the hardware.  The biggest requirement I had for hardware, which I neglected to include above, is price.  I wanted this all to be as cheap as possible.  Easier said than done, as hardware support on Solaris seems much more limited than it does with Linux.  Given that this is all for a headless fileserver, the only tricky hardware requirement was the hard disk controller.  Basically I needed something that was supported by Solaris and had at least 4 SATA ports.  Luckily, there is a [hardware compatibility list](http://www.sun.com/bigadmin/hcl/data/os/) (HCL) for Solaris available on Sun's website.  While I couldn't be sure from the HCL, I was fairly confident that I'd found hardware that would do the job.  Here's what I ended up buying, which I can now confirm, works fine with Open Solaris 2008.11.




	
  * **Asus M2N68-CM Motherboard** - This motherboard was really cheap, had a hard disk controller that appeared to work with Solaris, and included onboard video.  As AMD CPUs appear to be a bit cheaper at my local parts store, this really helped the price requirement.

	
  * **AMD Athlon 64 5000+ CPU** - The cheapest I could find for this mobo with the added bonus of being a 64-bit dual core

	
  * **Corsair 2GB (2x 1GB Kit) Value Select PC-5300** - again, the cheapest 2GB RAM I could find from a known brand

	
  * **Antec NSK4480B** - A mini tower that included enough drive bays and a 380 watt Antec PSU

	
  * **4x Western Digital 750G SATAII Green Power** - raidz allows for the failure of 1 drive in the array, so this configuration gives me 2TB of storage and keeps the costs down

	
  * **Old Seagate 120GB IDE hard disk** - for the OS

	
  * **Old Sony DVD±RW IDE optical drive** - to install the OS





## The OS


OpenSolaris 2008.11 is the flavour of Solaris I went with.  To be honest I didn't research this too heavily, it just seemed like the way to go as it's aimed at developers and students.  Before I installed any OS though, I enabled AHCI support in the BIOS on the mobo, as this apparently improves drive performance according to a forum post I read.  At the end of the day performance isn't a huge issue for me, but if it is for you, you might want to investigate this further.

Once I'd totally h4x0red the BIOS, I booted the OpenSolaris disc I downloaded via bittorrent.  The CD boots you to a live environment that you can play with before installing.  It also includes the Device Driver Utility (Applications -> System Tools on the Gnome menu) which checks for hardware compatibility.  The only thing that didn't work out of the box was the motherboard's onboard ethernet.  Following the advice on the HCL, I downloaded a [third party driver](http://homepage2.nifty.com/mrym3/taiyodo/eng/) (you want the nfo driver) on another PC and followed the instructions to install.  This worked without issue and gave me gigabit ethernet.



## The Filesystem


Now that I had an OS installed, it was time to setup the raidz pool and ZFS filesystems.  There are a lot of guides and how-tos for this, but the most comprehensive I found was [Sun's _ZFS Administration Guide_](http://docs.sun.com/app/docs/doc/819-5461).  While this is a fairly lengthy document, it's probably worth skimming if you're new to ZFS.  Especially the sections on setting up pools and filesystems, as well as the sections on maintenance.  To actually setup your raidz pool takes one command.  Each additional filesystem is one more command.  Yes, it is THAT easy.  For those who don't want to go reading through documents, these are the commands:


    
    <code class="brush: plain">
    # zpool create myfirstpool raidz c3t0d0 c3t1d0 c3t2d0 c3t3d0
    # zfs create stor1/mysecondfilesystem
    </code>



The first command creates a raidz storage pool called _myfirstpool_ using the four hard disks specified.  In order to find the device IDs of your disks you can once again use the Device Driver Utility.  Fire it up, right-click the AHCI controller under the storage heading and select show details.  Scroll to the bottom of the window that pops up and you should see four DISK headings.  Under each one of these is a path to the device on the filesystem.  What you're after is the filename at the end of the path, minus everything from the 's' onwards.  For example, my first device path reads /dev/rdsk/c3t0d0s2.  From that I only want the c3t0d0 at the end to identify the disk as part of the zpool command.  Once you've executed the command, you'll have a new raidz pool using the disks specified, as well as a ZFS filesystem spanning the pool.  Furthermore, the zpool command will automatically mount the new filesystem and make sure that it is mounted whenever the machine is booted.

The second command creates another ZFS filesystem on your storage pool and sets up mountpoints the same as the zpool command does.  This allows you to segregate your data and do more advanced things like compression, quotas, and coming soon - encryption.  I haven't delved into the advanced configurations of ZFS filesystems, as they are unnecessary for my needs.  I'm going with ZFS because it's so easy to administer and offers the best protection of my data in a RAID5-like setup, not for any of the other advertised benefits.



## SAMBA (Sharing is Caring)


Put simply, SAMBA is the open source implementation of Windows file and printer sharing that operating systems other than Windows use if they want to share with Windows clients.  As I have a mixture of Windows and Mac clients in the house, SAMBA is how I've chosen to share files.  It allows reasonably fine grained security controls, works with any client OS I care to use, and allows me to easily mount shares as a network drive on my Windows clients.  From memory the SAMBA server is not installed by default on OpenSolaris.  To install it, simply fire up Package Manager, search for samba and then install the _SUNWsmba_ package.

There is a vast range of configuration options for SAMBA (half of which I don't fully understand), so I will not go into a detailed explanation of configuring SAMBA.  Once SAMBA is installed, you can setup basic shares with the included GUI under System -> Administration -> Shared Folders.  If this doesn't offer you enough control, you can hack up the configuration file by hand.  The configuration file used by SAMBA is located at `/etc/sfw/smb.conf`.  If this file does not exist, you can create a new one, or copy across the example file from `/etc/sfw/smb.conf-example`.  The example file gives you a glimpse at some, but certainly not all of the options, as well as a brief explanation of some of them.  Once you've written the configuration file and you're ready to test it out, you can run the following command as root to restart the SAMBA server.


    
    <code class="brush: plain">
    # svcadm restart samba
    </code>





## It's Overrrrr


Well that concludes my little tutorial.  If I've left anything out or you have questions, feel free to contact me or post a comment.  Until then, happy fileserving!

