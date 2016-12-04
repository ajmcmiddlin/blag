---
author: Andrew
comments: true
date: 2009-02-25 12:33:11+00:00
layout: post
link: http://ajmccluskey.com/2009/02/installing-freeglut-on-vista-x64/
slug: installing-freeglut-on-vista-x64
title: Installing freeglut on Vista x64
wordpress_id: 196
categories: Guides Programming
tags: freeglut GLUT OpenGL Vista x64
---

I've recently had to install [freeglut](http://freeglut.sourceforge.net/index.php#download") as part of a uni subject and ran across an issue due to the fact that I'm running Vista x64 and wanted to dynamically link freeglut against anything I develop during the semester.  First of all, building freeglut (version 2.4.0) was problem free.  I simply opened `freeglut.dsw` (in the root directory of the freeglut download) in Visual Studio 2005 (VS2005), agreed to convert the project files to a newer format, and then built the solution.  An additional step I took, which may be completely unnecessary, was to change the build type from Debug to Release.  Both build types built fine, but I used the release versions as I doubt I will be debugging freeglut.  Also note that I compiled this as the default 32-bit, not 64-bit, as I'm building 32-bit GLUT apps for compatibility.

Once this was done, I followed the info in `README.win32` (in the freeglut directory) and copied the files to the relevant directories.  The specific directories I used for a Vista x64 system with VS2005 were as follows:





  * copy `freeglut.h`, `freeglut_ext.h`, `freeglut_std.h`, and `glut.h` from `freeglut-2.4.0\include\GL` to `C:\Program Files (x86)\Microsoft Visual Studio 8\VC\PlatformSDK\Include\gl`.


  * copy `freeglut.lib` from `freeglut-2.4.0\Release` to `C:\Program Files (x86)\Microsoft Visual Studio 8\VC\PlatformSDK\Lib`.  Note that the correct destination should already contain `opengl32.lib`, `glu32.lib`, and `glaux.lib`


  * Now here's the part that caused me problems: copy `freeglut.dll` to `C:\Windows\SysWOW64` and NOT `C:\Windows\System32` as you might expect, given that it's a 32-bit library being used in 32-bit projects



Once the files are in place, you should be ready to use freeglut in your apps.  To do so, simply include freeglut as follows:


    
    <code class="brush: cpp">
    #include "GL/glut.h"
    </code>



As you can see, we're referencing `glut.h` and not `freeglut.h`.  My guess is that this is done to maintain platform independence if your code was being built by a different incarnation of GLUT.  For example I used this method to build and run the test app provided by my lecturer (unchanged), who uses the [original GLUT](http://www.xmission.com/~nate/glut.html).

***UPDATE 2009-02-27 ***
I copied the files I built on my Vista x64 machine to a 32-bit install of XP SP3 on my laptop and GLUT worked fine.  The only difference to the above instructions (obviously aside from not building freeglut again) is to copy `freeglut.dll` to `C:\Windows\System32`.  I've also put together a [zip package of my build](/files/2009/02/freeglut-2.4-win32.zip) for anybody who wants to avoid building themselves.
