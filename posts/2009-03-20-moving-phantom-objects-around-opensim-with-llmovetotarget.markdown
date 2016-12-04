---
author: Andrew
comments: true
date: 2009-03-20 11:25:22+00:00
layout: post
link: http://ajmccluskey.com/2009/03/moving-phantom-objects-around-opensim-with-llmovetotarget/
slug: moving-phantom-objects-around-opensim-with-llmovetotarget
title: Moving Phantom Objects Around OpenSim With llMoveToTarget
wordpress_id: 235
categories: Guides Programming
tags: OpenSim
---

As part of a uni AI subject I've been playing around with implementing some basic path finding in OpenSim.  Part of that involved moving objects around a region using the [llMoveToTarget](http://wiki.secondlife.com/wiki/LlMoveToTarget) function.  After some frustration, I've discovered that phantom objects do not appear to respond to this function (i.e. you can't move them using this function).  I'm not entirely sure what a phantom object is exactly, so this might be bleedingly obvious to others.  Also, this may be incorrect as I haven't really investigated it thoroughly.  Anyways, if you're trying to move something in OpenSim (or maybe Second Life) using llMoveToTarget and it doesn't work, it might be worthwhile checking if the object is phantom.

On a personal note, I'm quite busy with uni at the moment, as well as courting (pun intended) my newfound sweetheart - squash (the sport, not the food).  So don't expect any major posts any time soon.  If I do anything original with OpenSim (i.e. not implementing my lecturer's code) I'll hopefully get around to posting it here for you all to play with.
