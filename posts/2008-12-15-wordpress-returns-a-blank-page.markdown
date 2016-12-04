---
author: Andrew
comments: true
date: 2008-12-15 13:12:25+00:00
layout: post
link: http://ajmccluskey.com/2008/12/wordpress-returns-a-blank-page/
slug: wordpress-returns-a-blank-page
title: Wordpress returns a blank page
wordpress_id: 81
categories: Fixes
tags: themes Wordpress
---

Here's a pro tip for anyone working with Wordpress.  If you're ever faced with a completely blank page when you visit your site, check that the theme you are meant to be using actually exists.  I just spent about half an hour checking permissions and error logs for my development environment, only to find that when I backed up the theme I was working on I inadvertently cut the theme's folder instead of copy it.  It appears that when Wordpress doesn't find the theme it is meant to use it just returns an empty page instead of an error message.  Better than an error message, it could even try using the default theme and email the adminstrator that something has gone wrong with their chosen theme.  I'm using Wordpress 2.7 at the moment, so if this is version specific or shouldn't be happening, can someone please leave a comment or email me.

Given that Wordpress appears to be open source, maybe I should attempt to write such a fix and finally contribute to an open source project like I've been planning to.
