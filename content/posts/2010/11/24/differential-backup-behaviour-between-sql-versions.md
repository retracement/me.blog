+++
date = '2010-11-24T13:47:00+01:00'
draft = false
title = 'Different differential backup behaviour between SQL versions'
+++

The other day I ran into a rather unique problem with a database when attempting to maintain it's transaction log. As it turned out my particular issue was related to old replication code -which I hope to post about at a later date, but essentially I needed to clear down some of the logfile's VLFs which I was struggling to perform. Anything I attempted to maintain it failed, however if I performed Transactional log backups, they would also fail due to the fact that there was no Full database backup in existence. Since the database was huge, and a full backup would have taken up to 12 hours to perform, it raised quite an interesting question namely *was there a way to force a Transactional log backup*?

My good friend Allen Dunn came up with the idea of taking a Differential backup first instead, but I had already tried this. However Alan insisted that it did work and upon questioning him further it transpired that he was doing so on a SQL 2005 server. 

![Twitter with Alan](/images/2010/twitterwithalan.png)

I was particularly interested since I was convinced that I had observed this exact same behaviour on that platform as well several years back, so I decided to run some tests. Below you will see the T-SQL code that I scrambled together to test, but essentially the test would first attempt a Differential backup on a new database (that obviously would not have a Full backup) and then followed the test up with a Transactional log backup on a new database. I was surprised to find that the behaviour does indeed differ between SQL 2005 and 2008.

```sql
SET NOCOUNT ON
 
SELECT @@VERSION
 
IF db_id('PuzzledDB') IS NOT NULL
 
    DROP DATABASE PuzzledDB
 
CREATE DATABASE PuzzledDB
 
--try performing differential backup with no full in place
 
BACKUP DATABASE PuzzledDB TO DISK = 'C:\PuzzledDB.diff' WITH DIFFERENTIAL
 
IF db_id('PuzzledDB') IS NOT NULL
 
    DROP DATABASE PuzzledDB
 
CREATE DATABASE PuzzledDB
 
--try performing transaction log backup with no full in place
 
BACKUP LOG PuzzledDB TO DISK = 'C:\PuzzledDB.log'
```

As you will see below the screen shot of the results from the SQL 2008 server did confirm exactly what I had experienced that not only the Differential backups require a full backup to have been taken, but also the Transactional log backups do too. I think this behaviour is understandable for normal operations on the basis that either backup without a corresponding *FULL* would essentially be useless for recovery purposes:

![2008 backup](/images/2010/2008backup.png)

```text
Microsoft SQL Server 2008 (SP1) - 10.0.2531.0 (Intel X86) Mar 29 2009 10:27:29 Copyright (c) 1988-2008 Microsoft Corporation Enterprise Edition on Windows NT 5.2  (Build 3790: Service Pack 2)
 
Msg 3035, Level 16, State 1, Line 9
 
Cannot perform a differential backup for database "PuzzledDB", because a current database backup does not exist. Perform a full database backup by reissuing BACKUP DATABASE, omitting the WITH DIFFERENTIAL option.
 
Msg 3013, Level 16, State 1, Line 9
 
BACKUP DATABASE is terminating abnormally.
 
Msg 4214, Level 16, State 1, Line 16
 
BACKUP LOG cannot be performed because there is no current database backup.
 
Msg 3013, Level 16, State 1, Line 16
 
BACKUP LOG is terminating abnormally.
```

Now here comes the surprise... As Alan quite rightly said, a differential backup works in SQL 2005! Interestingly the transactional log backups still fail with the same error:

![2005 backup](/images/2010/2005backup.png)

```text
Microsoft SQL Server 2005 - 9.00.4035.00 (Intel X86) Nov 24 2008 13:01:59 Copyright (c) 1988-2005 Microsoft Corporation Enterprise Edition on Windows NT 5.2 (Build 3790: Service Pack 2)
 
Processed 184 pages for database 'PuzzledDB', file 'PuzzledDB' on file 3.
 
Processed 1 pages for database 'PuzzledDB', file 'PuzzledDB_log' on file 3.
 
BACKUP DATABASE WITH DIFFERENTIAL successfully processed 185 pages in 0.342 seconds (4.410 MB/sec).
 
Msg 4214, Level 16, State 1, Line 16
 
BACKUP LOG cannot be performed because there is no current database backup.
 
Msg 3013, Level 16, State 1, Line 16
 
BACKUP LOG is terminating abnormally.
```

I'm not entirely sure why the ability to take Differentials on databases that haven't had a Full backup taken first (SQL 2008) has been removed or whether it being allowed in SQL 2005 was an oversight for that release. Personally I do not like this kind of "Nanny" style computing in restricting our actions because I believe that by forcing us down one resolution route hinders our ability to make an informed judgement to take another course of action. Going back to my initial problem that I touched upon at the beginning, I believe that had the transaction log backups been allowed to run they probably still wouldn't have resolved my problem with the VLF's, but what this would have done is shown me very early on that something else was *fundamentally wrong* in the database configuration and therefore arrived at a solution faster than I did.

**Disclaimer:** For important databases you should always try to guarantee that you are able to meet your RPO, this means always having a Full backup and any relevant Differential with respective Transaction log backups (depending upon your SLA and RPO). What this means to YOU as a DBA is to AVOID performing any operation that will break this chain!