---
author: Andrew
comments: true
date: 2009-03-10 13:28:44+00:00
layout: post
link: http://ajmccluskey.com/2009/03/sharing-glut-code-between-windows-and-mac-os-x/
slug: sharing-glut-code-between-windows-and-mac-os-x
title: Sharing GLUT Code Between Windows and Mac OS X
wordpress_id: 227
categories: Guides Programming
tags: apple C++ GLUT OpenGL
---

If you've seen my [last post](http://www.ajmccluskey.com/2009/02/installing-freeglut-on-vista-x64/), you're aware that I've been playing with OpenGL and GLUT for a uni subject.  As I like to straddle the OS fence and use a mixture of Windows Vista x64, Windows XP and Mac OS X 10.5, I like to have platform independent solutions to problems.  The latest little issue I've come across regarding GLUT apps is that GLUT on the Mac has a different include.  To include GLUT in a project for the mac, you must use the following:


    
    <code class="brush: cpp">
    #include <GLUT/glut.h>
    </code>



and not:


    
    <code class="brush: cpp">
    #include <GL/glut.h>
    </code>



In order to cut code that will compile successfully on either platform we can use optional includes with the `#ifdef`, `#else` and `#endif` preprocessor directives.  Props to "kainjow" for providing this information in [his/her forum post](http://forums.macrumors.com/archive/index.php/t-412567.html).  Here's how:


    
    <code class="brush: cpp">
    #ifdef __APPLE__
    #include <GLUT/glut.h>
    #else
    #include <GL/glut.h>
    #endif
    </code>



All this does is say "if we're running on an Apple machine, `include GLUT/glut.h`, otherwise `include GL/glut.h`.  Now to actually learn OpenGL/GLUT...
