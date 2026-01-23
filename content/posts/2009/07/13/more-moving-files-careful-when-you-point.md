+++
date = '2009-06-09T20:53:15+01:00'
draft = false
title = 'Issues when moving your SQL data and log files'
+++
You might recall that I recently touched upon how to go about moving your [SQL data and log files correctly](/posts/2009/03/05/moving-sql-server-data-and-log-files-dont-detach). In this post I will quickly walk through one common scenario.

Ok so your in a hurry and you've stopped your SQL Service. All relevant datafiles have been moved to a new location and you are ready to roll. SQL Service has been started back up and unfortunately (but you expected this) none of the user databases whose files you moved have come back up. Well thats alright you say, because you are going to repoint them.

Its only at this stage that you realise that you forgot to make a note of all the source locations and you start cursing and hoping that you dont have to move back that 300GB worth of files in order to bring the databases back up to run `EXEC sp_helpfile` or query the databases' sysfiles table.

After a biscuit and a cup of tea you remember that this is only after all meta data and should be stored in the master database. Upon further examination you find the sysaltfiles table and are able to reference the filegroup name and original location so that you may now issue the `ALTER DATABASE` command.

Thankfully this all works a treat and you may now stop panicking.

A couple of points of interest here. Firstly I wondered whether after repointing the database in this situation whether a simple `ALTER DATABASE SET ONLINE` would be enough or whether you would have to at first `OFFLINE`. Well as you would hope would happen, all SQL requires is that you issue the `ONLINE` directive. The other interesting thing that I noticed is that should you mistype the repointing statement, SSMS returns a rather misleading error message which suggests that something not very pleasant may have happened to your datafile, when in fact you have simply entered a wrong statement.

_**Update (16th August 2010):** Using SQL 2005 and upwards you should use sys.master_files instead of the sysaltfiles compatibility view, since the latter is obviously depreciated._