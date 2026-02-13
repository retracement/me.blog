+++
date = '2012-04-17T18:38:00+01:00'
draft = false
title = 'whoisactive on SQL 2000'
categories = ['Technology']
tags = ['SQL','SQLHelp']
+++

Yes yes yes I *know* we are currently on SQL 2012 but just hear me out for a second...

I think I am probably one of the only people on the planet who has not (yet) used Adam Machanic's ([blog](https://sqlblog.com/blogs/adam_machanic/default.aspx)|[twitter](https://sqlblog.com/blogs/adam_machanic/archive/2011/04/27/who-is-active-v11-00-a-month-of-activity-monitoring-part-27-of-30.aspx)) whoisactive Stored Procedure. One of the reasons for this is that going right back to the days of SQL Server 2000 I had rolled my own stored procedure which worked very nicely and did everything that I ever needed it to.

With the advent of SQL Server 2005 came the ever so useful DMOs and whilst these gave an even more insightful portal into the guts of SQL 2005 (and now all versions above) internals, my own Stored Procedure still gave me all the information I needed. If there was any extra information I needed to obtain, I would usually delve into the DMVs using code snippets taken primarily from Louis Davidson or Glenn Berry.

I'll be honest. I never got around to rewriting the code I wrote -I got lazy people! I always intended on doing so, but never quite found the time or excuse.

Well today whilst monitoring Twitter's *#SQLHELP* hashtag I came across a request for a SQL 2000 version of whoisactive. I offered up my dusty ol' proc and it was gratefully accepted...

---

Please be aware the the following code has been provided as is, taken straight from my very own (old) scripted *"dbaadmin"* solution database targeted specifically towards SQL 2000 but has been tested and works up to (at least) SQL 2008R2 (I have not tested on SQL 2012). Because of this, my `usp_activity` Stored Procedure requires two prerequisites to work:

1. A dbaadmin database
1. An activity table created in the dbaadmin database

I appreciate that in your environment, you may have your own administrative databases -but luckily I have extensively commented my code so it really shouldn't be very difficult for you to do the switch. With respect to the activity table, that is simply needed for the capture parameter if you wish to persist activity statistics. If that is not something that you need, then again you can remove the code segment for that and therefore remove the dependency.

Before I give you the code let me tell you roughly how it works. In SQL 2000, in order to obtain the last executed statement for a SPID we would use [DBCC INPUTBUFFER(spidid)](https://msdn.microsoft.com/en-us/library/ms187730.aspx). The other part of the equation is the querying of the `master.dbo.sysprocesses` table to view all active SPIDs on our instance. Unfortunately, since the `DBCC` result set is being returned per SPID and not as a tabular result set for all, we are unable to join our last executed statement result to the sysprocesses result set. In addition, the second challenge is that in order to store the output of each `DBCC INPUTBUFFER` request (using the `INSERT EXEC` technique)  we must insert each item separately into a table as is. The output does not have a SPID id and therefore this means we are missing the join key in this table. To overcome this second challenge I simply perform the insert (into a table that has a column for a SPID id) and then update the SPID number as a separate operation. We perform our `DBCC INPUTBUFFER` for each SPID in succession using a cursor. Now we have our last executed statement table with respective SPID ids. More importantly we have also solved our first problem and are able to join our `sysprocesses` data (first captured into a temporary table) against our statement data ...who needs DMOs!

The stored procedure uses a selection of named parameters with defaults and the most important of these is the `@help` parameter.

If you execute the stored procedure as follows:

```sql
exec dbo.usp_activity @help=1
```

Then you will receive the very useful help output:

```text
Help Specified.
 
    @dbname sysname = '', --optional option (if unspecified all activity is captured, otherwise only activity for specified database is captured)
    @capture bit = 0, --optional option (if unspecified results are not captured, otherwise enables capture process activity to dbaadmin..activity table)
    @show bit = 1, --optional option (show process list (default))
    @excludespid int = 0, --optional option (exclude spidid from process list (default))
    @filteronspid int = 0, --optional option (display only spid, 0 for all spids (default))
    @blockedonly int = 0, --optional option (display only spids participating in blocking, 0 for all active spids (default))
    @simple int = 0, --option option (display cut down column list for result set)
    @help bit = 0 --optional option (view help)
```

If you wish to return all current SPIDs then leave off all parameters OR if you wish to return all SPIDs currently running in a database context then specify the `@dbname` parameter. One parameter that comes in very useful is `@blockedonly` and will return only blockers or blocked SPIDs by specifying a value of 1:

```sql
exec dbo.usp_activity @blockedonly=1
```

This returns a very useful amount of information about our blockers and blocked SPIDs and has saved my bacon on many occasions (result set below has been condensed):
![usp_activity result set](/images/2012/spid4.png)

I shan't go into any more detail about any of the other parameters or how you would use them since I think they are fairly obvious but if not they shouldn't take you very long to figure them out. So for now I shall list the code that you will need to setup `usp_activity`.

First we create the dbaadmin database.

```sql
CREATE DATABASE dbaadmin
GO
```

Next we need to create the activity table within the dbaadmin database.

```sql
USE [dbaadmin]
GO
set ANSI_NULLS ON
set QUOTED_IDENTIFIER ON
GO
USE dbaadmin
GO
IF 'activity' IN (SELECT NAME FROM sysobjects WHERE TYPE = 'U')
BEGIN
PRINT 'Table activity already exists, dropping before executing create script...'
DROP TABLE activity
END
GO
-- =============================================
-- Author:        Mark Broadbent
-- Create date: 13/02/2001
-- Description:    Creates activity table to store results for stored procedure usp_activity
-- =============================================
CREATE TABLE [dbo].[activity](
[database] [nvarchar](128) NOT NULL,
[spid] [smallint] NOT NULL,
[kpid] [smallint] NOT NULL,
[blocked] [smallint] NOT NULL,
[waittype] [binary](2) NOT NULL,
[waittime] [bigint] NOT NULL,
[lastwaittype] [nchar](32) NOT NULL,
[waitresource] [nchar](256) NOT NULL,
[dbid] [smallint] NOT NULL,
[uid] [smallint] NOT NULL,
[cpu] [int] NOT NULL,
[physical_io] [bigint] NOT NULL,
[memusage] [int] NOT NULL,
[login_time] [datetime] NOT NULL,
[last_batch] [datetime] NOT NULL,
[ecid] [smallint] NOT NULL,
[open_tran] [smallint] NOT NULL,
[status] [nchar](30) NOT NULL,
[sid] [binary](86) NOT NULL,
[hostname] [nchar](128) NOT NULL,
[program_name] [nchar](128) NOT NULL,
[hostprocess] [nchar](8) NOT NULL,
[cmd] [nchar](16) NOT NULL,
[nt_domain] [nchar](128) NOT NULL,
[nt_username] [nchar](128) NOT NULL,
[net_address] [nchar](12) NOT NULL,
[net_library] [nchar](12) NOT NULL,
[loginame] [nchar](128) NOT NULL,
[context_info] [binary](128) NOT NULL,
[sql_handle] [binary](20) NOT NULL,
[stmt_start] [int] NOT NULL,
[stmt_end] [int] NOT NULL,
[request_id] [int] NULL,    --new column in sysprocesses in 2005
[eventtype] [nvarchar](30) NULL,
[parameters] [int] NULL,
[eventinfo] [varchar](4000) NULL,
[snaptime] [smalldatetime] NULL
 
) ON [PRIMARY]
 
GO
SET ANSI_PADDING OFF
 
PRINT '======================================================================================================='
IF 'activity' IN (SELECT NAME FROM sysobjects WHERE TYPE = 'U')
BEGIN
PRINT 'Table activity successfully created...'
END
 
GO
```

And finally the Stored Procedure itself must be created.

You can download the entire script from [here](https://bit.ly/1fhTcZv).

So there you have it. I don't pretend for one minute that the T-SQL code is perfect, even during the process of putting this post together I spotted a couple of things that should be changed such as wrong choices for datatypes and a slightly roundabout way for checking the existence of objects, but the important thing is that the code works and works well. Therefore if you are still using SQL Server 2000 in your existing environment and need whoisactive style functionality then look know further. Enjoy.