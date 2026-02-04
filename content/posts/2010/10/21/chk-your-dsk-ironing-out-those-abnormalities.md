+++
date = '2010-10-21T13:59:00+01:00'
draft = false
title = 'More cluster disk corruption'
+++

Ironically not more than a week or so after I emphasised the importance for DBAs to check file system integrity in my recent post [Recovering from corruption in SQL Failover Cluster](/posts/2010/10/08/cluster-aint-misbehavin/), another incident raises it's ugly head. The post highlighted that file system corruption can cause unusual behaviour with your SQL instance, so when the other day the following errors in the event log were passed to me, I knew exactly what to look at.

In this first error Event ID 823: 
> The operating system returned error 21 (error not found) to SQL Server during a read at offset 0x000000000a0000 in file 'R:\Microsoft SQL Server\MSSQL.1\DATA\tempdb.mdf'"

It is fairly obvious that this error relates in some way to the file system. The clue is in the words *"operating system returned"*.

![Application log](/images/2010/corruption1_c.png)

The next error Event ID 9645:  
> An error occurred in the service broker manager, Error: 823, State: 2

This is less obviously related to the first, however there does indeed appear to be a pattern in the Application error log where by one error follows the other. The other thing that you will note is that the error code listed in the description matches the Event ID of the previous error.

![Event 9645](/images/2010/corruption2_c.png)

Anyway, irrespective of whether we believe there is a connection, let us troubleshoot the first problem. So the absolute first thing you need to ensure is that the file system has no errors. Let us run a read only `chkdsk` against it...

![chkdsk](/images/2010/corruption3_c.png)

Unsurprisingly we find that there are issues. Since this disk is actually a Clustered disk resource we need to ensure that we `OFFLINE` any resource that is dependant on it. When we run the `chkdsk /f` we will have to close all open file handles. Should any dependant resource be left `ONLINE` then we will find our Clustered Group failing over and we want to avoid that happening. We can leave the SQL Network Name and SQL IP Address both `ONLINE` but don't forget not to touch your disks. If you `OFFLINE` them you will not be able to fix them!

![offline services](/images/2010/corruption4_c.png)

So once we have taken everything `OFFLINE` we can then fix the file system.

![chkdsk /f](/images/2010/corruption6_c.png)

It is usually prudent to run another read only chkdsk post fix to ensure that this time it comes back clear, but once you are happy then all your `OFFLINE` resources can be turned back on.

Another final point to note is that when you get these file system corruption issues, they usually happen for a reason. They are generally suggestive that you may have a failing drive on you RAID array, and certainly in this particular instance this was the root cause of the problems. Our hardware management software gave us a delayed signal of impending drive failure a few days later, so this was definitely an early warning signal, and is perhaps is time to check you have replacement drives available to swap out. Most importantly at this stage you should *ENSURE* your Full and Transaction Log backups are succeeding in the event something goes horribly wrong and if you have a DR solution then start planning for failover.