---
author: Andrew
comments: true
date: 2008-12-14 10:47:09+00:00
layout: post
link: http://ajmccluskey.com/2008/12/mamp-after-upgrading-to-leopard/
slug: mamp-after-upgrading-to-leopard
title: MAMP After Upgrading to Leopard
wordpress_id: 77
categories: Fixes
tags: Leopard MAMP
---

I just went to do some work on a Wordpress theme for my girltrond and discovered that upgrading to Leopard has broken my MAMP setup.  Here is a quick rundown on what I've done to fix it.  First of all, trying to visit pages under my user account's Sites folder left me with a HTTP 403 Forbidden error.  In order to fix this I follwed the [instructions on Apple's support site](http://support.apple.com/kb/TA25038?viewlocale=en_US).

Next, to get PHP running.  I was using [entropy](http://www.entropy.ch/software/macosx/php/) to get PHP5 with Tiger, however it appears that Leopard includes a PHP5 install by default.  To enable it, we need to edit Apache's configuration file (_httpd.conf_) which you can do with the following command in Terminal (under Utilities):


    
    <code class="brush: bash">
    sudo nano /etc/apache2/httpd.conf
    </code>



<!-- more -->
To enable PHP5, simply uncomment the line that begins `#LoadModule php5_module...` by removing the hash symol from the beginning of the line.  Once this is done, restart Apache by opening System Preferences, then go into Sharing and untick Web Sharing and then check it again to restart.  I'm not sure if the upgrade process keeps your php.ini file if you were using the PHP4 install provided by Tiger.  It certainly doesn't if you were using entropy like I was, in which case you'll need to update that with any settings you had in there (e.g. includes paths).  In order to get a working copy of php.ini, type the following commands into Terminal:


    
    <code class="brush: bash">
    sudo cp /etc/php.ini.default /etc/php.ini
    sudo nano /etc/php.ini
    </code>



This makes a working copy of your _php.ini_ file from the default settings and then opens it for editing in nano, a command line text editor.  Alternatively you can just enter the first command and then edit _/etc/php.ini_ using your preferred method.

Next on my problem list was getting MySQL to start.  I installed MySQL from their official binaries and had a preference pane to start and stop the MySQL server.  This no longer works after upgrading from Leopard, however the people that make MySQL have released a [new preference pane](ftp://ftp.mysql.com/pub/mysql/download/gui-tools/MySQL.prefPane-leopardfix.zip) that works in Leopard.  Just unzip it somewhere, then double click it to install it and enter your administrator's password in when you're asked.

Another problem I came across was that PHP was unable to communicate with MySQL.  This happened because PHP was looking for the MySQL socket in the wrong place.  You have two options to fix this.  Apple's support site has [an article that walks you through changing your php.ini](http://support.apple.com/kb/TS1999?viewlocale=en_US) file to look for the MySQL socket in the right place.  Alternatively you can cheat like I did by creating a symbolic link to the socket.  This is basically a shortcut that sits where PHP looks for the socket, and points to the real socket that MySQL creates.  You can do this by entering the following command in Terminal:


    
    <code class="brush: bash">
    sudo ln -s /tmp/mysql.sock /var/mysql/mysql.sock
    </code>



I think this covers all of the major issues I came across while fixing my MAMP environment.  If I come across any more issues I'll add them here.  Otherwise, if you have any issues or comments to add, feel free to post a comment or [contact me](/contact/).
  *[MAMP]: Web platform using: Mac Apache MySQL PHP
