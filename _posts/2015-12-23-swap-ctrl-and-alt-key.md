---
title: "Swap Ctrl and Alt Key"
layout: post
comments: true
tags: keyboard windows
---


I use to use Emacs key-bindings(now I use vim keys) and intended to avoid emacs users' [pinky problem](http://ergoemacs.org/emacs/emacs_pinky.html) at the first place. Thus I forced myself to rebuild muscle memory for ctrl-alt-swapping in qwert keyboard.

<div class="message">
you should try `Sharpkeys` app first for key remapping purpose, unless Sharpkeys does not meet your demand or you are eager to learn the lower level.
</div>

###What I Want:
1. In Windows system, swap both left and right sides of Ctrl and Alt keys, i.e. Press Ctrl would send Alt, press Alt would send Ctrl.
2. I do not use menu key at all, so I map menu key, which sits on the left of RAlt key in my Logitech Deluxe 250 keyboard, to RAlt in order to allow my right hand palm comfortably to hit Alt.

###Solution:

1. save the below content into a reg file, e.g. swap_keys.reg, and double click to import it.
2. log off and re-log in to take it into affect. Note. no need to reboot, log off suffices.

**Warning: you should modify this file to meet your own requirement, unless you have the same desire as I have - "1. swap both left and right sides of Ctrl and Alt keys, 2. remap menu key to RAlt key"**
<div class="message">
Windows Registry Editor Version 5.00
 [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
 "Scancode Map"=hex:00,00,00,00,00,00,00,00,06,00,00,00,1d,00,38,00,38,00,1d,00,1d,e0,38,e0,38,e0,1d,e0,38,e0,5d,e0,00,00,00,00
</div>


* In order to understand what above does, I recommend you to check these articles:
[article one](http://www.northcode.com/blog.php/2007/07/25/Securing-Windows-For-Use-As-A-Kiosk),
[article two](http://www.howtogeek.com/howto/windows-vista/disable-caps-lock-key-in-windows-vista/)

* If you want to remap special keys your specific keyboard offers, you should look at this [article](http://www.win.tue.nl/~aeb/linux/kbd/scancodes-6.html).

###Long story, what challenges I encountered

In Linux (mine is Ubuntu), it is fairly easy to use Xmodmap to do key remapping. I found, however, that it is not such a trivial task as I thought it be in Windows system (Win7 in my case).

Autohotkey was recommended for windows user by [Xah Lee](http://xahlee.info/mswin/Windows_keybinding.html), but I found challenging to

1. enable sticky key feature for remapped keys. Autohotkey is not sticky-keys feature friendly.
2. tinker it to accept two modifier keys simultaneous, e.g. C-M-h is interpreted wrongly as M-h in Emacs if one or both of Ctrl and Meta/Alt keys used here are remapped ones.

In addition, `Sharpkeys` is a good GUI for remapping keys, but it cannot understand keyboard with AltGr, which unfortunately is the case for my Finnish/Swedish layout keyboard.
