---
author: Andrew
comments: true
date: 2012-04-28 08:46:48+00:00
layout: post
link: http://ajmccluskey.com/2012/04/xbmc-on-linux-mint-debian-edition/
slug: xbmc-on-linux-mint-debian-edition
title: XBMC on Linux Mint Debian Edition
wordpress_id: 398
categories: Guides
tags: debian htpc Linux linux mint lirc lmde lvm lvm2 xbmc xorg
---

Since I built my file server (now running FreeBSD after a stint with Mythbuntu) I've used a variety of front ends to view all of the juicy media stored on it.  My second most recent front-end was a re-purposed Lenovo Thinkpad Edge with a broken screen.  It ran Windows 7 with Windows MCE and a Microsoft MCE remote.  It was dead easy to setup, requiring only that I plugged in a HDMI cable and the IR receiver, and provided some media locations from the comfort of my couch.  Unfortunately, the terrible WiFi performance meant that video often lagged for brief periods of time, or at its worst, stopped playing altogether.  After a few months of frustration, I decided to try out Linux Mint Debian Edition to see if it performed better, given that all of the other WiFi connections in the house worked well.  It did.  So far I've been very happy with the whole system, so I thought I'd share how I got here.  A warning now: if you're not comfortable with basic Linux commands and editing configuration files by hand, then you should probably stick to Windows or Mac.  Having said that, attempting things beyond your current skill level is how you learn.  If you get really stuck, you can always post a comment and I'll see if I can help you out.

<!-- more -->



## The Technology


I chose Linux for a number of reasons; it's open, it's highly configurable, I'm familiar with it, and it's mainstream enough to have good media support without jumping through hoops.  Deciding on a distribution wasn't hard either.  I wanted something that was going to be stable and easy to use, and I liked the idea of rolling updates rather than big, sporadic version hops that had been one of my undoings with Mythbuntu.  Having installed Linux Mint on my laptop recently, I decided to give Linux Mint Debian Edition a go as it seemed to tick all of my boxes.  More specifically I ended up with the 201109 64-bit DVD with XFCE.  The 13" Thinkpad Edge isn't burdened with an optical drive, so I used the very helpful [Unetbootin](http://unetbootin.sourceforge.net/) to create a bootable USB stick.  If you're playing at home, an external optical drive should also work fine.

As I mentioned earlier, my media server had a stint running Mythbuntu, which is Ubuntu with MythTV pre-installed.  It worked well enough, but the configuration and architecture seemed a bit full on for my simple needs.  From memory, the fact that it supported TV tuner cards was a big plus for me, as the front end had a tuner card and I had dreams of recording my favourite shows on free-to-air.  The little Lenovo doesn't have one of those, so I opted for [XBMC](http://xbmc.org/) as the media centre software.  It has all of the features I need (subtitle and audio track control being two important ones) and has been far simpler to setup and maintain than MythTV.

Finally, as I need would like to be able to control everything from the couch, I chose the terribly useful [LIRC](http://www.lirc.org/) to enable my MCE remote.



## The LMDE Install: Adventures With LVM and a Less Than Helpful Udev


When it came to install time, I wasn't sure if the LMDE option would perform better or worse than the Windows solution I'd be using, so I didn't want to simply wipe the laptop's hard drive and start again.  With 3 of the maximum 4 primary partitions already used by Windows and Lenovo stuff, Logical Volume Management (LVM) seemed like the way to go.  The theory behind LVM, at least for my simple use of it, is straight forward; For the price of one primary partition, and some extra configuration, you have the option to create a large number of resizable logical partitions (or volumes, as the name suggests) that can be used just like any other partitions by Linux.

Having [the mistaken impression](http://askubuntu.com/questions/76095/what-is-the-use-of-boot-lvm-based-in-partitioning) that I couldn't have /boot on an LVM partition, I also backed up a "SYSTEM" partition of unknown origins and usefulness and replaced it with a boot partition.

For whatever reason, LMDE's installer does not support LVM, so I decided to follow [a tutorial](http://forums.linuxmint.com/viewtopic.php?f=141&t=71159) for the installation.  Rather than rehash the steps, I'll just cover the parts of the install that caused me issues.  Firstly, creating the logical volumes did not work straight away, as for some reason Udev didn't create the symbolic links from `/dev/<vg>/<lv>` to the actual devices.  I banged my head against a wall on this one for a while, and ended up booting into a Ubuntu live CD I had lying around and using that to create the logical volumes.  Once the volumes were created, I booted back into the LMDE DVD with hopes that the volumes would be detected and linked.  No dice.  At this point I discovered the magic command: `vgmknodes`.  This will create the links to the device nodes that are expected in the tutorial; that is, `/dev/<vg>/<lv>`.  Once this was done, I could proceed with the tutorial.

While I was searching for solutions to my LVM woes, I instructed the _lvcreate_ command to output verbose logging information.  This led me to the issue, and a possible workaround.  When `lvcreate` is creating a new logical volume, it will zero out the first KB of storage by default.  According to [the man page](http://linux.die.net/man/8/lvcreate), attempting to mount a logical volume that does not have the first KB of space zeroed out can hang the system.  It is when zeroing out this section of the volume that `lvcreate` attempts to access the logical volume using the missing link to `/dev/<vg>/<lv>`, and consequently fails.  Luckily, this default behaviour can be changed by using the `-Z` option.  From memory I did attempt this, and it worked, however I didn't like the sound of possible hangs, so I opted to create the volumes using a different live CD, and then manually create the device links as outlined above.  However, I believe it's possible to do the zeroing out manually, allowing you to complete the entire install process from LMDE.  If I had my time again, I'd probably try something like this.





  1. create the physical volume and volume group as usual


  2. create the logical volumes with the `-Z` option


  3. manually create the device links with `vgmknodes`


  4. manually zero out the first kilobyte of the logical volume with `dd`


  5. proceed with installation



At a command line, this would look something like the following.  While I've tested this out on an LMDE virtual machine, and it worked fine, it may not be a sane thing to do, so do not hold me accountable if this does bad things to your system.  As always, have a recent backup before playing with your drives.


    
    <code class="brush: plain">
    pvcreate <device>
    vgcreate vglmde <device>
    lvcreate -Z n -n swap -L 4GB vglmde
    lvcreate -Z n -n lmde -L 20GB vglmde
    vgmknodes
    dd if=/dev/zero of=/dev/vglmde/swap bs=1024 count=1
    dd if=/dev/zero of=/dev/vglmde/vglmde bs=1024 count=1
    </code>



The only other diversion I made from the instructions was to not disable automatic login.  Instead, I changed the TimedLogin and AutomaticLogin users to the new user I created for myself.  I figured if I had issues during boot, I could always edit the file from the live DVD again.



## The Media Centre and Remote


Now that I had a functioning OS, it was time to apply a couple of updates and get the software working.  To start with, I updated to [Update Pack 4](http://blog.linuxmint.com/?p=1949), which included updating apt's repositories.  When I tested out LMDE and XBMC in a virtual machine I had issues installing XBMC from the default repositories (incorrect versions of dependencies due to out of date linuxmint repository, and debian media repository), but these were avoided after installing update pack 4 and moving to the new repositories.

Once I was up to date, I proceeded to install XBMC and LIRC using Synaptic.  XBMC configuration is straight forward, and can be done from the couch with a remote or with a mouse and keyboard before deploying the machine to your lounge room.  LIRC took a little longer, but hopefully won't cause you too many issues.  Different remotes require different setups, so my settings may not apply, but there is some good documentation on LIRC's homepage, and plenty of guides to be found with a quick search.  I should point out, their [supported hardware page](http://www.lirc.org/remotes/) is a good place to start, and had exactly the lircd.conf I needed for my [model 1039 Microsoft MCE remote](http://www.mythtv.org/wiki/MCE_Remote).  In addition to installing the correct lircd.conf (copy it to `/etc/lirc`), I recall having to update `/etc/lirc/hardware.conf` with a driver and device setting, as lircd refused to start without a driver setting.


    
    <code class="brush: plain">
    DRIVER="default"
    ...
    DEVICE="/dev/lirc0"
    </code>



Once the two configuration files are in place, the lirc daemon should be restarted.


    
    <code class="brush: plain">
    /etc/init.d/lirc restart
    </code>



While the above seems straight forward, I had some issues when I was trying to get LIRC working.  I now believe they were caused by me forgetting to restart the lirc daemon after making configuration changes to lircd.conf and hardware.conf, but it was past 3am so I'm not really sure what happened.  In any event, I found [a post on the XBMC forums](http://forum.xbmc.org/showthread.php?tid=45972&pid=629937#pid629937) to be most helpful with debugging.  It takes you through checking that a signal is received, and then on to checking that it's being interpreted correctly.



## The Remote is Not a Keyboard


While I was searching for ways to configure LIRC, I stumbled across a [forum post](http://forums.linuxmint.com/viewtopic.php?f=197&t=91831) that mentioned configuring X (as in, the [X Window System](http://en.wikipedia.org/wiki/X_Window_System)) to ignore the remote, so it did not try to use it as a keyboard.  When I first started playing with the remote in XBMC I realised that my system was afflicted with this issue, as every time I pressed an arrow key, XBMC would jump two menu items; one from LIRC's input, and one from X's input.  As the post suggests, one needs to add an entry to the X configuration to ignore the remote.  However, the forum post mentions editing your `xorg.conf`, which is no longer created by default for new versions of X as explained in an [askubuntu.com post](http://askubuntu.com/questions/5314/where-is-etc-x11-xorg-conf).  Instead, The X server checks a few directories for configuration files that can contain sections that would have been found in xorg.conf.  As explained in [a different post on the Ubuntu forums](http://ubuntuforums.org/showpost.php?p=10800958&postcount=13), `/usr/share/X11/xorg.conf.d/11-evdev-quirks.conf` may be created with the following contents to ignore the remote.  This can also be added to a xorg.conf apparently, but I get the impression this is the preferred method, as X no longer creates a xorg.conf by default.


    
    <code class="brush: bash">
    Section "InputClass"
        Identifier "MCEUSB Remote"
        Driver "evdev"
        MatchProduct "Media Center Ed. eHome Infrared Remote Transceiver"
        Option "Ignore" "on"
    EndSection
    </code>



If you're not using the same remote as I am, you'll need to enter a different string on the MatchProduct line.  To find out what your product is, you can grep or manually search through `/var/log/Xorg.0.log` for lines that mention adding input devices, and look for one that looks like your remote.  The following might help.


    
    <code class="brush: bash">
    grep "Adding input device" /var/log/Xorg.0.log
    </code>



Based solely on my experience, the product name you're looking for is between the "Adding input device" string and the parentheses containing the device.



## The Home Straight


Once the remote was working, I was almost home.  Almost.  I don't think anyone wants to have to fumble around with a trackpad to start their media centre.  I know I don't; I want to hit the power button, get comfy on the couch, and then kick off XBMC with the remote.  To reach couch potato nirvana, I enlisted the help of [irexec](http://www.lirc.org/html/irexec.html).  `irexec` is a simple helper program that comes with LIRC, and it allows you to run arbitrary commands that are triggered by buttons on your remote.  There are 3 steps required to get `irexec` setup so that it will launch XBMC when the HOME button is pressed.




  
  1. Write a script to launch XBMC if it's not running

  
  2. Write a .lircrc config file that tells irexec to call your script when the HOME button is pressed

  
  3. Configure _irexec_ to start as a daemon whenever you login



I wrote the following script and placed it in `~/bin/start_xbmc.sh`.  It simply counts how many processes are running that have "xbmc" in their name, excluding the grep and the script itself, and starts XBMC if the count is 0.


    
    <code class="brush: bash">
    #!/bin/bash
    
    typeset -i xbmc_count=$(ps -ef | grep -Ev "(grep|"$0")" | grep -c "xbmc")
    
    if (( xbmc_count < 1 )); then
        xbmc &> /dev/null
    fi
    
    exit 0
    </code>



To configure irexec all I had to do was create `~/.lircrc`, the default configuration file for irexec, and add the following.  It tells irexec to call our `start_xbmc.sh` script when the HOME button is pressed.  By adding an ampersand to the end of the config option, `irexec` will not wait for our script to exit before moving on.


    
    <code class="brush: plain">
    begin
      prog = irexec
      button = HOME
      config = ~/bin/start_xbmc.sh &
    end
    </code>



Finally, to start irexec as a daemon whenever I login, I went to Menu -> Settings -> Sessions and Startup.  Under the "Application Autostart" tab, I added an entry for irexec where the command is `irexec -d`.  The `-d` option causes `irexec` to run as a daemon.  Now whenever I login, `irexec` is started as a deamon, ready to start XBMC as soon as I hit the HOME button.

That's it.  Everything you should need to know to get a basic home media centre working on Linux Mint Debian Edition, complete with LVM workarounds.  If you have issues, comments, or suggestions, then please post a comment.
