---
author: Andrew
comments: true
date: 2011-02-02 14:52:01+00:00
layout: post
link: http://ajmccluskey.com/2011/02/how-to-setup-mercurial-on-windows-for-use-with-bitbucket-over-ssh/
slug: how-to-setup-mercurial-on-windows-for-use-with-bitbucket-over-ssh
title: How to setup Mercurial on Windows for use with bitbucket over SSH
wordpress_id: 359
categories: Guides Programming
tags: atlassian bitbucket mercurial programming
---

I enjoy making software.  I also enjoy playing with the systems that support making software: for example source control, unit testing, continuous integration, issue tracking.  As a result, I have wanted to set up my very own build server to manage personal projects for a while now.  As an intermediate step to building my own server, which looks to be a rather large undertaking, I have elected to use [bitbucket](http://bitbucket.org). Using bitbucket not only provides me with these services while I work on developing my own, it should help to inform the choices I make regarding my own system.

For those who are new to bitbucket, as I am, it is an online service that provides, among other things, Mercurial hosting for your source code.  Its use of Mercurial, which is my current source control tool of choice, and its use of ssh keys for simple and secure communication are the two main reasons I have chosen it.  Speaking of ssh keys for communication, there are two different ways to access your code once it is on bitbucket: via https using a username and password, or via SSH using public/private keys.  I've elected to use SSH because using https requires that you enter a password whenever you commit, and I'm too lazy for that.  My personal philosophy is to automate as much as possible, and make things as easy as possible long term; even if that requires additional effort up front.

<!-- more -->


## Prerequisites


There are a few things that I will leave to you to do before following this guide.

Firstly you need to Obtain a bitbucket account and create a repository.  This is a fairly obvious requirement and I'll let you work it out.

Next you'll need to install Mercurial.  The easiest and recommended method is to download and install [TortoiseHg](http://tortoisehg.bitbucket.org/), however if you've installed it some other way you're probably capable of modifying these instructions to suit your installation.

Finally, if you're using SSH to interact with bitbucket (as this guide suggests) you'll need a set of public and private ssh keys.  For this I used [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).  If you haven't installed Mercurial using TortoiseHg then you'll also need Plink.  If you'd like to use pass phrases to protect your private key, or want (in my opinion anyway) a more robust configuration, then you should also grab Pageant.  Plink is used for automated (i.e. not interactive) SSH operations, and Pageant is used to store private keys in memory for use with Plink/PuTTY so that they don't have to be specified with each use.  It also removes the need to enter the pass phrase each time you use a key that is protected with a pass phrase.  As far as this guide is concerned, Pageant is required for private keys with pass phrases and optional, though recommended, for keys without pass phrases.



## SSH Keys


I'm not going to go into the depths of SSH keys and encryption (mostly because I don't know them), however I will offer my basic understanding of public-key cryptography as it applies here.  When you share code with bitbucket over the internet, using SSH allows both sides to encrypt the data being sent so that nobody else is able to read it.  The way this is achieved by SSH is using asymmetric keys.  That means that each party has two keys: a public key that anybody can have, and a private key that is kept, well, private.  A public key is used to encrypt data that only the private key can decrypt.  A simple analogy might be you having a vast collection of locks that can only be opened by the one key.  You hand out locks to anyone who wants to send you something private (the public key) and they then send you locked packages that only you can open with your key (private key).  Wikipedia offers much more detailed explanations of both [SSH](http://en.wikipedia.org/wiki/Secure_Shell) and [public-key cryptography](http://en.wikipedia.org/wiki/Public-key_cryptography).

To generate a public and private key that you can use to communicate with bitbucket, use the PuTTYgen tool.  The default settings for the keys worked for me, so don't play with them unless you know what you're doing.  To create a public/private key pair, simply click the generate button.  The tool will then ask you to wave your mouse over the window to generate random data that will be used to create the keys.  For now we'll assume that you're not protecting your private key with a pass phrase, so leave that blank.  If you wish to use Pageant to load and manage your private keys more securely (see below) then this is where you also add the pass phrase.  You can change a key's pass phrase by loading it in this GUI, editing the pass phrase field, and saving the key again.  This includes adding a pass phrase to a key that previously had none.

Once this is complete, you are able to save your keys to file.  Complying with the Unix convention, I saved my keys in a folder I created under my user directory called ".ssh".  Also, as different accounts on bitbucket require different public/private key pairs, I gave my key files fairly explicit names that labeled them as bitbucket keys for a particular user.

Now that you have your keys, you need to give bitbucket your public key so that it can encrypt the data that it sends you.  To do this, log into your bitbucket account and click on the "Account" link at the top.  On that page there should be a box with "SSH Keys" as its title.  Before you go copying the contents of the public key file into the text box, you need to clean it up a little so that bitbucket understands it.  This is explained in the first comment on the [official guide for setting up SSH on bitbucket](http://confluence.atlassian.com/display/BITBUCKET/Using+SSH+to+Access+your+Bitbucket+Repository).  I didn't check to see if this is necessary, but it works regardless and doesn't take long.


    
    <code class="brush: plain">
    ssh-rsa
    <encryption key all on one line>
    <email address>
    </code>



For example, a snippet from my public key for bitbucket is shown below.


    
    <code class="brush: plain">
    ssh-rsa
    AAAAB3NzaC1yc2EAAAABJQAAAIBmjcBE94aCAvlNCof4Svt...
    andrew@ajmccluskey.com
    </code>



Once you have formatted your key like this, send it to bitbucket via the form mentioned above and you should be ready to configure Mercurial.



## Configuring Mercurial


You should now have both a public and private SSH key for use with your bitbucket account, and bitbucket should have your public key.  Now you need to configure Mercurial to use SSH and to use your private key when talking to Mercurial.  Please note, most of these instructions were taken/borrowed/adapted from Atlassian's official guide linked to above.  Also, a lot, if not all, of what I'm about to take you through can be done in TortoiseHg's GUI, however I'm getting my hands dirty writing config files.  I find you learn more doing this, and in Mercurial's case, the knowledge is portable to other platforms.

First of all, let's set our user details so that Mercurial knows who is doing what.  To do this we need a Mercurial config file for the current user, so create a plain text file: `%USERPROFILE%\mercurial.ini`. `%USERPROFILE%` is a Windows environment variable that should, on most people's systems, point to `C:\Users\<current user>\`.  Open mercurial.ini in your [favourite text editor](http://notepad-plus-plus.org/) and add the following line, replacing my name and email with yours.


    
    <code class="brush: plain">
    [ui]
    username = Andrew McCluskey <andrew@ajmccluskey.com>
    </code>



To make Mercurial easier to work with, we're going to configure an alias for the address of our repository, as well as tell it the SSH command and private key to use.  This saves us from having to remember the full SSH path to our repository every time we interact with it, and allows Mercurial to use our SSH certificates.  For the purposes of this exercise we're going to imagine we're working with a repository called Happy Owls located at `C:\HappyOwls`.  We're also going to imagine that this is the repository you've already set up on bitbucket, and that the SSH address for this repository is `ssh://hg@bitbucket.org/user/happy-owls`.  To make the desired changes create/edit the repository specific configuration file that is located within the .hg directory for our repository (`C:\HappyOwls\.hg\hgrc`).  The following should be added to this file.


    
    <code class="brush: plain">
    [ui]
    ssh = TortoisePlink -ssh -2 -C -batch -i C:\Users\me\.ssh\bitbucket_key.ppk
    </code>




    
    <code class="brush: plain">
    [paths]
    bitbucket = ssh://hg@bitbucket.org/user/happy-owls
    </code>



The first of the above entries tells Mercurial what command to run when this clone of the repository interacts via SSH.  In this case we're using TortoisePlink, which is installed with TortoiseHg.  In order to specify the executable like this we must add the install path of TortoiseHg to our PATH environment variable.  Alternatively we could specify the complete path to TortoisePlink, but I'm lazy.  We could also use the plink executable that comes with PuTTY, including the same options.  Each option and what it does is specified below.



	
  * -ssh tells plink that we're using the SSH protocol

	
  * -2 tells plink to use version 2 of the SSH protocol

	
  * -C tells plink to use compression when transmitting data

	
  * -batch tells plink that it is being used in a script and cannot interact with the user

	
  * -i tells plink which private key to use



The second entry allows us to use the name "bitbucket" instead of the full path to the repository in bitbucket.  For example, the entry allows us to say `hg push bitbucket` rather than `hg push ssh://hg@bitbucket.org/ajmccluskey/storm-timer`.  Alternatively we could specify the alias as default, that is `default = ...`, and not have to specify a location at all.  This would allow us to say `hg push` and have Mercurial automatically push our change sets to bitbucket.  I prefer the verbosity of always specifying a location, but that's just me.



## Adding a Pass Phrase and Using Pageant


For the more security conscious among you, using a private key with a pass phrase is a good idea.  When this is done, the private key is itself stored as an encrypted file on disk.  Decrypting the key and loading it into memory requires the user to enter the pass phrase for the key.  As entering the pass phrase each time we use the key is tedious, we can instead do it once each time we start working with bitbucket by loading it into Pageant.  Pageant is a tool that loads private keys and keeps them in memory for use with PuTTY/Plink.  As the decrypted private keys are kept in memory by Pageant, we only have to provide our pass phrase once when the key is loaded.

To load a key into Pageant, simply start it on the command line and provide the path to any keys you would like it to load.  When it starts, it will ask you for the pass phrase for each key that requires one.  Keys that don't require a pass phrase are loaded silently.  Alternatively, once Pageant is started, one can right click the tray icon and select "Add key".  Loading keys at the command line means that all of the required keys for a given coding session can be loaded via one shell script, and the pass phrase for each key entered once.

Using Pageant also means that we no longer have to specify the private key as part of our SSH command.  As a happy consequence we can move the SSH command from our repository specific configuration file and add it to our user specific one, minus the -i option.  In my opinion this is a nicer way to go as we don't duplicate our SSH command/parameters for every repository, and our key management is separated from our Mercurial configuration.



## Now Start Coding!


You're now ready to start interacting with your bitbucket repository and get to what really matters: coding.  If you have any issues or questions, don't hesitate to add a comment, send an email, or bug me on Twitter.  Happy coding!
