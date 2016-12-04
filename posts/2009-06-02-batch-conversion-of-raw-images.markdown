---
author: Andrew
comments: true
date: 2009-06-02 05:53:28+00:00
layout: post
link: http://ajmccluskey.com/2009/06/batch-conversion-of-raw-images/
slug: batch-conversion-of-raw-images
title: Batch Conversion of RAW Images
wordpress_id: 267
categories: Guides
tags: batch scripting image JPEG RAW
---

A little while ago I took some RAW images with my DSLR.  Having never used a DSLR or dealt with a RAW format before, I had no idea how to view or use the files.  After a brief search, I discovered a most useful program called [UFRaw](http://ufraw.sourceforge.net/index.html).  It's free (and I think open source), and allows you to open and edit a variety of RAW image formats.  Not only that, but it also acts as a GIMP plugin and a batch converter, so you can load your images directly into [The GIMP](http://www.gimp.org/) for editing.  All of these features appear to be documented on the UFRaw site, although to tell the truth I really haven't used it all that much.

Today I wanted to convert about 45 RAW images to JPEG.  Obviously this is a job for the batch conversion tool that comes with UFRaw, however, the batch conversion program converts images to .PPM by default.  In order to batch convert images to JPEG, an additional command line argument must be passed to the program.  Having to get into the command line removes the drag-and-drop simplicity you get with the default behaviour, and I knew I could do better. It quickly became apparent that it was time to get my batch file on.  Below is my example script that allows you to drag-and-drop any number of files onto it and have them converted to JPG.  This same script could be re-applied to any other program that works in the same way by simply changing the program and command line arguments.


    
    <code class="brush: powershell">
    @ECHO OFF
    START "converting RAWs to JPEG" "C:\Program Files (x86)\GIMP-2.0\bin\ufraw-batch.exe" --out-type=jpg %*
    </code>



The first line of the script just turns off printing to the command line.  The second line is where the magic happens.  Breaking up the second line we have:



	
  * `START` - used to start a program

	
  * the title of the command line window that opens to run the command

	
  * the path to the executable we're calling (in this case, `ufraw-batch.exe`)

	
  * the command line arguments passed to `ufraw-batch.exe` to convert images to JPEG instead of PPM

	
  * finally, the `%*` at the end passes all of the arguments/files passed to the script, onto the `ufraw-batch.exe` command we've just defined


Obviously if you want to use the script yourself you may need to change the path to `ufraw-batch.exe`.
