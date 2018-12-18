---
layout: post
title: Erlang runtime tools not found
date: 2018-12-18
categories: 
- erlang
- tips
tags: 
- erlang
- tools
author: Kai Fan
excerpt: Appmon, pman and etc. are replaced by new observer
---


# Appmon:start(). failed in erlang 21.0

I am learning Elang OTP in these days, while trying to launch Appmon
in Erlang shell I got a failure:

    1> appmon:start().
    * 1: syntax error before: descript
    1> 

How could it be? This is supposed to be one of the Erlang system
bundled runtime tools!


## Okay, the tools got a revolution in Erlang 17, as described

in [release note](http://erlang.org/download/otp_src_17.0-rc1.readme):

    --- appmon --------------------------------------------------------------
    
       OTP-10915  Removed gs based applications and gs based backends. The
    	      observer application replaces the removed applications.
    
    ...
    ...
    
    --- pman ----------------------------------------------------------------
    
       OTP-10915  Removed gs based applications and gs based backends. The
    	      observer application replaces the removed applications.
    
    
    --- toolbar -------------------------------------------------------------
    
       OTP-10915  Removed gs based applications and gs based backends. The
    	      observer application replaces the removed applications.
    
    
    --- tv ------------------------------------------------------------------
    
       OTP-10915  Removed gs based applications and gs based backends. The
    	      observer application replaces the removed applications.

