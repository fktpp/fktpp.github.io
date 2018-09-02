---
layout: post
title: Why not use neovim in windows
date: 2018-09-02
categories: 
- editor
tags: 
- vim
- neovim
- win
author: Kai Fan
excerpt: The Goods and Bads for neovim on windows
---


# Things I like neovim better

1.  The tab-complete options are shown as horiztal line just above the modeline, really cool.
2.  Yanking to windows clipboard seemlessly in the backgound, saves me lot of mouse clicks.
3.  The :checkhealth command


# Things I dont like

1.  filenames for nvim-qt.exe wrapper must be sperated by ****double
    double dash**** with its own options&#x2026; like this:
    
        c:\opt\neovim\nvim-qt.exe --maximized -- -- foobar.txt


# Things I can't live with

1.  The anonymous buffer is changed issue. I am not sure how to
    reprodcue it, anyway it is really anoying to run into it. It
    proven the process from exiting, neither :q! nor windows close
    button works. I have to kill the nvim process in windows task
    manager, then kill the nvim-qt wrapper.

