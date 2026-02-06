+++
date = '2011-02-08T12:50:00+01:00'
draft = false
title = 'Automate now to save time later'
categories = ['Personal Development']
tags =['TSQL Tuesday']
+++

![TSQL Tuesday](/images/shared/tsql2sday150x15032.webp)

It's T-SQL Tuesday time again and this month I'm a little late off the starting block. This time it is being hosted by Pat Wright [blog](https://sqlasylum.wordpress.com)|[twitter](http://twitter.com/SqlAsylum) and the subject of the month is all about "Automation in SQL Server".

So what is my favourite automation script or technique that I use in SQL Server? I had to have a think about this and my problem is that I have quite a few. I believe that the accessibility and usability of the Windows and SQL GUI (and related tools) means that SQL DBAs in particular have a tendency to automate much more than other IT Professionals.

So when should we automate? There are lots of answers to this question and here are a few possibilities:

- To enforce standards – for instance naming through run-time script checks.
- For repetitive tasks – how often are you prepared to repeat a task before you bang your head against a wall?
- For time consuming operations -if you can automate a task you can schedule it to run out of hours.
- To simplify/ customise operations in an easy to use way -finding blocked spids is easy via querying DMVs, but doing so via executing a stored procedure or custom view could be even easier.

In my current environment one very common task is to refresh databases from adhoc backups of like databases from other environments, and scripting the restore statement every time can be a big pain. In particular I want to ensure that the file names of the database to be refreshed are overwritten (remain the same) so I have written a quick script that simplifys this exercise somewhat. I don't pretend it is the finished article and need to make improvements, but it does enough right now. Please note that I am using the compatibility view `sysfiles` for the reason of backwards compatibility at this stage (so the code can run on SQL 2000). This is how to use it:

1. On the server that you are going to overwrite the database in question, load up the script in SSMS and change context into that database.
1. For the @backupfilefull variable, assign the location to the full backup file of the database you wish to restore.
1. Execute the script and it will output a T-SQL script that you can copy and execute.

Please be warned that *this will OVERWRITE the database* specified in the script but will at least first will backup the transaction log. Also note that I do not attempt to kill off SPIDs from the database so at least if you do make a blunder, if your database is active it gives you a chance of it failing.

```sql
-- =============================================
-- Author:            Mark Broadbent
-- Create date: 01/01/2009
-- Description: generates script to restore the current context database
-- Version: 0.1
-- =============================================
 
--this script generates a script to restore the current context database
--set source database to generate restore script from
DECLARE @backupfilefull SYSNAME
DECLARE @databasename SYSNAME
SET @databasename = db_name()
--set path to source database backup file
SET @backupfilefull = ''
 
DECLARE @@cmd VARCHAR(4000)
SET @@cmd = '--***WARNING*** this script will completely overwrite the database name'
   + ' that has been specified please CHECK this connection is on the correct deployment'
   + ' server!!!' + CHAR(10)
SET @@cmd = @@cmd + 'BACKUP LOG ' + @databasename + ' TO DISK = ''' + @backupfilefull 
   + '_tail_of_log_prior_to_norecovery_mode.log'' WITH STATS = 1, NORECOVERY' + CHAR(10)
--generate restore statement
SET @@cmd = @@cmd + 'RESTORE DATABASE ' + @databasename + ' FROM DISK = ''' 
   + @backupfilefull + ''' WITH STATS = 1, NORECOVERY'
SELECT @@cmd = @@cmd + CHAR(10) +',MOVE ''' + RTRIM(name) + ''' TO ''' 
   + RTRIM(filename) + '''' FROM sysfiles
 
--output generated restore script
PRINT @@cmd
 
--ensure connection leaves context
USE MASTER
GO
```

Well I hope you find this script useful, and please remember to be careful with it – I take no responsibility for it's use. 🙂

So to wrap up this post, I'll pose the question "How do we know when is a good time to automate?" and from my experience I would suggest that the answer is usually always. Even on those occasions where your automation code is never used again for the purpose you wrote it for, nine times out of ten I bet you will refer back to it and use snippets of it for something else. Always try to... *spend time now to save time later!*