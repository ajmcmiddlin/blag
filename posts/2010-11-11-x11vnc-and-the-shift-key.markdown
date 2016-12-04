---
author: Andrew
comments: true
date: 2010-11-11 14:49:12+00:00
layout: post
link: http://ajmccluskey.com/2010/11/x11vnc-and-the-shift-key/
slug: x11vnc-and-the-shift-key
title: x11vnc and the Shift Key
wordpress_id: 350
categories: Fixes
---

For anyone out there who has read my previous posts on building your own OpenSolaris file server, I have another addition to make.  In [my second post](/2009/02/more-fun-with-opensolaris) regarding, among other things, setting up a VNC server, I included a script to start x11vnc.  This setup has always worked fine for me with one exception: the shift key.  Whenever I want to type a capital letter, or use the shift key for any reason, it doesn't work.  Tonight, after a cursory google search, I found [someone with an answer](http://www.bramschoenmakers.nl/en/node/714).  I rushed off and added the option to my script before running it to apply the changes.  Lo and behold, the shift key was working.  "Great, that was too easy" I thought to myself.  However, I quickly discovered that the arrow keys were doing weird things.  The most problematic of these things being the lack of repeating the previous command entered at the command line when pressing the up arrow.  This was not a sacrifice I was willing to make.  Luckily, the ever informative [serverfault](http://www.serverfault.com) came to the party with [an even better solution](http://serverfault.com/questions/172800/x11vnc-how-to-enable-shift-key).

I'll leave it as an exercise for the reader to apply these changes to the original script.  If you have problems, don't hesitate to contact me or comment.  Finally, a comment on the page with the first solution mentioned that this issue has been fixed in a more recent version of x11vnc, however I am yet to check this out.
