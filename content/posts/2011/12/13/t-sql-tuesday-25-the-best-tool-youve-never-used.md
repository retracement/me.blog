+++
date = '2011-12-13T22:00:00+01:00'
draft = false
title = 'Visualising disk space usage'
categories = ['Technology']
tags = ['Windows','Utilities']
+++

![T-SQL Tuesday](/images/2011/tsql2sday150x15032.webp)

Wow, it's been a long time since my last T-SQL Tuesday post and I have disappointingly missed it 5 times for one reason or another. Still, the important thing is that I am back people!

This month's T-SQL Tuesday is brought to you by Allen White ([blog](https://sqlblog.com/blogs/allen_white/default.aspx)|[twitter](https://twitter.com/SQLRunr)) who many of you will know for his PowerShell, SMO and SQL Server expertise.

---

Ironically for this post (or the subject at least) I was saving up towards a larger Data Visualisation article that I was putting together, but when I read the subject of this months T-SQL Tuesday I just knew it was time to spill the beans.
# Let me give you the backstory...

Around four years ago, week in and week out I used to religiously buy a Technology Publication just on the "off chance" that they would have something of use in their pages. Very occasionally a little gem would crop up and eventually one day I found what I was looking for. This particular little gem used a rather ingenious way to visualise folder and file space usage and most importantly it was free and fast.

One of the biggest problems I used to experience as a production DBA was the filling of various disk drives. From backups, log, data, tempdb and even system drive we would get it all and having a substantial amount of servers made it a common problem.

When these issues would occur I was very happy to receive them because I knew I could identify the problem area in minutes (if not seconds) and it was always very satisfying to whip out my secret tool (ooo errr missus!)

# So why is it so good...

There are of course lots of other space analysis tools on the market, many commercial and many shareware or even free. But most of those I have seen have usually been slow, used poor visualisation and mostly required a local or remote installation. My tool is a simple Windows based executable that does not require installation and can live on your network home folder. This can be mapped across to any server you Remote Desktop into, so hey presto it is always available to you 🙂

One of the biggest usability points of the tool is the ability to zoom into out out of folders very easily and quickly. It is this ability which makes finding problem files and folders incredibly simple.

Imagine the following scenarios:

1. Your system drive is close to filling up, you are not sure why and your Windows Administrator is struggling to locate the problem folder.
1. You have thousands of databases across many instances on a server and your:
   - Data drive is full
   - Log drive is full
1. Your Backup drive is getting very full. Backup and archive routines appear to be working.

...and some real-world explanations for those situations that I have uncovered in seconds using this awesome utility:

1. A DBA left a long running Profiler trace locally on the SQL Server. It transpired that even though the trace file can be saved on any drive, Profiler unfortunately generates a ever growing temporary trace file locally on the C: Drive until it is stopped (a recipe for disaster!).
1. A mis-behaving bulk loading process caused a database data file to grow out of control.
1. A long running transaction prevented the log file VLFs from being truncated and caused it to grow out of control.
1. DBAs did not follow standards and practices and stored VLDB adhoc backups in random locations.

Remember fixing a SQL Server problem is usually pretty easy in most cases, the hard part is identifying *where* the actual problem lies. With the best Tool you have have never used, even identifying your space issues becomes a breeze.

# The reveal

Ladies and Gentlemen I give you [SpaceMonger](https://www.sixty-five.cc/download/)! Ensure that you download the old v1.4.0 version located on the *Free Software* tab since this is the superior release. Now imagine using this on a multi TB disk and being able to zoom into your problem. You will be astounded how fast and easy it is.

[![Spacemonger report](/images/2011/winsxsdirsucks.jpg)](/files/SpaceMonger.exe)<br/>
*The more data you have the easier those problem files are to find!*