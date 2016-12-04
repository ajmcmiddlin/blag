---
author: Andrew
comments: true
date: 2009-02-17 05:43:13+00:00
layout: post
link: http://ajmccluskey.com/2009/02/more-fun-with-opensolaris/
slug: more-fun-with-opensolaris
title: More Fun with OpenSolaris
wordpress_id: 158
categories: Guides
tags: fileserver solaris vnc
---

Partly due to laziness, and partly due to me still working out the kinks, I never included this stuff in my [original post on setting up an OpenSolaris fileserver](http://www.ajmccluskey.com/2009/01/how-to-build-a-2tb-zfs-fileserver-for-aud800/).



## VNC


In order to save physical space and make your server more accessible, it can be a good idea to setup a VNC server so you can control it remotely without having to have a monitor, keyboard and mouse plugged in.  This is a fairly straight forward procedure and I hope to outline it for you now.

First of all, you're going to need the software.  This can be obtained from [sunfreeware.com](http://www.sunfreeware.com/programlistintel10.html).  Simply search for x11vnc and download the binary package, which at the time of writing is [x11vnc-0.7-sol10-intel-local.gz](ftp://ftp.sunfreeware.com/pub/freeware/intel/10/x11vnc-0.7-sol10-intel-local.gz).  There is a note there on the dependencies that must be installed for this to work, however I believe these were already installed for me, but your mileage may vary.  Once you've downloaded the gzip file, you should be able to extract the contents by simply double clicking the downloaded file and then dragging the contents somewhere (e.g. the desktop).  Now, RTFM.  It's only two lines, so I'll repeat it here for you =P
<!-- more -->


    
    <code class="brush: plain">
    install the package as root with:
    
    # pkgadd -d <package name>
    </code>



In our case, replace `<package name>` with the name of the only other file in the gzip package.  Remember to sign in as root first (use the `su` command and then type the root password when prompted).  When prompted about which packages to install, you can just hit enter, which will install all packages by default.  Once the package is installed, you should have a new binary: `/usr/local/bin/x11vnc`.  If you'd like to read the help, you can do so with `/usr/local/bin/x11vnc -h | less`.  Piping the help through `less` will allow you to scroll up and down using the arrow keys.  You can also search the help by typing a forward slash character, followed by your search term.  If the pattern is found, you can move to the next occurrence of that pattern by pressing n.


After scanning the help, I've come up with a little script to start the server with the options I like.  These options keep the server running untill it is manually taken down, as the default behaviour is to kill the server after the first user disconnects.  It also allows connections from your internal network only and runs in the background so that you can close the terminal window after running the script without the server closing also.  Finally, the script will kill any instances of `x11vnc` that are already running.  So, here's the script!


    
    <code class="brush: bash">
    #!/bin/bash
    
    #kill any running instances of x11vnc
    pkill ^x11vnc$
    
    #start new x11vnc
    /usr/local/bin/x11vnc -forever -allow <local subnet> -display :0 -bg
    
    exit 0
    </code>



In order to run the above script, fire up `gedit` and copy and paste the script into that.  Now this is important, you have to replace the `<local subnet>` bit with your local subnet.  For example if the router/gateway on your local network is 192.168.1.1 then 192.168.1 should be your subnet.  Save the script as `startx11vnc` or whatever you'd like to call it.  Now, open a terminal and change directory to wherever you saved the script and run the following command to make your script an executable.

`chmod a+x startx11vnc`

So that you don't have to write the full path to the script every time you want to run it, you can also move it to a directory that is in the `PATH` environment variable.  Basically your `PATH` environment variable is a list of directories that the bash shell will look in for executables so that you don't have to remember where they're all installed.  To see which directories are included, run the following in a terminal window.

`echo $PATH`

In my case I moved the script to `/usr/bin`.  In order to move the script to a system directory such as `/usr/bin`, you'll probably need to be root.

`mv startx11vnc /usr/bin`

Finally, to get your VNC server up and running automatically every time OpenSolaris starts, you need to open System -> Preferences -> Sessions.  Click Add in the window that pops up.  Fill in the name and description as whatever you want, and then fill in the command with the name of your script.  If you haven't put the script somewhere in your `PATH` variable, you'll have to put in the full path to the script.

Now all you need to do is install some VNC client software on any of the machines you want to use to access your server.  In my case I'm running [TightVNC](http://www.tightvnc.com) on Vista, however there are options for any major platform you can think of ([Mac](http://www.apple.com/downloads/macosx/networking_security/chickenofthevnc.html), Linux, Solaris, iPhone...).  For the record the default port will be used for the VNC server, so all you should need to connect is the IP address of your server, but if it asks for a port, it should be 5900.



## Scheduled Tasks


Another little utility I recently discovered is `at`.  This utility allows you to schedule a command to run at a particular time.  I have only used this command in a fairly basic sense, so please check the man pages/help for more info.  An example case might be to shut down your vnc server when you go to bed.  I don't know why you'd want to, but this is just an example.  Maybe you're worried a ninja will break in and then try to remote into your OpenSolaris box.  If this is the case, you might want to move this down your priority list and get some help.  Anyways, on with the example!


    
    <code class="brush: plain">
    :~$ at -t 200902171530
    at> pkill ^x11vnc$
    at> <eot>
    ...
    </code>



In the above example, we run the at command and use the `-t` switch to specify the time the following command(s) will run.  In this case the time is 15:30 on 17 Feb 2009.  Despite my interpretation of the documentation, I found that I had to specify the full time including the year, month and day as follows: `yyyymmddhhmm`.  Once the at command is run, you will then be given a different prompt that begins with `at>`.  This is where you begin entering the commands you would like to run at the specified time.  When you're done, type Ctrl+d to tell `at` that you're finished specifying the commands.  Once this is done, `at` will spit out a little message summarising what you've just done.

So, this concludes the first round of additions to my OpenSolaris fileserver setup.  As always, feel free to ask questions in the comments.

