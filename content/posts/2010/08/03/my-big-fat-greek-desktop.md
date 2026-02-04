+++
date = '2010-08-03T22:40:00+01:00'
draft = false
title = 'My Big Fat Greek Desktop'
categories = ['Technology']
tags = ['Linux']
+++

**Disclaimer:**
Although this post may look at first to be about Linux and whilst many of you visiting this post will be either Windows or SQL centric you might think this is not for you. *Actually you are exactly the right people to be reading it because I want you to understand that you have options out there open to you*. Your desktop can be a blank piece of paper and you can draw what you want. You are not constrained by software boundaries if you do not want to be. My desktop is all about flexibility, speed and usability. I can do anything I want whether it be Linux, Windows, SQL, Oracle whatever on my beloved desktop.

This post also shows how you can use Linux and make it more usable, way beyond what you might expect from this humble open source OS. Go on, live a little!

# In the Beginning
I've recently been suffering from unexplained machine freeze for several weeks and decided enough was enough. Resolve or upgrade, but no more cold boots first thing in the morning. I'm getting ahead of myself though, let's go back to the start of the problem...

![Flip over desk](/images/2010/flip_over_desk.jpg)

Over quite a considerable period of time I had come to grow and love my GUI customizations of my Linux host (Mint 7) and it had everything. It looked slick, had little effects which looked cool such as the magic carpet effect when minimizing Gnome windows and it was fast. My Windows XP guest ran beautifully fast within its virtual environment (arguably as fast as native) and since I was addressing the whole 5GB of RAM (1.5GB more than my original 32 bit XP build could) I wasn't even losing out on memory).

I even migrated to a new Windows guest, a brand spanking new Windows 7 build and I was quite proud of my flexibility on my lovely desktop. I mean what could go wrong, if I didn't like it I could just use my old guest. Hell I could even run them together until I was satisfied the day had come to decommission XP. Things continued for a while in a state of Nirvana. Linux host, Windows 7 guest and a whole multitude of lovely things in the background. At this point I shall not discuss how I'd "merged" my Linux home and desktop with my Windows documents and desktop, but just be assured they got along just dandy. Two OS's working as one.

Over time something happened and I didn't know what. I would arrive in the morning to find the keyboard lights all on but unresponsive to touch (or hammer) and a blackened monitor. Initially I would just cold boot the machine in frustration and lose everything that was un-saved. Over time I learnt that the Linux host was still active and therefore installed the SSH Daemon so now first thing I could ssh in and save the state of my VirtualBox guest and then put the machine through a soft reboot. This continued for some weeks until the fateful day I told you about earlier.

After a little bit of investigation I noticed some strange errors in the messages log (and perhaps in a few of the other system logs) that indicated there was a problem somewhere in the region of the GUI. Evidence suggested that this was with the graphics card or graphics driver but I really couldn't be sure. Even the Linux guru I consulted believed that this might be the case, but we were both unsure.

Finally one day I decided to install Mint 9 (2 iterations up). Upon completion I put back in place various bits and pieces, such as Documents from my old home and some dot configuration files. Upon arrival the very next day, to my frustration a similar problem as before had occurred. Although this time the problem was a slightly more of a stable result. I simply had a message stating that my machine was running in low graphics mode and an option menu where I could choose to close X if I wanted to. I have seen this very same message on another of my machines and it always seemed to occur after around 5 hours of usage. I had guessed that this was because the Graphics card was starting to overheat, but I couldn't be sure. Anyway I now had two very similar problems on two different machines, one of which had been upgraded – which as I have said had an issue before. I was convinced that this issue could not (or would be unlikely to be) a hardware or driver related issue but perhaps something to do with configuration.

![Get out of my way 'cos there is no turning back!](/images/2010/getouttaway.jpg)

So I asked myself -on the presumption that my clean build resulted in a good working machine, what was changed to make it unstable. The answer was that I had replaced my .gconf folder in my new home with my previous one. It seemed a little too obvious to me but I thought I would assume this was actually the problem.

http://projects.gnome.org/gconf/

The .gconf folder contains various Gnome (Windows environment) based customizations and settings and is located in each Linux user's home folder. Now one thing I was absolutely positive about was that from time to time I had added the extra GUI tweak here or there to make my interface even prettier. That's one thing I love about Linux, it's customizability. With respect to the GUI, it is also its greatest downfall.

Anyway to cut a very long story short, the next morning my machine had not frozen – Yay! The day after that, it still hadn't failed. Better still my machine and all guests absolutely fly. I've been very careful about what I've altered, but with a few bits of tinkering here and there my desktop is back to looking damn sexy again.

# So why are you telling me this?
Well the first thing I want to say is that when you make customizations to any environment, you should make sure you remember that you have made them and always remember to have a "Last Known Good" backup available. Easier said than done I know, but it could have saved me a little pain had I remembered what I had changed all those weeks back and could easily revert what I had done.

The second point I want to make is related to the GUI itself on Linux. I am pretty sure that the settings I had changed which caused me the issue occurred after playing with the compiz manager setting lots of fancy effects. Be very careful of the compiz changes you make and always remember what you did.

The final thing I wish to discuss is how you can make customizations to your Gnome GUI in the first place. There are lots of changes to the settings you can make which will help your GUI do lots of the things you love about MS Windows, and lots more extra goodies. All these settings though are not in the same place, and I am hoping at very least this article will serve as a good reminder to myself.

The first GUI changes you should make are accessed on the Gnome panel, simply right-click the vertical line (the separator) near your Menu button and show desktop and select the Preferences menu item.

![Menu](/images/2010/cust12.jpg)

This will give you access to the *Window List Preferences* dialog, and I like to set the option to *Always group windows*.


![Windows List Preferences](/images/2010/cust2.jpg)

The next change you may want to make are the *Menu Preferences*. To access this, right-click the Menu button and select Preferences. The changes I like to make here are the icon sizes for the applications. Another change you can make is for the actual Panel itself. To access the *Panel Properties*, simply right-click empty space of the panel in question and select *Properties*. to get the dialog. I like to change the height of the panel (making it smaller) and making it slightly transparent. I also like to set the lower panel to autohide which ironically I don't like to do in Windows.

Some of other GUI settings that I use are accessed via the Menus. I use an Ubuntu Custom Menu (added as an item to a new top panel) to access these. First select *System/ Preferences/ Desktop* to access the *Desktop Settings* menu. Here you will find all the Window-like stuff, such as Desktop icons to add (which I do) and a few other customizations to set.

![Desktop Settings](/images/2010/desksettings.jpg)

Moving swiftly on to the desktop appearance, you will probably at some stage want to change the theme you are running on Gnome. To access this, simply right-click a blank part of your desktop and select Change Desktop Background. This will take you to the *Appearance Preferences* menu where you can alter lots of the lovely things you can see below. Generally I choose the theme you can see selected in the screen shot below and also change all of the Fonts to a smaller size, because I think the defaults are far too large.

![Change your theme](/images/2010/appearance.jpg)

Last for this post, but certainly not least you have the *Compiz Settings Manager*. This is a bit of a beast and you can do some weird and wonderful things in here such as the lovely magic carpet (and lots of other effects) that I have mentioned previously. You can also significantly slow down you machine or introduce an error to your config just as I had done, so please be careful and remember what you changed and take backups or copies of you system settings in your home directory. Access this again through *System/ Preferences* menu

![Be careful with your compiz](/images/2010/compiz.jpg)

# ...and finally
So now our work is done, my desktop is complete (for today at least) and its looking just like I want it. So I think it is only fair I present it to you.

![Welcome to my world](/images/2010/topbananaobf.png)

In the screen shot, to the left you will notice my lovely Windows 7 desktop. This is the machine that pays my bills because it's got lots of Microsoft goodies such as **SQL 2008 R2 client tools**, **Visual Studio 2010** and **Office 2010** to name but a few. With the simple click onto this machine and a press of my right `CTRL` key whilst also pressing the `F` key, this guest is fully maximized to fill the entire screen as if it was installed natively. The same sort of thing we are used to in VMWare Workstation or even Remote Desktop connection (if you can ever remember that key combination). Unlike VMWare Workstation, there is absolutely no sluggishness at all.

Next I would like to draw your attention to the machine underneath it. You would be wrong to think that this is another guest, it is infact a remote connection to another Linux machine over the WAN using an SSH tunnel using some lovely software that does VNC like (but so much better) desktop. Again I can maximize this entire machine so it looks and feels like I am there. This time I have to use the left `CTRL` and `ALT` keys whilst also pressing down (yes you guessed it) the `F` key. It's really nice.

Moving swiftly on, and looking across to the top right of the desktop you will see a machine called Neon. This is my little pet play thing which I have several different versions of SQL installed going right up to R2 which I use to help me take screen shots and also to trouble shoot tweets and test out any scenarios of my own that I may have. This machine is a guest, so I can pause it, save it's state in seconds and then close it if I don't want it around or don't need it. This usually happens when my pet play thing is misbehaving itself or needs a rest.

To the bottom right of the screen is a remote desktop connection (using the linux `rdesktop` command) to a random live work server. I tend to have a very nasty habit of having a lot of these, but I am currently in treatment and hope to keep them at a minimum sometime soon. Within my Windows 7 guest I might also have a couple of remote desktops going on, depending upon where the focus of my attention has been.

People I would like to welcome you to my big fat greek desktop! I hope you like it.