+++
date = '2011-02-08T21:07:00+01:00'
draft = false
title = 'Where to go for WAIT stat information'
categories = ['Technology']
tags =['SQL','Performance']
+++

Fairly recently I was talking with a rather talented SQL Specialist internally at Microsoft and was asked the question by him as to where I get my Wait Stat information from. Query tuning isn't really something I currently am required to do on a regular basis so I tend to get my information as and when I need it (even though I have a couple of articles and urls bookmarked I couldn't particularly reference them in a conversation so perhaps I could go one better and officially post them). My answer then was that I Bing for my Wait Stat references (actually replace that with another search engine – but I was on the MS site!).

The *"SQL guy"* then mentioned a white paper by Tom Davidson, and as soon as this name was mentioned I thought he was talking about Louis (since he has recently co-authored this great book "Performance Tuning with SQL Server Dynamic Management Views" which I actively encourage you to get hold of). Nope he really did mean Tom 🙂 and I really should have come across him (perhaps I have) but the name didn't register at the time.

![Performance Tuning with SQL Server Dynamic Management Views](/images/2011/perf_tuning.jpg)

Anyway after a little bit of searching I managed to pull up a rather nice White Paper by him on the subject of Wait Stats (albeit SQL 2005 focused). So I thought I would first share this with you and then provide some of the other references that I have accumulated all in one handy location.

So the next time you are "waiting" perhaps you might want to find out why and check out these links:

[SQL Server Best Practices Article](https://msdn.microsoft.com/en-us/library/cc966413.aspx) by Tom Davidson with [whitepaper download](https://download.microsoft.com/download/4/7/a/47a548b9-249e-484c-abd7-29f31282b04d/Performance_Tuning_Waits_Queues.doc) for Performance Tuning Waits Queues. -the original article that spawned a post 🙂.

[sys.dm_os_wait_stats (Transact-SQL)](https://msdn.microsoft.com/en-us/library/ms179984.aspx) – this official MSDN page actually looks rather useful and might give you all the information you need.

[CSS SQL Engineers: The SQL Server Wait Type Repository](https://blogs.msdn.com/b/psssql/archive/2009/11/03/the-sql-server-wait-type-repository.aspx) – unfortunately this page comes across a bit like work in progress, but hopefully provides you with extra explanation.

[Description of the waittype and lastwaittype columns in the master.dbo.sysprocesses table in SQL Server 2000 and SQL Server 2005](https://support.microsoft.com/kb/822101) – yup ok a slightly older article but nether-the-less has some useful bits on it!

[Performance Monitoring Tools: SQL Server 2008 Activity Monitor](https://www.informit.com/guides/content.aspx?g=sqlserver&seqNum=329) by Buck Woody – yeah I know this one aint really about DMVs and Wait Stats but quite frankly it might be a good place to start!

[Top SQL Server 2005 Performance Issues for OLTP Applications](https://sqlcat.com/top10lists/archive/2007/11/21/top-sql-server-2005-performance-issues-for-oltp-applications.aspx) – a very simple but useful 6 point article to help you troubleshoot your system (courtesy of a great DBA Sean Kotze ...who currently isn't on or using Twitter, Blog, LinkedIn etc!)

Now I'm sure I had a few more urls on this subject, but as always when you go looking for something you can never find it. Well if I do come across any others then I will update this post with them, but for now I hope this small selection of links helps you troubleshoot your Wait Stats.