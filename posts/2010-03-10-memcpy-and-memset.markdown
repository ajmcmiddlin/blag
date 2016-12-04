---
author: Andrew
comments: true
date: 2010-03-10 10:22:34+00:00
layout: post
link: http://ajmccluskey.com/2010/03/memcpy-and-memset/
slug: memcpy-and-memset
title: memcpy and memset
wordpress_id: 294
categories: Programming
tags: C++ memcpy memset programming
---

Now that I am back in the world of full-time work, I have begun to think more seriously about my goals.  One of my life goals, which I have recently decided are a superset of career goals, is to learn as much as possible.  I have also decided that learning as much as possible is also a member of my set of career goals.  To alleviate any confusion, I drew a Venn diagram summarising the situation.

![terrible Venn diagram of my life goals](/wp-content/uploads/2010/03/venn_of_goals.png)

As part of my new job, I am programming in C++ for a good part of each day.  A consequence of this is that I am constantly learning new methods for writing and debugging C++ code, and programming in general.  In an effort to document and share this knowledge, I'm going to start by telling you about `[memcpy](http://www.cplusplus.com/reference/clibrary/cstring/memcpy/)` and `[memset](http://www.cplusplus.com/reference/clibrary/cstring/memset/)`.  `memcpy` and `memset` are two functions included in the C standard libraries.  `memcpy` is used to copy a block of memory from a source to a destination, and `memset` is similarly used to set a block of memory to a particular value.  While quite low level functions, these can both be very useful if speed is critical and you're not working with higher level containers for whatever reason (yes, I know everyone says to use vectors if you're not using straight C).

<!-- more -->



## `memcpy`


The situation where I find myself using _memcpy_ most is when copying from one array to another.  This can be the entire array, or a contiguous part thereof.  For example:


    
    <code class="brush: c">
    #include <string.h>
    #include <stdio.h>
    
    int anMyNumbers[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int anMyNumbersCopy[NUMBERS_COUNT];
    
    /*copy one array to another*/
    memcpy(anMyNumbersCopy, anMyNumbers, sizeof anMyNumbersCopy);
    </code>



Above we have our first and most obvious example.  This code, as you can probably guess, simply copies integers from one array to another so that both arrays store the numbers 1 through 10.  In addition to copying whole arrays, we can also copy a contiguous subset of the array like so:


    
    <code class="brush: c">
    /*initialise to 0 and copy half of the array*/
    memset(anMyNumbers, 0, sizeof anMyNumbers);
    memcpy(anMyNumbersCopy + 5, anMyNumbers, 5*sizeof(int));
    </code>



Here we have set all of the values in `anMyNumbers` to 0 (see section `memset` below), and then copied the first 5 values in `anMyNumbers` to the last 5 in `anMyNumbersCopy`.  Finally, we can also copy a row, or range of rows, from one 2D array to another.  This is because 2D arrays are still just 1D blocks of memory underneath.  Observe:


    
    <code class="brush: c">
    int anMyFirst2D[5][5] = {
    	{1, 2, 3, 4, 5},
    	{6, 7, 8, 9, 10},
    	{11, 12, 13, 14, 15},
    	{16, 17, 18, 19, 20},
    	{21, 22, 23, 24, 25}
    };
    int anMySecond2D[5][5];
    
    /*set all of the values in anMySecond2D to 0 and then copy across 2 rows*/
    memset(anMySecond2D, 0, sizeof anMySecond2D);
    memcpy(anMySecond2D+2, anMyFirst2D+2, 2*5 * sizeof(int));
    </code>



In the above code we have copied only rows 2 and 3 (starting at 0) from one array to the other.



## `memset`



Now, `memset` can be used in a similar fashion to `memcpy`, however there is a big pitfall that I don't want to see you fall into.  Basically, `memset` doesn't care about the type of your destination.  Even worse than that, although the value used to set the block of memory is specified as an int in `memset`'s signature, it actually casts this to an unsigned char!  So the end result of using `memset` is that each and every byte of your destination is individually set to whatever the unsigned char version of your value is.  The most obvious side effect of this is that you cannot use `memset` to initialise an array of ints with any value other than 0.  Well, technically you can, but you won't get the intuitive result, as evidenced by the below code.


    
    <code class="brush: c">
    memset(anMyNumbers, 12345678, sizeof anMyNumbers);
    </code>



Executing the code above will set the value of each integer to 1313754702, and not 12345678 as you might think (at least on Windows using VC++ Express 2008 to compile it).  This is because `memset` casts 12345678 to an unsigned char, which is equivalent to the lowest order byte of the integer, which is 0x4E.  This value is then copied into each and every byte in `anMyNumbers`, resulting in each integer's value being 0x4E4E4E4E, which is 1313754702.  If you don't believe me, here's a screen cap of me debugging the above code.
![screen cap of debugging memset](http://www.ajmccluskey.com/wp-content/uploads/2010/03/memcpy-20100308.png)

Now that's all well and good Mr. My-Integer-Is-Bigger-Than-A-Byte, but what about if my integer fits neatly within a byte?  The short answer is, you're still screwed.  The slightly longer answer is that `memset` will still work in exactly the same way, meaning that it will copy the lowest order byte of your integer into EVERY SINGLE BYTE of your destination individually.  For example, the following code will give you an array of integers that all have the value 67372036.  Not 4.


    
    <code class="brush: c">
    memset(anMyNumbers, 4, sizeof anMyNumbers);
    </code>



Now that I've told you about when `memset` breaks, I can tell you about the cases where it will work as expected.  Firstly, initialising an array to 0, as seen in the `memcpy` examples above, is perfectly safe given that 4 bytes of zeros has the same integer value as 1 byte of zeros.  Secondly, setting an array of booleans to either true or false will work just fine as a boolean is 1 byte in size, so the value will correspond correctly the each element in the array.  That is to say, the code below will do what you think.


    
    <code class="brush: c">
    bool abMyBools[5];
    memset(abMyBools, true, sizeof abMyBools);
    memset(abMyBools, false, sizeof abMyBools);
    
    memset(anMyNumbers, 0, sizeof anMyNumbers);
    </code>



Now go forth and propagate this knowledge, and then sit back and wait for the next thrilling installment.  Maybe I'll quickly explain why using sizeof in these calls is a good idea, and when that will break on you as well...

