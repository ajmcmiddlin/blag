---
author: Andrew
comments: true
date: 2011-06-26 11:03:29+00:00
layout: post
link: http://ajmccluskey.com/2011/06/swearing-at-your-file-system/
slug: swearing-at-your-file-system
title: Swearing at your File System
wordpress_id: 385
categories: Fixes
tags: shell unix
---

Recently I was working on a Solaris box remotely and lost my temper.  I can't remember why, and it's not relevant.  What is relevant is that in my rage, I swore at my file system.  I know it was childish, and I definitely know better, but in the heat of the moment, I made a mistake.  Unfortunately, I was in the process of naming a file in Vi, and I ended up with a file name full of punctuation marks.  Realising my mistake, I immediately tried to remove the offending file.  To my surprise, while I could see the file by issuing a simple _ls_, I couldn't remove it.  Bash issued a syntax error, interpreting the punctuation marks as special symbols.  I now realise that I could have very easily escaped the offending characters, but I wasn't that smart at the time, so what follows was my rather roundabout solution.  While these steps may not be useful in this scenario, parts of the solution may prove useful for others.

Remembering back to my systems architecture classes at university, I started thinking of how I might get to my file via its inode rather than its file name.  A quick search of the man page for _ls_ revealed that I could list all of the inodes in a directory.  Success; I now had another method of accessing the file.  Checking the man page for _mv_ and _rm_, I couldn't see any way to rename or delete the file via its inode.  Having recently done some work with _find_ I knew how versatile and useful a tool it is.  Checking its man page, I was pleased to find that I could search for files by their inode number.  Combining this with the exec flag, I was able to rename my file to something sensible.  The code snippet below gives an example of how this all fits together.


    
    <code class="brush: plain">
    ajmccluskey@solaris $ ls
    ^()$@  Documents
    ajmccluskey@solaris $ ls -i
    264015 ^()$@    262180 Documents
    ajmccluskey@solaris $ find . -inum 264015 -exec mv {} test \;
    ajmccluskey@solaris $ ls
    test  Documents
    </code>



So there you have it.  My lacking knowledge of Bash will never get in the way of me solving a problem; albeit in a very roundabout manner.
