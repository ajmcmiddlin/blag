---
author: Andrew
comments: true
date: 2015-01-08 17:20:42+00:00
layout: post
link: http://ajmccluskey.com/2015/01/managing-users-and-groups-from-the-os-x-terminal/
slug: managing-users-and-groups-from-the-os-x-terminal
title: Managing users and groups from the OS X terminal
wordpress_id: 618
categories: Guides
tags: OSX
---

I was recently playing around with [SaltStack](http://www.saltstack.com) and wanted to create a salt user and group to run the master. This led me to learn some basics of user and group management on OS X, and I thought I'd put them here to remember. FYI I did all this on Yosemite, but it seems like this is the way things have worked for a while, so it should work on just about any OS X system that's still running I would think. No guarantees though.

<!-- more -->

First things first: there's no useradd command like on standard *nix systems. Instead, OS X uses `dscl`, the "Directory Service command line utility", to manage users. I'll let Wikipedia explain what a [directory service](https://en.wikipedia.org/wiki/Directory_service) is. It seems like one of those fairly simple concepts that becomes shrouded in terminology and abstraction, so I'm going to make a hand wavey attempt at explaining what I've intuited.  The abstraction for datasources is directories (hence the name I guess), so things tend to look like paths on a filesystem. From the man pages, `dscl` deals in paths and keys, where paths are what you'd expect and keys are like the leaves of a tree (think files in directories) that store facts about path they're in. For our purposes, users and groups are paths that contain keys with facts about that user or group. Hopefully this will become clear when we run some commands. Finally, `dscl` operates on datasources, which can be local or remote - e.g. other servers running a directory service. Everything we do will be on the local datasource (i.e. the computer you're running the commands on), so our first argument to `dscl` will always be a '.' to signify the local datasource.

Before we start using `sudo` and making changes, let's start with some queries to get comfortable. I've fabricated the output of commands based on real output, as well as trimmed a bunch of data to save on space, but you should get the idea.


    
    <code class="brush: text">
    # List the subdirectories under /Users - i.e. list the users on the system
    $ dscl . -list /Users
    greententacle
    purpletentacle
    bernard
    ...
    
    # List the subdirectories under /Groups - i.e. the groups on the system
    $ dscl . -list /Groups
    tentacles
    daemon
    staff
    ...
    
    # Read all keys in directory and output their values
    $ dscl . -read /Users/purpletentacle
    PrimaryGroupID: 42
    RealName:
     Purple Tentacle
    UniqueID: 505
    UserShell: /bin/bash
    
    # Read a specific key
    $ dscl . -read /Users/purpletentacle UniqueID
    UniqueID: 505
    
    # List paths with the value of a key
    $ dscl . -list /Users UniqueID
    greententacle	501
    purpletentacle	505
    bernard			502
    ...
    </code>



We can use `dscl` to modify this information, or in our case, add new information. We're about to create a new user and group, but before we do I should mention that while you _can_ add a new group with `dscl`, it's better to use `dseditgroup` as explained on [Super User](http://superuser.com/a/214311/89966).


    
    <code class="brush: text">
    # Add the humans group. See man page for explanation of options.
    $ sudo dseditgroup -o create -r "Group for Humans not tentacles" humans
    
    # Get the current highest user ID so we can pick something higher
    $ dscl . -list /Users UniqueID | awk '{print $3}' | sort -n | tail -1
    
    # Create the user
    $ sudo dscl . -create /Users/hoagie 
    $ sudo dscl . -create /Users/hoagie UniqueID <one more than current highest above>
    $ sudo dscl . -create /Users/hoagie UserShell /bin/bash # /usr/bin/false if you don't want the user to have a shell 
    $ sudo dscl . -create /Users/hoagie RealName "Hoagie"
    
    # Add user to group
    $ sudo dseditgroup -o edit -a hoagie -t user humans
    </code>



That's about all there is. Once you get a feel for these commands, their man pages should be accessible enough to work out anything else you want to do.

References:



	
  * [Super User answer mentioning `dseditgroup`](http://superuser.com/questions/214004/how-to-add-user-to-a-group-from-mac-os-x-command-line).

	
  * [Server Fault post with `dscl` commands](http://serverfault.com/questions/20702/how-do-i-create-user-accounts-from-the-terminal-in-mac-os-x-10-5).

	
  * [Day of the Tentacle (DOTT) on Wikipedia](https://en.wikipedia.org/wiki/Day_of_the_Tentacle)


