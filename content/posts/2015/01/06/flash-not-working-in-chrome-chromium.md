+++
date = '2015-01-06T14:33:00+01:00'
draft = false
title = 'Fixing Flash in Chrome/ Chromium on Linux'
categories = ['Technology']
tags = ['Audio','Video','Utilities','Linux','Installation']
+++

![Flash not working](/images/2015/parliament_not_working.jpg)

When Linux works it works well, and when it doesn’t it can be a real pain in the backside. Usually though, the failings are not necessarily the fault of the OS, but more the fault of the way in which packages, snap-ins, extensions and other third party code is deployed. Granted, it doesn’t really help that there are numerous methods and *Package Managers* available to install applications, codecs and their ilk, but often the fault lies partially with the Third-party vendors for not making the process as easy and transparent as possible (especially for the less technical savvy amongst us).

I recently have had the privilege of deploying a new Linux Desktop for home use over the Christmas break and ran into a problem in Chromium playing Flash based Videos. The problem wasn’t present on initial setup, but was caused by my tinkering and reconfiguration for another Flash based issue on another browser. My problem surfaced after fixing my other issue (which incidentally was due to the browser configuration files stored in the home partition being inaccessible to my user account – fixed easily by the `chmod` command), I had gone a little crazy and uninstalled/ re-installed a selection of packages and hadn’t really taken enough care in doing so. Chromium was now broken (or at least, Flash videos were not playing in it). The Adobe site was no help with its usual Download page offering no suggestions for Chromium and simply providing a generic Linux package to download.

I got the standard “You need to install Flash Player to play this content” message when trying to play those videos and my first thought was that perhaps I had somehow uninstalled the flash codecs (despite the fact that Flash videos played perfectly in Firefox). Various attempts of uninstalling and reinstalling the flashplugin-installer package, along with the *chromium-browser* package and even the *chromium-codecs-ffmpeg* or *chromium-codecs-ffmpeg* still drew blanks. Rather frustratingly I took to Google and attempted to search for an answer, and while I found lots of different suggestions, none of them were the fix. I did however come across this rather old article titled [Adobe to Linux users: Get Chrome or forget Flash](https://www.computerworld.com/article/2501956/linux/adobe-to-linux-users--get-chrome-or-forget-flash.html). While the title gives the impression that this post is not going to give me the answer, it did give me enough information to figure out the solution.

The article describes how Adobe are no longer supporting Flash on Linux platforms, and if you are using Flash Player 11.2, then that is *it*. Instead, they have worked with Google Developers in order to utilize the Pepper Plugin API (or *PPAPI* for short). A quick scan through the Synaptic Package Manager allowed me to stumble across the *pepperflashplugin-nonfree* package with a description of *"This package will download Chrome from Google, and unpack it to make the included Pepper Flash Player available for use with Chromium. The end user license agreement is available at Google"*.

![Flash working](/images/2015/parliament.jpg)

The package sounded exactly like what was missing so I marked for installation and applied the change. Upon reloading the flash embedded page in the browser, I still sadly had the same message. Thankfully after closing and reopening Chromium, the problem was fixed!

As an aside, if I had wanted to install the package through the command line I would have simply typed:

```bash
sudo apt-get install pepperflashplugin-nonfree
```

I hope this saves you minutes (or hours) of frustration getting Flash media to work in Chromium on Linux should you be unlucky enough to be experiencing a problem. It is simple to fix when you know how!