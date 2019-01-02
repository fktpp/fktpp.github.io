---
layout: post
title: Fira code fonts mess-up in Intellij IDEA
date: 2018-12-18
categories: 
- tips
- tools
tags: 
- fonts
- idea
- intellij
- mess
- fira
- code
author: Kai Fan
excerpt: Recover the Fira Code for beautiful font ligatures after font mess up
---


# The beautiful, open source, with ligatures, font Fira Code

Some month ago I see someone talking about best fonts and themes for
their Intellij IDEA setup. One of the setups is really good, it was
my first time to know that symbols such as \`<=' or \`->' can be
display as beautiful arrows at runtime by supported software and a
supported font.

The font that mentioned was [Fira Code](https://github.com/tonsky/FiraCode) 


## Install FiraCode and fonts mess up in various programs

I do like the beautiful ligatures. And, [\`choco](https://chocolatey.org/) search fira' shows
that the font can be easily installed by chocolatey, Great!!!

Anyway, things is not as good as I expected. Fira code fonts seems
not work well in my Windows 7 ClearType environment. 

It shows up really ugly in my Emacs buffer, even worse, it will
mess up totally in J2SE Swing program if subpixel hint is turned
on.


## Recovery method

I tried several recovery methods, they are either no help or fail
back to old situation after one or two cycle of windows reboot.

-   remove/reinstall

-   manually remove fonts from windows fonts dir

    -   Fira code fonts got default medium style after reinstall.
    
    -   removed extra fonts? not sure if font awesome conflicts with Fira Code.


## revert back to \`Consolas' after all

Sadly, this is what really helps at last. Without beautiful
ligatures, but at least I got clean and beautiful fonts displayed.

