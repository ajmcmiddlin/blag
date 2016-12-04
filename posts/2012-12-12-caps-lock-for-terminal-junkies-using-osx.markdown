---
author: Andrew
comments: true
date: 2012-12-12 16:12:55+00:00
layout: post
link: http://ajmccluskey.com/2012/12/caps-lock-for-terminal-junkies-using-osx/
slug: caps-lock-for-terminal-junkies-using-osx
title: Caps Lock for Terminal Junkies Using OSX
wordpress_id: 436
categories: Guides
tags: keyboard Mac OSX terminal
---

It's been a couple of years since I've owned a Mac, but now I'm happily back in Apple's clutches.  During the intervening period between owning my last Mac and this one, I fell in love with remapping the Caps Lockkey to be another control key.  Given that I spend a fair chunk of my day in Vim and a terminal, this saves alot of strain on my pinky finger and puts an otherwise useless key to good use.  However, now that I'm back in Mac land I faced a dilemma: should Caps Lock be the control key, or the command key.  You see, I've become very use to hitting caps+c/v to copy and paste, but Apple, bless their infinite wisdom, have a command key that is used for this purpose.  What's a geek todo?



<!-- more -->



Luckily, the very useful [KeyRemap4MacBook](http://pqrs.org/macosx/keyremap4macbook/) and [PCKeyboardHack](http://pqrs.org/macosx/keyremap4macbook/pckeyboardhack.html.en)came to my rescue.  KeyRemap4MacBook gives you a very powerful preference pane to remap keys.  Among its most useful features is the ability to remap keys in different contexts, that is, when different apps are receiving keyboard input.  Furthermore, the configuration options can be extended by editing an XML file.  With these two tools at hand I was able to remap my Caps Lock key to act like the control key in Vim, MacVim, and terminal applications; and act like the Command key everywhere else.  Rather than repeat what is already covered in the documentation I'll just encourage you to go read it now.  The following sections should give you the background you need.  I know it looks like a lot, but it's pretty brief and there are plenty of pictures.  If you're really pressed for time you can probably just read the last section on the Caps Lock key and proceed on to the final instructions.






	
  * [Remapping Keys](http://pqrs.org/macosx/keyremap4macbook/document.html.en)

	
  * [Event Viewer](http://pqrs.org/macosx/keyremap4macbook/document-eventviewer.html.en)

	
  * [Adding Your Own Settings](http://pqrs.org/macosx/keyremap4macbook/document-private-xml.html.en)

	
  * [Event Viewer](http://pqrs.org/macosx/keyremap4macbook/document-eventviewer.html.en)

	
  * [Specifying an Application](http://pqrs.org/macosx/keyremap4macbook/xml-appdef.html.en)

	
  * [String Replacement](http://pqrs.org/macosx/keyremap4macbook/xml-replacementdef.html.en)

	
  * [Caps Lock](http://pqrs.org/macosx/keyremap4macbook/faq-capslock.html.en)





Putting all of the above knowledge to use I ended up with the following solution.  Firstly, follow the instructions above on changing the Caps Lock key.  However, don't actually map the Caps Lock key as we'll define our own mapping option and select that.  Once the Caps Lock key is available for mapping add the following to your private.xml file.  It creates a list of applications where Caps Lock should be control and not command; defines a new application for iTerm2, which is my default terminal; and defines the mappings.  This should create a checkbox option at the top of the KeyRemap4MacBook preference pane that you can use to enable/disable your new, custom feature.




    
    <code class="brush: xml">
    <?xml version="1.0"?>
    <root>
      <replacementdef>
        <replacementname>CAPS_IS_CONTROL_APPS</replacementname>
        <replacementvalue>TERMINAL, ITERM2, VI</replacementvalue>
      </replacementdef>
    
      <appdef>
        <appname>ITERM2</appname>
        <equal>com.googlecode.iterm2</equal>
      </appdef>
    
      <item>
        <name>Caps Lock is Command Except in Terminals and Vim</name>
        <identifier>private.caps_is_command</identifier>
        <block>
          <not>{{ CAPS_IS_CONTROL_APPS }}</not>
          <autogen>--KeyToKey-- KeyCode::PC_APPLICATION, KeyCode::COMMAND_L</autogen>
        </block>
        <block>
          <only>{{ CAPS_IS_CONTROL_APPS }}</only>
          <autogen>--KeyToKey-- KeyCode::PC_APPLICATION, KeyCode::CONTROL_L</autogen>
        </block>
      </item>
    </root>
    </code>
