---
author: Andrew
comments: true
date: 2008-08-18 11:26:57+00:00
layout: post
link: http://ajmccluskey.com/2008/08/when-a-leap-year-isnt-a-leap-year/
slug: when-a-leap-year-isnt-a-leap-year
title: When a Leap Year Isn't a Leap Year
wordpress_id: 29
categories: Programming
tags: batch scripting leap year
---

Have you ever had one of those moments when you realise that something you held as a fundamental law of the universe doesn't work the way you thought it did?  Well I have, today in fact, while writing a batch script that needed to be aware of leap years.  Multiple choice question (I promise it's relevant), which of the following is a leap year: a) 1700 b)1900 c)both 'a' and 'b' d)none of the above.  Correct answer: d).  That's right, it turns out that being evenly divisible by 4 isn't enough to qualify as a leap year.

So, now that I've dropped that bombshell, who wants to know how to pick a leap year every time without embarassing yourself?  Those of you with your hands raised can lower them and simply keep reading, because I'll fill you in now.  In plain English, a leap year occurs when the year is evenly divisible by 400, OR if the year is evenly divisible by 4 but NOT 100.  Put another way, a leap year occurs every four years as you'd expect, except for the beginning of a new century, where a leap year only occurs if that year is evenly divisible by 400 (e.g. 1600, 2000 etc.).  A nice way to show this process is with the aid of a [spiffy flow chart](http://visualbasic.about.com/library/graphics/dykleapyr1-1.gif) referenced in the [about.com article](http://visualbasic.about.com/od/vbnetspecialtopics/a/bldykleapyra.htm) I found today.

For those of you who might need to code this, the wiki article on leap years has pseudocode and C-style [code examples of the algorithm](http://en.wikipedia.org/wiki/Leap_year#Algorithm).  Furthermore, for those of you who might need to check leap years in a batch script, here's a simple batch script to do so.  In its current form, the script is called with the year you're checking as the first and only argument, printing an answer in plain english before finishing.

    
    <code>@echo off
    
    SET _year=%1
    SET _bLeapYear=is not
    
    ::Check if the year is evenly divisible by 400
    SET /a _modYear=%_year% / 400
    SET /a _modYear=%_modYear% * 400
    If %_modYear%==%_year% SET _bLeapYear=is
    If %_modYear%==%_year% GOTO END
    
    ::Check if the year is evenly divisible by 4
    SET /a _modYear=%_year% / 4
    SET /a _modYear=%_modYear% * 4
    If NOT %_modYear%==%_year% GOTO END
    
    ::Check if the year is evenly divisible by 100
    SET /a _modYear=%_year% / 100
    SET /a _modYear=%_modYear% * 100
    If NOT %_modYear%==%_year% SET _bLeapYear=is
    
    :END
    echo %_year% %_bLeapYear% a leap year</code>


As you might have noticed, I'm substituting the modulus operation found in most algorithms (including the ones on the wiki article) with a division followed by a multiplication.  This is simply because batch scripts don't have a modulus operator.  For the mathematically curious, this works because we're using integer arithmetic.  For example, in a batch script, 2003 ÷ 4 = 500.  As you can see, there is no remainder in the result.  Furthermore, there is no rounding.  This means that _a _÷ _b _* _b_ will only equal _a_ when _a_ is evenly divisible by _b_, or, _a mod b = 0_.

And there you have it, more than you probably wanted to know about leap years.
