+++
date = '2016-01-13T13:30:00+01:00'
draft = false
title = 'Problem removing files from TempDB'
categories = ['Technology']
tags = ['SQL','TempDB']
+++

I recently ran into an interesting problem while attempting to remove approximately half of the TempDB data files configured in a testing environment. As you might expect, there are various SQL Server tasks that are performed infrequently by a DBA, and this is a good example of one of them. Most environments have usually been misconfigured with too few TempDB data files or wrongly sized usually resulting in the classic allocation page contention problems (which is explained by this excellent SQLSkills article [The Accidental DBA (Day 27 of 30): Troubleshooting: Tempdb Contention](https://www.sqlskills.com/blogs/paul/the-accidental-dba-day-27-of-30-troubleshooting-tempdb-contention/)), but this particular environment had been temporarily over provisioned with disks and TempDB data files (for testing purposes).

---

*Bonus fact: SQL Server 2016 attempts to automatically address the tempdb datafile allocation page contention problem by defaulting to 8 TempDB data files (or less if the number of cores is smaller). This can be overridden on install through the GUI or by using the `/SQLTEMPDBFILECOUNT` switch if performing a command line installation. Further optimisations have been implemented such as the adoption of uniform extent allocations by default and auto-growing all files simultaneously -both behaviours would formally have required turning on Traceflags [1117](https://blogs.technet.com/technet_blog_images/b/sql_server_sizing_ha_and_performance_hints/archive/2012/02/09/sql-server-2008-trace-flag-t-1117.aspx) and [1118](https://blogs.msdn.microsoft.com/psssql/2008/12/17/sql-server-2005-and-2008-trace-flag-1118-t1118-usage/). Several other improvements have been made to the performance of TempDB database which you can read about yourself in this (currently preview) [documentation](https://msdn.microsoft.com/en-us/library/ms190768.aspx?f=255&MSPPError=-2147217396).*

---

So the plan was to remove each TempDB file one by one, restart the instance (if required) and decommission those spare disks so that they can be returned and reallocated elsewhere in the environment. In order to remove each data file we would:

1. Empty each file at a time (using `DBCC SHRINKFILE`).
1. Remove each file upon empty.

Now there was a time (prior to SQL Server 2005) when we were told to tread very very carefully when shrinking TempDB files, and doing so could result in corruption. This perception remains with some of the longer serving SQL Server professionals (I question myself frequently) but if we take a look at [KB307487](https://support.microsoft.com/en-gb/kb/307487) article [How to shrink the tempdb database in SQL Server](https://support.microsoft.com/en-gb/kb/307487) we can see that it is now safe to do so -although (as the Knowledge Base article states) certain scenarios can cause the shrink operation to fail.

So I ran the following code for each TempDB data file:

```sql
DBCC SHRINKFILE (tempdb_15, EMPTYFILE)
go
ALTER DATABASE [tempdb] REMOVE FILE [tempdb_15]
GO
```

The code succeeded for the first few files, right up to TempDB data file 12, where I hit the following error:

>DBCC SHRINKFILE: Page 14:56 could not be moved
because it is a work table page.
Msg 2555, Level 16, State 1, Line 1
Cannot move all contents of file "tempdb_12" to
other places to complete the emptyfile operation.
DBCC execution completed. If DBCC printed error
messages, contact your system administrator.
Msg 5042, Level 16, State 1, Line 1
The file 'tempdb_12' cannot be removed
because it is not empty.

As odd as this error was (especially since the SQL instance was not currently *in use*) I decided to bounce it but after restarting the instance was again greeted with the same error message! After some very quick Google-Fu I came across an old MSDN Forum question [DBCC SHRINKFILE: Page 4:11283400 could not be moved because it is a work table page](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/2a00c314-f35e-4900-babb-f42dcde1944b/dbcc-shrinkfile-page-411283400-could-not-be-moved-because-it-is-a-work-table-page?forum=sqldatabaseengine). I ruled out some of the responses but came across the response by Mike Rose:

>I realize this is an old thread but I have found that in most cases work tables are related to Query Plans.
>Try issuing the following commands and the shrinking the tempdb:
>
>DBCC FREESYSTEMCACHE (‘ALL’)
>
>DBCC FREEPROCCACHE
>
>there will be some performance hit for this as SQL will have to recreate its Query Plans, but it should allow you to shrink you TEMPDB.
>
>Hope this helps,
>
>M Rose
 
Now my initial thoughts were that this response was clearly nonsense since I had just restarted the SQL Service for the instance (thereby flushing the SQL caches), but hey what the hell, why not give it a whirl...

**...and success! It worked.**

Apart from the flushing of the caches, the outcome was even more strange to me for another reason. Upon reboot TempDB is *recreated* upon startup<sub>*1</sub> and I would have expected that fact alone to have fixed the problem but from the behaviour I had experienced, something had persisted across instance restart.

Also of interest to me was whether running `FREEPROCCACHE` and `FREESYSTEMCACHE` was overkill so when the opportunity arose I attempted first to try only clearing the systemcache specific to TempDB through:

```sql
DBCC FREESYSTEMCACHE ('tempdb')
```

...and then tried clearing temporary tables and table variables through:

```sql
DBCC FREESYSTEMCACHE ('Temporary Tables & Table Variables')
```

...and then by using both together.

```sql
DBCC FREESYSTEMCACHE ('tempdb')
DBCC FREESYSTEMCACHE ('Temporary Tables & Table Variables')
```

Sadly these did not seem to work.

On my final opportunity I tried clearing the procedure cache (`DBCC FREEPROCCACHE`) and after running this a few times it appeared to solve the problem and I was able to remove the TempDB file without error.

---

Clearing system caches on a Production SQL Server instance is never a good thing and should be avoided, but at least the next time I run into this problem I have (what appears) to be a work around to the problem I ran into and at least the requirement to remove TempDB data files should be a rare occurrence for most of us!

I confess that I am left with a few unanswered questions and clearly what I experienced does make me doubt the results somewhat (and my sanity) and there were further tests that I would like to perform at some point. So if you run into this problem yourself I encourage you to try this (_**disclaimer:** your fault if you destroy your SQL Server_ 😉) and I would be delighted if you leave feedback of what works for you (and what doesn’t)!

---

*<sub>*1</sub> you may want to read this interesting post by Jonathan Kehayias [Does tempdb Get Recreated From model at Startup?](https://sqlblog.com/blogs/jonathan_kehayias/archive/2010/05/14/does-tempdb-get-recreated-from-model-at-startup.aspx)*