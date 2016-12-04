---
author: Andrew
comments: true
date: 2012-06-27 06:10:21+00:00
layout: post
link: http://ajmccluskey.com/2012/06/hdmi-audio-on-lmdexfce/
slug: hdmi-audio-on-lmdexfce
title: HDMI Audio on LMDE/XFCE
wordpress_id: 422
categories: Guides
tags: audio hdmi Linux lmde PulseAudio sound xfce
---

While at home sick today I fired up Firefox to watch some videos on my media centre PC and was quickly disappointed to hear the audio coming from the laptop's speakers rather than through my television.  XBMC on this machine uses HDMI for audio out, and has since I set it up, so I was confident that it was just a matter of configuration.  After some messing around trying to find config windows in XFCE and searching the world wide web, I found a post mentioning _pavucontrol_.  Hoping pavucontrol offered a solution I installed it and launched it from the command line.  It gave me a nice GUI as seen below, and more importantly, allowed me to configure the output device that PulseAudio should use by default.





![screen capture of choosing HDMI output in pavucontrol GUI](/wp-content/uploads/2012/06/pavucontrol_output_configuration.png)





After I'd enjoyed my video, I thought I should try and gain at least a cursory understanding of what I'd just done.  Wikipedia came to the rescue with a [high level description of PulseAudio](http://en.wikipedia.org/wiki/PulseAudio), and a helpful diagram that shows where it sits in the grand scheme of things.  One thing that really caught my eye was the following; "One of the goals of PulseAudio is to reroute all sound streams through it...".  With that in mind I'm guessing that a fair few applications on my system are using Pulse, but I'd just never noticed the issue because I could configure XBMC to use a specific output (it has an independent sound system?).  In any event, I hope this helps some other confused soul to get their audio pumping through HDMI.

