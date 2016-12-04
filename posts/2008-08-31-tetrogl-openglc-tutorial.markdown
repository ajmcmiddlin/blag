---
author: Andrew
comments: true
date: 2008-08-31 11:06:26+00:00
layout: post
link: http://ajmccluskey.com/2008/08/tetrogl-openglc-tutorial/
slug: tetrogl-openglc-tutorial
title: TetroGL - OpenGL/C++ tutorial
wordpress_id: 37
categories: Programming
tags: C++ OpenGL programming
---

I'm currently planning on a career in games programming and was looking for some beginners' tutorials in OpenGL using C++ and found [a goodun](http://www.codeproject.com/KB/game/TetroGL_Part1.aspx) over at code project.  It's a 3 part tutorial, and as I'm writing this the first two parts have been posted, complete with code and Visual Studio project files.  As a relative beginner with C++ and a complete beginner with OpenGL I've found this fairly easy to follow.  The code is pretty well commented, and the tutorial itself explains things clearly and doesn't take too much for granted aside from a basic knowledge of programming/C++.

A trap for young players, there was a slight error in the code from the first part that caused the program to not shut down and stay in memory after the window had closed.  Nobody else seemed to have this problem, and it could have just been me being difficult and using Vista.  Nonetheless, if the code for part 1 hasn't been updated, the fix is in the comments below part 1.  Basically the main application loop did not clear the message queue with each iteration, only taking the first message.  From what I've read on the issue this means that the WM_QUIT message sent from PostQuitMessage() is never received.  This is because PostQuitMessage(0) sets a flag on the message queue which tells it to return a WM_QUIT message if the queue is empty.  Obviously if the queue is never cleared, this message is never returned when the queue is queried for the next message.

If you have any problems with this, feel free to [contact me](http://www.ajmccluskey.com/contact/).  Happy coding!
