---
author: Andrew
comments: true
date: 2015-01-07 15:28:55+00:00
layout: post
link: http://ajmccluskey.com/2015/01/brew-encfs-osxfuse-yosemite-upgrade-problem/
slug: brew-encfs-osxfuse-yosemite-upgrade-problem
title: Brew + encfs + osxfuse + Yosemite upgrade = problem
wordpress_id: 609
categories: Fixes
tags: brew EncFS OSX OSXFUSE
---

I finally pulled my finger out and updated to Yosemite yesterday. I probably don't need any of the features, but I prefer the look. It also seems like a good idea to stay up to date for warranty purposes - I once got an out of warranty battery replacement 4 years after purchase because I was a good Apple kid and stayed up to date. Anyway, as with the update to Mavericks, I ran into a problem with [EncFS](https://vgough.github.io/encfs/) not working.

<!-- more -->

From the error message about kexts (kernel extensions) I believe upgrading the OS upgrades the kernel, which then requires a new kext for [OSXFUSE](http://osxfuse.github.io). OSXFUSE is the low level software that allows you to implement filesystems without having to talk to the kernel directly, and is what EncFS sits on top of. It used to be that you could install EncFS from brew, and it would install OSXFUSE as a dependency. However, [Yosemite has added some security features](https://github.com/Homebrew/homebrew/issues/31164) that don't allow unsigned kernel extensions to be loaded. Period. This means that homebrew cannot compile and install OSXFUSE for you - which is its default approach. The workaround is that you can install OSXFUSE yourself using the packages on their site, or you can install a binary version (I'm assuming the same package as on the OSXFUSE site) of OSXFUSE that has been signed by its developers using brew.


    
    <code class="brush: text">
    $ brew update
    $ brew install Caskroom/cask/osxfuse
    </code>



Homebrew is helpful enough to tell you this when you go to intsall EncFS again. I followed their instructions and installed using the cask, but it didn't seem to work - Homebrew didn't see that I had OSXFUSE installed. I uninstalled all the old EncFS and OSXFUSE builds using this. It didn't work, but you might want to do it if you're in my situation because there's probably no point keeping those old versions around.


    
    <code class="brush: text">
    $ brew uninstall --force osxfuse
    $ brew uninstall --force encfs
    </code>



Finally, I discovered a System Preferences item named "FUSE for OS X". I opened that up and noticed a "Remove OSXFUSE" button. Figuring a re-install couldn't hurt, I used this pane to remove OSXFUSE and re-install it. Voila, brew saw that OSXFUSE was indeed installed and happily installed EncFS for me. Uninstalling and installing via brew might work too, but I didn't try it.

![osxfuse-syspref-dialog](http://ajmccluskey.com/wp-content/uploads/2015/01/osxfuse-syspref-dialog.png)

No idea on the details of what was wrong or why this seemed to fix it, but hopefully it helps someone.
