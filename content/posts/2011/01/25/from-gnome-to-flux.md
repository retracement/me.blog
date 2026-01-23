+++
date = '2011-01-25T07:31:00+01:00'
draft = false
title = 'From Gnome to Flux'
categories = ['Technology']
tags = ['Linux','Fluxbox']
+++

In between the time I am looking into all things SQL and all things Microsoft I am usually tinkering in Linux, and this usually consists of tweaking my user desktop environment so that I may have a more productive and enjoyable experience – you might recall the post [My Big Fat Greek Desktop](/posts/2010/08/03/my-big-fat-greek-desktop).

![Flux Menu](/images/2011/my_menu.png)

I knew the time would come when I would finally move on from the [Gnome](https://www.gnome.org/) desktop on my trusty Linux boxes, and I had toyed with that idea several times without the jump. Over the last few weeks and months I have been trialling [Linux Mint Fluxbox](https://blog.linuxmint.com/?p=1523) edition and to be honest I really didn't like it at first. One of the things that struck me was the simplicity of it, but then that was the whole point for the move. Simplicity equals speed and speed equals productivity as long as I could navigate my way around. One of the things that I first hated about this desktop that I now love, was the right-click behaviour on the taskbar. For instance if you right-click any running application on the taskbar, a close signal is sent to the application. At this stage you might be thinking that this would drive you mad and believe me for a small amount of time it did -until I started to realise actually how many times I right-click applications in Gnome or Windows and then do something with the context menu -which is usually always to close anyway. This action has now been reduced to a one-click operation, and I find that I am not constantly minimizing or maximizing windows all the time because I can close applications very quickly and tend to do just that. The other part of the equation (opening applications) can also be performed very quickly by moving the mouse to any part of the blank desktop, some of which can always be found at the bottom left and bottom right of the taskbar and right-clicking. This initially was a real pain until I discovered that the Fluxbox menus can be easily edited and customized to your requirements via a simple text edit of the menu files under the .fluxbox directory in your home.

The removal of a quick launch style toolbar at the top of my desktop has resulted in more screen space for the windows -and before you suggest that is what auto-hide is for then think again please because for me auto-hide is one of the most visually distracting features of Gnome and Windows and slows the launch process down. I believe now I have the best of both worlds, the ability to launch an application quickly and the full use of the screen (if you ignore the taskbar at the bottom).

There have been a few problems along the way. For instance, the ever reliable VirtualBox installation has had problems as you will see in the screen shot below.

![VBox problems](/images/2011/vbox_error_install.png)

This issue can be overcome by installing dkms by simply typing `sudo apt-get install dkms` in a terminal. The next time you attempt the VirtualBox install, this time it goes through cleanly (phew!).

![Vbox succeed](/images/2011/vbox_succeed_install.png)

Another quite annoying issue that I have found is occasionally I will end up with Windows that have their tops cut off the screen. I have somehow managed to drag them off and it is incredibly frustrating trying to get them accessible again. In other-words, if I cant access the tops then I can't get to the maximize, minimize and close buttons! As stated earlier, close is not a problem but the other two are. Thankfully without key customization, even this is not a problem since there is an incredibly simple way to grab the window and move it. The shortcut is to hold down *Mod 1* (usually *Left ALT* key) and then click on the window in question with the *Left Mouse button* which will allow you to then drag it anywhere you choose.

Along the way I've also found a few niggles with various applications I've installed, and usually always they are related to missing packages (since the Fluxbox edition is NOT bloat-ware). However I did encounter a surprising issue when I installed and tried to use [AdobeAir](https://get.adobe.com/air/) and [TweetDeck](https://www.tweetdeck.com/). IT-NO-LIKEY! Actually the workaround is rather simple, once you know it. You simply have to fool TweetDeck into thinking you are running the GNOME desktop. First thing you do is create a batch file with the following contents:

```bash
export GNOME_DESKTOP_SESSION_ID=1
/opt/TweetDeck/bin/TweetDeck
```

Of course you now need to make the shell script executable and set-up a short-cut to it in your menu file, but once that is sorted….. its Twitter time!

Something else I need to seriously look into is the fact that Linux Mint Fluxbox edition is only released as an x86 edition and I *really* want to be running x64. So the workaround is (apparently) to install the lightweight [Linux Mint x64 xfce](https://www.linuxmint.com/edition.php?id=63) release and then install the Fluxbox Desktop. I am sure with a bit of tinkering this will all work just dandy and I hope to attempt it at some point.