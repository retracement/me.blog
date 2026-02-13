+++
date = '2015-12-18T17:12:00+01:00'
draft = false
title = 'Restoring CDC enabled databases'
categories = ['Technology']
tags = ['CDC','SQL']
+++

![Surprised](/images/2015/surprised.jpg)

Performing database and transaction log backups (and restores) are the meat and veg of a DBA's responsibilities and if you are a seasoned professional I am sure you will have performed those operations ad-infinitum. Being complacent is perhaps the biggest mistake you can make as a *"Guardian of Data"* so you should always be prepared for the unexpected...

# What happened

Several years ago in a Disaster Recovery scenario, I was asked to perform the migration of a large database which was used for a fairly complex ETL process that was both undocumented and something that the DBA team had no insight or involvement.

Due to a complete lack of a Disaster Recovery Plan for the system and process (how many times have we seen this guys!) I was forced to follow best practices, common sense and draw on my experiences in order to bring the service back to a *"best known good"* as quickly as possible. Since we had access to all existing backups taken so far, and having the ability to take a tail log backup I was able to recover across to DR with no data-loss (at least as far as I was concerned). I handed the environment back to the Business Intelligence team for their validation, operational testing and re-acceptance.

Of course it passed acceptance and resumed operation in production.

Now of course, what was not known at the time I performed this work was that the Database in question was enabled for Change Data Capture (CDC). Perhaps it should have been a logical assumption for me to make, but I suspect that even if I was aware of its presence, the significance of the situation might have escaped me.

I am not really sure what acceptance testing was performed (if indeed it was!), because after a few hours of activity (just enough time for a scheduled ETL process to execute and fail) it became clear that something was very wrong with the database.

Apparently the Change Data Capture tables were missing!

# The problem
 
So now I knew about the existence of CDC in the database (and after digging around MSDN documentation for a few minutes) our mistake was obvious – we had not used the `KEEP_CDC` option upon restore, meaning that the CDC tables were no longer present. Unfortunately acceptance testing hadn't detected this problem ahead of time and now our recovered database was live. After some discussion, the Business Intelligence team decided for us to re-create CDC on this warm database and to temporarily restore a copy of the old one (with KEEP_CDC!) to pull out those missed changes.

So what's the problem with restoring CDC enabled databases?

*The main problem with CDC enabled databases is that the `KEEP_CDC` option is incompatible with the `NORECOVERY` option.*

For instance the following command:

```sql
RESTORE DATABASE [CDCEnabledDB] 
    FROM DISK = 'CDCEnabledDB.bak'
    WITH NORECOVERY, KEEP_CDC
```

Will result in:

> Msg 3031, Level 16, State 1, Line 1
Option 'norecovery' conflicts with option(s) 'keep_cdc'.
Remove the conflicting option and reissue the statement.

This automatically poses the question of *how it is possible to restore a backup chain with CDC?* On a database restore, in order to apply differential backups and transaction logs the `NORECOVERY` clause is required to prevent SQL Server from performing database recovery.

If this option is required but using KEEP_CDC in conjunction with it is incompatible, surely this means point in time restores are not possible for restores that require CDC tables to be retained?

**Wrong!**

---

# The investigation

When I first ran across this problem (and after reading [Connect Item 587277](https://connect.microsoft.com/SQLServer/feedback/details/587277/keep-cdc-conflicts-with-norecovery-and-standby-options) -which has now been updated by me with the correct resolution) I was incensed that Microsoft could be so stupid as to prevent something so necessary under certain situations. I saw it as a huge gaping hole in database recovery for any database requiring CDC. As I (and others on the Connect Item) saw it, if you cannot restore a log chain and KEEP_CDC then this will cause problems for:

- Database Mirroring
- Availability Groups
- Log Shipping
- Replication
- Recovery to Standby

Quite infuriatingly the Connect Item was closed with:

> "we looked into this but there is no plan as of now to change the current design and extend the support of this in the foreseeable future. Again, thanks for taking the time to share your feedback, this is really important to us."

And that was that... for a time.

---

# So what has changed?

Several months ago I was investigating the use of CDC enabled tables for a client project when I stumbled across the following statement in the MSDN documentation Replication, Change Tracking, Change Data Capture, and AlwaysOn Availability Groups :

>"SQL Server Replication, change data capture (CDC), and change tracking (CT) are supported on AlwaysOn Availability Groups."

Remembering the problems I faced years earlier I read that statement with interest and decided to spend time playing with the recovery of Change Data Capture when it dawned on me that mine and the other posters to the Connect Item (including the Microsoft respondent) had completely misunderstood the use of the KEEP_CDC clause.

Our misunderstanding was (unsurprisingly) that the KEEP_CDC clause was necessary to protect Change Data Capture tables from being lost on every restore sequence (which is wrong). Instead the clause is required to protect CDC tables upon RECOVERY. Change Data Capture schema and data is transparently restored behind the scenes during NORECOVERY in anticipation that you use the KEEP_CDC clause on RECOVERY -if not then CDC is dropped (again transparently).

*Therefore in a restore sequence, recovery is the ONLY time that you need to specify KEEP_CDC in your restore sequence meaning that the incompatibility between this option and NORECOVERY is irrelevant.*

For instance the following will work:

```sql
RESTORE DATABASE [CDCEnabledDB] 
    FROM DISK = 'CDCEnabledDB.bak'
    WITH NORECOVERY
 
RESTORE LOG [CDCEnabledDB] 
    FROM DISK = 'CDCEnabledDB_1.trn'
    WITH NORECOVERY
 
RESTORE LOG [CDCEnabledDB] 
    FROM DISK = 'CDCEnabledDB_2.trn'
    WITH RECOVERY, KEEP_CDC
```

---

# In summary

Regardless of however experienced you believe yourself to be with SQL Server, there is always something that can come and bite you. Not even Microsoft Support and technology owners always fully understand how every single scenario or configuration is going to play out, so it is important that you *Do Your Own Research*.

While the use of CDC does introduce certain considerations that I will talk about another time, you should rest assured that as long as you understand the use of the KEEP_CDC clause (and make sure you use it when necessary) then you will not go far wrong.