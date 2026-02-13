+++
date = '2014-02-11T12:10:00+01:00'
draft = false
title = 'Read your old CHM files from network storage on Windows'
categories = ['Technology']
tags = ['Books','Learning','Utilities','Windows']
+++

I was just recently going through my old SQL Server documents and reference material and attempting to organize and tidy things up, when I re-discovered a huge collection of literature that even now is incredibly useful. The only problem is that many of these old electronic books and files are stored in *CHM* format. Those old timers of you will remember that this was how SQL Server Books online was deployed up to and including SQL Server 2000.

So I tried opening one of these documents and I got the classic (and expected) symptom of a totally blank document body pane.

![Book](/images/2014/addison_blank.png)<br/>
*Try reading this sucker!*

Now this wasn't really a big surprise since the restriction was actually put in place by Microsoft many years ago to prevent content loading from remote and untrusted sources (i.e. the Internet or Network/ UNC paths) and resulting in an infection to your machine. In the past I seemed to remember that it was a simple registry hack, and this time around various searches across the internet were again suggesting that this would be the case. Unfortunately no single article  appeared to join all the dots but after a frustrating period of time I finally managed to figure it out again. This fix works on Windows 8/ 8.1 and should also work on prior versions of Windows going back to XP but I have not tested it on these platforms.

I have decided to blog it, not only to help anyone else struggling getting your remote CHM files working, but also so I don't have to waste one single minute more trying to figure out what the fix is. Assuming you trust the Remote source and the files therein, you have two choices:

1. Copy the remote CHM file/s locally.
1. Implement the fix described below.

Since I hate having to duplicate reference material, I want to implement the fix. So the first piece of advice you will see quite frequently if you perform a Google or Bing search for this problem, is that you need to add the following registry key into your machine:

```text
Windows Registry Editor Version 5.00
 
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\HTMLHelp\1.x\ItssRestrictions]
"MaxAllowedZone"=dword:00000001
```

Now the important line here is the MaxAllowedZone DWORD (line 4) created under the ItssRestrictions registry key. I first struggled to find any description of what MaxAllowedZone value did, but the following values describe the purpose of each:

```text
0 = My Computer
1 = Local Intranet Zone
2 = Trusted sites Zone
3 = Internet Zone
4 = Restricted Sites Zone
```

It was only really when I discovered these meanings that the next step fell into place. Essentially these values are setting the maximum zone in which you trust the remote CHM content. So a setting of 1 essentially tells Windows to allow any CHM content that is considered to be part of the Local Intranet Zone (and by default this will not include your network paths). So setting a value of 1 by itself would not (in this case) solve your issue. We could in fact change the value to 3 (Internet Zone) which would fix the problem since this is the Zone that my remote drive is considered to be part of, but that is obviously putting my machine at risk.

![MaxAllowedZone key value is set to a value of 1](/images/2014/reg.png)<br/>
*MaxAllowedZone key value is set to 1*

The solution then is to ensure that the *MaxAllowedZone* is set to a value of *1* and to ensure that your network path is added into the *Local Intranet Zone* and we do this through Internet Explorer:

1. Select *Internet Options* via it's Tools menu (Alt+x).
1. Click the *Security* tab and click on the *Local Intranet Zone*.
1. Next click the *Sites* button and then click the *Advanced* button from the *Local intranet* dialog.
1. Ensure that you deselect *Require server verification (https🙂 for all sites in this zone* option.
1. Finally all you need to do is type the path to your network drive. In my case I have mapped the network path to my *Z:\\* drive, so I simply type Z:\ in the "Add this website to the zone:" textbox. The path will be resolved to its real hostname.
1. Click the *Close* button and click any resulting *OK* buttons and close Internet Explorer.

![Internet Options](/images/2014/sites.png)

Now when you try loading your file all of the content loads! I now have Ken Henderson on tap again!

![Ken Henderson](/images/2014/addison_page.png)<br/>
*I want you back for good!*