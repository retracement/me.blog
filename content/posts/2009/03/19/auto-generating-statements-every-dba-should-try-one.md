+++
date = '2009-03-19T02:13:49+01:00'
draft = false
title = 'Auto generate RESTORE scripts from a live database'
+++
Have you ever performed a scripted restore or some of similar operation and then got entirely frustrated at the laborious and error prone nature of this exercise especially when the database consists of many datafiles and transaction log files?

Of course the SQL Server 2005 GUI is very good at making the restore process much easier but it's biggest failing is that it doesn't preserve filenames (especially when the restored database name is changed). This is a very big problem when you have a large number of data files and causes a big headache.

My solution is to use TSQL to auto generate a restore script, and I have found this to work very well and is much faster and more flexible than the GUI approach. When performing operations such as setting up a database mirror (since this can be a more repetitive process AND it is fairly important to preserve file paths) this code really becomes quite useful.

```sql
--Ensure that connection is set to source database to
--generate restore script from
DECLARE @backupfilefull SYSNAME

--set path to source database backup file
SET @backupfilefull = 'C:\backuppath\backupfile.bak'

DECLARE @@cmd VARCHAR(4000)

--generate restore statement
SET @@cmd = 'RESTORE DATABASE ' + db_name() + ' FROM DISK = '''
+ @backupfilefull + ''' WITH STATS = 1, NORECOVERY'
SELECT @@cmd = @@cmd + CHAR(10) +',MOVE ''' + RTRIM(name) + ''' TO '''
+ RTRIM(filename) + '''' FROM sysfiles

--output statement
PRINT @@cmd
```

Running this under the context of the master database would generate the following script:

```sql
RESTORE DATABASE master FROM DISK = 'C:\backuppath\backupfile.bak'  WITH STATS = 1, NORECOVERY
,MOVE 'master' TO 'D:\sql2005\MSSQL.1\MSSQL\DATA\master.mdf'
,MOVE 'mastlog' TO 'D:\sql2005\MSSQL.1\MSSQL\DATA\mastlog.ldf'
```

I have created similar auto generation scripts for creating database snapshots and various other things which makes life SO much easier. Hope you get the idea!