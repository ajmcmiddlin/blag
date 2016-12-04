---
author: Andrew
comments: true
date: 2010-10-16 07:02:10+00:00
layout: post
link: http://ajmccluskey.com/2010/10/new-language-instead-of-autocomplete-in-ide/
slug: new-language-instead-of-autocomplete-in-ide
title: New Language Instead of Autocomplete in IDE
wordpress_id: 344
categories: Fixes
tags: autocomplete Eclipse languages settings Windows
---

While messing around with a personal project in Eclipse, I attempted to use the Visual Studio keyboard shortcut for autocomplete (ctrl + space).  Rather than completing the variable name I had started typing, it began trying to convert my keyboard input into Chinese characters.  At first I thought this might have been some idiosyncrasy of Eclipse.  After a little research I discovered that Eclipse does in fact use ctrl + space for autocomplete, but Windows (bless its soul for trying to help) was hijacking the shortcut and using it to change languages.  For anyone in a similar situation, here's how you fix it in Windows 7:



	
  1. Open Control Panel -> Region and Language

	
  2. Select the "Keyboards and Languages" tab

	
  3. Click the "Change keyboards..." button

	
  4. Select the "Advanced Key Settings"



You should now see a list of actions and keyboard shortcuts.  For each action that has a conflicting keyboard action, select it and click the "Change Key Sequence..." button.  Here you can either change the action to have no shortcut at all, or give it an alternate shortcut.  Keep clicking OK until you're out of the menus.  Immediately the keyboard shortcut stopped switching my language, and after a restart of Eclipse, started autocompleting.  Simple!

_References:_
[social.answers.microsoft.com](http://social.answers.microsoft.com/Forums/en-US/vistaappearance/thread/d8df1edc-2c9a-4f53-a492-df3165da46fa)

