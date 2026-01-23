+++
date = '2010-04-18T06:55:00+01:00'
draft = true
title = 'Learning is not a bad thing, dont be afraid to know too much'
+++

THIS POST NEEDS REWRITING

I recently had a quick discussion with a fellow DBA about learning new technologies. He had said that he only does SQL Server work and that he was a *specialist* not a *generalist*.

The task in hand was simply to promote an Oracle script and I had offered to show him what to do, so I so a little taken aback at the unwillingness to learn something new. This feeling of surprise was further compounded after subsequently meeting with an ex work mate and his colleague at the SQL Bits event in London a few days later. The subject came up as to what exactly I did, to which I frivolously responded with a witty *"good question and when I find out I'll let you know :)"*. More seriously I went onto say that I do the same job as my ex colleague (primarily SQL Server DBA work) but also perform lots of other activities such as development (primarily C#). I heard the same phrase repeated, *"better to be a specialist than a generalist"*.

Moving the discussion the conversation on, I started discussing how good Linux was becoming -particularly Debian based builds and that how disappointing Vista had been. My colleague's friend responded that Linux was rubbish because he hated the command line, likes GUIs, had a nightmare installing the release after Gutsy Gibbon (Hardy Heron) ON Hyper-V and that Microsoft was going to batter Linux in the years to come.

My own personal feeling is that Command lines are great, and if you have a strong historic Microsoft background you should have learnt to love the command line. Why you might ask? Well one reason is that it helps you to learn the full extent and power of the environment.

![How I Organise My Desk](/images/2010/howiorganise.jpg)

When recently I said to someone how great the SQL Server Management Studio GUI was they replied, "you mean that nice looking GUI that wraps around the command line you use!"….Er yes actually they are mostly correct. In my day to day SQL Server activities I tend to quickly launch new Query tabs and very quickly type a command to perform some operation that through the GUI would take either longer to action OR will cause various additional operations on the SQL Server or Database that you was not expecting (these are well documented). Another recent example was me asking another SQL DBA to launch SSMS in another user context in Windows 2008. Unfortunately they didn't know what to do because "the Run As option looks like it is removed in Windows 2008". From the command line I ran my usually runas command and we were in business.

Secondly the beauty of the command line is that you can batch operations therefore allowing extensibility -which is surely the whole point of modern day computing, making the computer perform the repetitive operation?! Try extending the GUI natively and I think you will find that quite difficult to do without resorting to command line/ input commands.

*"like GUIs"* – GUIs are great, in fact GUI's are excellent. They are however pointless on there own for a serious professional. GUIs are there to make you life easier but there will always be a point at which the GUIs ability to give you exactly what you want ends. There are times however when a GUI is not necessary all the time, and this was once understood at Novell during the earlier days of NetWare. The most obvious reason is that a GUI uses a lot of system resources, so why is it necessary on a server? To dismiss Linux for GUI reasons is not very easy to understand by someone who understands Linux, because X is the most customisable GUI interface there is and has numerous windows managers to choose from unlike Microsoft Windows.

Some are more resource intensive, some very lightweight, or you can go with none at all.

*"nightmare installation"* – yep we have all had these scenarios but it doesn't happen just on Linux, in fact the same is true on Windows. Obviously a massive commercial operation that Microsoft is generates a much higher rate of vendor support and that is primarily the reason why Windows installations tend to be less painless. However most recently I have noticed the Apps team having all sorts of problems with their Windows 2008 installations due to driver support and so that demonstrates to me the problem is not unique to Linux. Secondly I would also suggest that on newer hardware the most recent builds of Linux (for me anyway) have installed seamlessly and the only problems issues I ever really have these days surround OpenGL configuration (or in other words games playing!). If you are not really interested in gaming then I would be very surprised if you are hitting any major problems with any of the major releases. The development release cycle of Linux is also much much faster and so if one distribution didn't work the successor might. Failing that there are numerous distributions to try, You will find one that works seamlessly if you try. Most of the Linux installers are now all GUI driven and provide wizards and for me are much easier and quicker than Windows. Another bonus is that due to the partition structuring and file structuring of Linux you can very quickly destroy and overwrite your System leaving your home partition in place, install a clean copy, install any apps you require (very quickly from the package manager – unlike Windows) and map your old home partition back in place. All your configurations remain intact! Try doing the same with any version with Windows and getting your build back together will take you the best part of a day.

*"Linux on Hyper-V"* – oh come on, you cannot seriously suggest to me that just because you have had a nightmare installing Linux on Microsoft's virtualization technology that there is a problem with Linux!! How about on VirtualBox, how about VMWare or on the other numerous virtualization environments?

*"battering Linux for years to come"* – the stats really don't support this theory do they? Windows 7 really does look to be much better than Vista, but Microsoft has consistently lost OS market share over the last two years.

![OS market share](/images/2010/marketshare.jpg)

Every other OS has persistently gained in varying amounts. Although the losses and gains are small, they are very significant due to the power that Microsoft wields and the pre-installed nature of new hardware. My guess is there will continue to be further continued decline in market share over the years and I think this will only benefit Linux as people realise there is not just one OS in the market place.

Let us not forget that Linux is free!

http://marketshare.hitslink.com/os-market-share.aspx?qprid=9&qptimeframe=M&qpsp=111&qpnp=25

But anyway I have digressed a little from my subject of specialisation against generalisation, but the point is that there is a big different between being a specialist in one thing as opposed to being a specialist in perhaps several skills with a generalist knowledge of others. For instance if you don't understand RAID (subject:hardware design), how can you hope to understand or design and troubleshoot efficient SQL IO subsystems? I have lost count of the amount of times I have tried to educate fellow DBAs in RAID If you don't understand Active Directory, how can you possibly hope to understand SQL Server security – for instance how does SQL Server authentication become more available than Active Directory?

So yes we all need to try a little harder to be more specialist, but if the cost of this is to ignore your broader general knowledge then I believe you haven't quite grasped the whole point of Information Technology. To demonstrate I shall give you a final example. For years you have worked on a development language and you pretty much can do anything that you could ever need to. You ignore new languages one by one as they are designed and released since as far as you are concerned you do not require them. One day you watch in awe as someone puts together a GUI interface in minutes written in perhaps C# or Swing to display a record set. You suddenly realize that writing Assembly perhaps is not such a great idea any more.

By knowing a little about many systems we can learn a lot about a few.