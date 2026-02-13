+++
date = '2010-11-30T10:56:00+01:00'
draft = false
title = 'The fantastically forgetful IP stack'
+++

So the first thing you are probably wondering is... *does this post have anything to do with SQL Server?* Well if you have ever listened to my views on specialization and generalization you may be aware of the fact that I strongly believe that we all should be running our own sand-boxed virtualized environment distinctly separate from our corporate virtual environment and you (the humble DBA) will be your own Domain and Server admin -master of your own educational destiny. I am of course talking about one of the my key concepts that I like to practice myself, namely trying to be what I term a DBA 2.5. More on this though another time.

![No one is there](/images/2010/noman.jpg)

Is this post relevant to you? It depends. Do you care about connectivity to your SQL Server and if you come across an issue obviously outside of the SQL Server application domain will you take ownership of the problem or pass the buck to the Windows admins? If it is the latter then now is the time to stop reading...

...If you are still with me then congratulations because what I am going to discuss is a little problem that I have seen a few times over the last couple of years and relates to a rather nasty and annoying problem where the Windows IP stack decides it is going to forget the statically configured IP address and give your server a horrible auto configured address! This means that the connectivity to your SQL instance will obviously be severely hampered 🙂.

This problem is a most unusual one to troubleshoot because the behaviour being displayed really shouldn't happen. The static IP address is just that... *STATIC*, so the fact that Windows is deciding to auto configure it is rather infuriating to say the least. If you view the IP configuration of TCP/IP through the GUI, all looks absolutely fine. However when you view the configuration through `ipconfig /all` a different story appears.

![IP gone wrong](/images/2010/ipgonewrong.png)

So what did I try to do next? Well since the IP configuration looks fine in the GUI I decided to swap the configuration over to DHCP and then back again to static. On the switch to static I re-typed the desired static IP information. After this takes, running `ipconfig /all` seems to show that the problem has been resolved.

![Correct IP](/images/2010/correctip.png)

Upon reboot of the server the same auto-configured address raises it's ugly head once more. Try this a few times and see the frustration mount! Luckily I came across a few articles that discussed the same sort of problem which lead me to the following KB article:

> The Internet Protocol (TCP/IP) Properties dialog box displays the default IP address settings after you manually configure a static IP address in Windows 2000 Server or in Windows Server 2003" [http://support.microsoft.com/kb/937056](http://support.microsoft.com/kb/937056)

The first few registry keys listed in the article could not be found on my server. My test server is a Windows 2008 Enterprise machine running SQL Server 2008 Enterprise and was also acting as a Domain Controller. I therefore followed the article's advice for Domain Controllers and removed the network adapter device from Device Manager.

After putting my server through a reboot and entering the correct static IP information in the TCP/IP properties, subsequent reboots did not result in this issue reappearing.

A simple fix to a rather annoying problem, and if you stuck with me through this post, you will now be able to tell your Windows Admins how to fix it!