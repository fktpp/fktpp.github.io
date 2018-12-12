---
layout: post
title: Junction as symbolic link on windows 7+
date: 2018-12-10
categories: 
- tips
tags: 
- link
- junction
- win32
- symbolic
author: Kai Fan
excerpt: Create Junction to simulate the symbolic link to directory behavior
---


# Why?

It's all about the vim on windows thing.


## Gvim and vim-in-git

The official vim for windows (namely Gvim), compiled for win32 system,
looks for user vim packages from windows style directory
\`$HOME/vimfiles\`. 

While the vim bundled with git for windows, which was compiled by
mingw64, looks into a Unix style directory \`$HOME/.vim\` for the same
thing.

The diverse of user packages will not cause any trouble, if you don't
have any vim related customization for yourself like me does. Things
get changed until recently VIM upgraded to 8.0.


## The new vim packages system

The new vim comes with its own package management system namely vim
packages.

This package system makes it simple to install third-party vim plugins
by just clone github plugin repo into the vimfiles directories.

-   SOMENAME/pack/PACKAGENAME/start (for ftplugins)
-   SOMENAME/pack/PACKAGENAME/opt (for colorscheme plugins)


## How to merge the two vimfiles

1.  Solution 1: Make use of multiple repo management software such as
    \`repo\` or \`gclient\` to register and manage all the vim plugin repos
    on various directory as a whole.
2.  Solution 2: Manually maintain the repos in side a single
    place. First, combine home directory by set environment variable
    %HOME% with content of %USERPROFILE%. Then, make a **Symbolic
    link** \`.vim\` to \`vimfiles\` (which contains all the vim packages)
    directory.


# How to create symbolic link?

If you are using Win7, as I do. Just download and install the
awesome [Link Shell Extension](http://schinagl.priv.at/nt/hardlinkshellext/linkshellextension.html) and follow the instructions in [The
Complete Guide to Creating Symbolic Links (aka Symlinks) on Windows](https://www.howtogeek.com/howto/16226/complete-guide-to-symbolic-links-symlinks-on-windows-or-linux/)

Or maybe using the \`mklink\` ****builtin**** (cmd.exe)

    $ mklink
    Creates a symbolic link.
    
    MKLINK [[/D] | [/H] | [/J]] Link Target
    
    	/D      Creates a directory symbolic link.  Default is a file
    		symbolic link.
    	/H      Creates a hard link instead of a symbolic link.
    	/J      Creates a Directory Junction.
    	Link    specifies the new symbolic link name.
    	Target  specifies the path (relative or absolute) that the new link
    
    $ mkdir vimfiles
    
    $ mklink /j .vim vimfiles
    Junction created for .vim <<===>> vimfiles
    
    $ dir | findstr /i vim
    12/12/2018  10:07 AM    <JUNCTION>     .vim [C:\Users\fktpp\vimfiles]
    04/19/2016  01:05 PM             1,636 .viminfo
    12/12/2018  10:07 AM    <DIR>          vimfiles

Lesson learn by failure attempts:

1.  Windows shortcut is not symbolic link. It acts as a normal file
    if you try to access the shortcut inside git bash.
2.  \`ln -s\` command in git bash will not create symbolic link on
    windows, it merely create a copy of directory tree -\_-
3.  There's no official command to create a Junction (think about it
    as analog of symbolic link to directory in Unix) on top of NTFS
    until you upgraded to windows 10.

