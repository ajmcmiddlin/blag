---
author: Andrew
comments: true
date: 2015-01-03 21:12:09+00:00
layout: post
link: http://ajmccluskey.com/2015/01/executing-your-code-from-vim/
slug: executing-your-code-from-vim
title: Executing your code from Vim
wordpress_id: 591
categories: Guides
tags: vim
---

My girlfriend is currently learning Python, and using Vim as her text editor (because she's a boss). She told me this morning that she was quitting Vim to run her code each time she changed it, which was becoming annoying. I showed her a couple of methods to do this and thought I'd quickly document it for her and others. We both run Macs, and run Vim from Terminal/iTerm, but this should at least point you in the right direction if you're running a different OS. Also, the examples below use Python, but should work equally well for similar scripting environments like Perl or Ruby.

<!-- more -->



## A note on executables on Unix like operating systems


Before we proceed, I thought it'd be worth pointing out that there are two subtly different ways to run your scripts on Unix like operating systems. The first is to type the name of the interpreter (usually just the name of the language your script is in) followed by the script. This method tells the OS to start the script interpreter, and then send it your script as input.


    
    <code class="brush: bash">
    lappy486:~ andrew$ python my-code.py
    </code>



You may have also seen people just run their scripts as if they were normal programs like below.


    
    <code class="brush: bash">
    lappy486:~ andrew$ ./my-code.py
    lappy486:~ andrew$ /path/to/another-cool-python-script
    </code>



To make this happen you need to do two things.




  
  1. Tell the OS that the script is executable.

  
  2. Add a hint to the top of your script so that the OS knows which program to run your script with.



To tell the OS that the script is executable, simply run this at the command line. It's a standard Unix command to set the permissions on a file, and in this case it's giving your user execute permissions on your script.


    
    <code class="brush: bash">
    lappy486:~ andrew$ chmod u+x my-code.py
    </code>



Now, to let the OS know which program to run your script with, add this to the top of your Python scripts. It tells your OS to go find the python program, and then run the rest of the file with it. If you're using a different language, replace the `python` with whatever program you run your scripts with - e.g. `ruby` if you run your scripts with `ruby sweet-code.rb`. Please also note that you have to be able to run your scripts with `python my-code.py` for this to work.


    
    <code class="brush: python; highlight: [1]">
    #!/usr/bin/env python
    
    print "Hello, world!"
    </code>



Now, onto the workflow.



## Solution 1: Windows or tabs


The easiest solution is to have a second shell open in another window or tab. Your other window/tab is then only a keyboard shortcut away. Once there, you can execute your code with `python <my-code.py>`, or if you've already executed it once before, try pressing the up arrow to give you the last command for free.

If you're going the window route and you're on a mac, then Cmd+` is your friend for switching between windows in the same application. If you're using tabs, then Cmd+Shift+[ and Cmd+Shift+] will move back and forward between your tab list respectively. That should work in both iTerm and Terminal.



## Solution 2: Run a shell command from Vim


If you're new to Vim, it's entirely likely that you didn't realise you could execute shell commands from Vim. To do so just enter your shell command as an Ex command, but put a ! at the beginning. Don't know what an Ex command is? It's just one of those commands where you type a colon followed by your command - like :e to edit a file. If you're still confused, run `vimtutor` at a shell and then come back.

In our case, if we're editing `my-code.py` in Vim and we want to run it without leaving Vim, we can run any of the following Ex commands. Note that these all assume that you started Vim from the same directory that your script is in, and the second two assume your script has been setup as an executable. Obviously you can add the path to your script relative to where you started Vim from. If you don't know where Vim was started from, just run `:pwd` in Vim and it should tell you.


    
    <code class="brush: bash">
    :!python my-code.py
    :!python %
    :!my-code.py
    :!%
    </code>



Doing this repeatedly isn't great, as your previous outputs build up and things start to get ugly. To remediate this, we could add in a `clear` command first to blow away old output before each execution.


    
    <code class="brush: bash">
    :!clear; python my-code.py
    </code>



Much nicer. The inescapable downside of this approach is that we have to re-type this command every time we want to run our code. This can be improved by scrolling through our history of commands by entering Ex command mode (just type : from normal mode) and pressing the up arrow or Ctrl+p to move back one command in history. Still, it's a little cumbersome.



## Solution 2b: Turning our Ex command into a keyboard shortcut


One of the great things about text editors like Vim and Emacs is that we can script the crap out of them™. Rather than remembering the command above and having to type it at least once each time we start coding, let's just make it a keyboard shortcut. To do that, edit your `.vimrc` file by running `:e ~/.vimrc` and adding the following.


    
    <code class="brush: text">
    filetype on
    autocmd FileType python nnoremap <buffer> <F9> :exec '!clear; python' shellescape(@%, 1)<cr>
    </code>



That second line there is pretty scary, but all it says is: "for anything you think is a python file, add a keyboard mapping for the F9 key in normal mode that clears old shell output and then runs our current file with python". Restart vim and open up your python script, and now you should be able to press F9 whenever you want and it'll run it for you. Some other things to note about this are:




  
  * You may already have a line with `filetype ... on` in your `.vimrc` file, in which case you don't need another.

  
  * If you want to use a different F key, just replace the "F9" in the second line with whatever F key you want.

  
  * You can use keys that aren't F keys, but I'm not going to go into that.

  
  * Depending on your keyboard setup you might have to hold down a Fn or similar key to make your F keys work. I believe this is necessary on a default Mac setup because the F keys are generally used to control media playback and stuff like that.





## Bonus round: Run tmux


Given this post is aimed at beginners using Vim who just want a quicker, easier way to run their scripts as they're working on them, running [tmux](http://tmux.sourceforge.net) is almost certainly overkill for those users. Still, it felt remiss of me to not at least mention it because it's what I use. If you're manually running a script every time you update it, it's about as good as using a different window or tab. However, you can tail log files and and have other things running and keep an eye on them next to your code if you're working on bigger projects. I also like that it keeps your project contained to one OS window that you can quickly jump around using keyboard shortcuts, including jumping between different tmux sessions. If you feel like you're getting to the point where that sounds interesting, then I recommend checking it out.



## Bonus round 2: Script watcher


You can also get fancy and run a script in another tab/window/tmux panel to watch your script for you, and run it whenever you save it. If that sounds like a good idea, here's a hacky one I just wrote up. I call it _watchit_. Put it in a file called `watchit`, make it executable, then run it like any other script and just add the command you want it to run (with arguments) after the `watchit`. For example, `watchit ./my-code.py foo bar`, where `foo` and `bar` are arguments. It'll warn you if your code exits with a non-zero exit code too. It's probably buggy, so if you find any issues you can post them on [my github](https://github.com/ajmccluskey/misc-scripts), or leave a comment.


    
    <code class="brush: bash">
    #!/usr/bin/env bash
    
    script="$1"; shift
    last_mod=0
    
    while true; do
        curr_mod=$(stat -f "%m" "$script")
        if ((curr_mod != last_mod)); then
            clear
            printf "\nOutput of %s:\n\n" "$script"
            "$script" "$@"
            script_ec=$?
            if (( $script_ec != 0 )); then
                printf "\nWARNING: %s exited with non-zero exit code %d" "$script" $script_ec >&2
            fi
            last_mod=$curr_mod
        fi
    
        sleep 1
    done
    
    exit 0
    </code>



Happy coding!
