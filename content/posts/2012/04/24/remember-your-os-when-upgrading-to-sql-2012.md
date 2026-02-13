+++
date = '2012-04-24T19:18:00+01:00'
draft = false
title = 'Remember your OS when upgrading to SQL 2012'
categories = ['Technology']
tags = ['SQL', 'Upgrade', 'Windows']
+++

Probably as far back as the end of last year when I first started putting material together for my Enter the Dragon and Moves like Jagger presentations and giving very early mash up presentations of them combined together, I started emphasising one very important upgrade consideration which is very often ignored. It is also a fact that is quite nicely touched upon by Glenn Berry ([twitter](https://twitter.com/#!/glennalanberry)|[blog](https://sqlserverperformance.wordpress.com/)) in his excellent book SQL Server Hardware and if you have yet to discover its unique delights then I strongly advise you to check it out.

The consideration I am talking about is of course the mainstream support dates of our Hardware and Software but most specifically the Operating System (in our case Windows) which is the one of most concern.

![Mainstream support](/images/2012/mainstream_24hoursofpassppt_moves_like_jagger.jpg)<br/>
*Spot the problem?*

During my Moves Like Jagger – Upgrading to SQL Server 2012 presentation (most recently given to [24HOP](https://www.sqlpass.org/24hours/spring2012/SessionsbySchedule/SessionDetails.aspx?sid=2562)) I have discussed that should you decide to upgrade to SQL Server 2012 then you should watch out for the mainstream support end dates for Windows 2008 and Windows 2008 R2. If you are installing SQL 2012 to Windows Server Core then your only consideration anyway is Windows 2008 R2 (SP1) otherwise using Windows 2008 (SP2) is a supported Operating System. However from a mainstream support perspective the version of Windows you use is kind of irrelevant since (as you can see in the slide above) each expires on exactly the same date -surprisingly mid July next year!

Yes people you heard me right, should you choose to upgrade your environment to SQL 2012 today, tomorrow or have already done so, then I hope you have taken this into consideration -for if your company are not prepared to pay for Extended Support then you could be looking at yet another upgrade (this time the OS) sometime next year. This scenario is one very strong reason why if you have implemented SQL 2012 in an Availability Group or Clustered configuration you can probably give yourself a pat on the back right now. It is going to simplify your OS upgrade and maintain a level of High Availability where this is required.

![Beautiful cottage](/images/2012/beautiful-cottage.jpg)<br/>
*Beautiful now, but rebuild costs could be large!*

At the turn of the year I went slightly out on a limb by saying that because of the Mainstream Support expiry dates it was my firm belief that Microsoft would release their newest Operating System (I called it Windows 8 Server at the time 🙂) before mid July and that if you did not have a compelling reason to upgrade to SQL 2012 now, then it is probably a good idea to wait a while.

Well so far my guesstimate appears to be coming true. We now know that the next Server release of Windows is called Windows 2012 (I personally like the fact it is in-line with the Server based Application names) and  today I have seen the following url- [Windows Server 2012 Release Candidate Timing](https://blogs.technet.com/b/windowsserver/archive/2012/04/24/windows-server-2012-release-candidate-timing.aspx?). So we know by the very name that the OS is coming this year and at the time of writing all indications point to an imminent release but let me stress that I have no inside knowledge and this is all guesswork so please don't get upset if the release happens on the 31st of December 2012 -however unlikely!

In summary I believe you should take three approaches to implementing SQL Server 2012 at this present time:

1. The first and probably my most recommended option would be to wait for (hopefully only) a few more months if you can until the release of Windows 2012.

The second and third options are to plan for the Mainstream Support end of Windows 2008/R2 by:

2. Arranging and paying for Extended Support.
3. Implementing SQL 2012 in an HA configuration that enables you up blow away your base Operating System and start again (for instance using AlwaysOn or even Peer to Peer Replication are possible routes you can take).

Whatever you do, don't forget the obvious... and remember your dates.