---
layout: post
title: Cross SDK is python26 incompatible
date: 2018-07-27 14:07:06 +0800
categories: memo fun
---

As Title
========

While this is supposed to be true, at a glance of the error
message. It is really difficult to find a community prof in
mail/forum-loop.

Note if I come back in the future.

**The error message**

    [ericsson@GitGSC chromiumos]$ cros_sdk
    Traceback (most recent call last):
      File "/opt/chrome/ChromiumOS/chromite/bin/cros_sdk", line 77, in <module>
        from chromite.lib import commandline
      File "/opt/chrome/ChromiumOS/chromite/lib/commandline.py", line 26, in <module>
        from chromite.lib import constants
      File "/opt/chrome/ChromiumOS/chromite/lib/constants.py", line 410
        'ARM_USERDEBUG',
                       ^
    SyntaxError: invalid syntax

**The [google group thread](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-os-discuss/Gg9tEhSYK2E) talking about python version things**

> Bernhard, 
> 
> The "cros_sdk" script uses shebang "/usr/bin/python2" (see 
> chromite/bin/cros_sdk). Perhaps that's why /usr/bin/python is always 
> invoked. 
> 
> You're pretty much on your own out there, though. I do not think we 
> want to go back to support 2.6. In fact, some scripts have even gone 
> to 3.4 now. 
> 
> Cheers, 
> Nam


And you'd better upgrade your distributed python
------------------------------------------------

Alas, the cross SDK will do lots of sudo during its execution, simple
PATH hack for the running user will not go to much further.

