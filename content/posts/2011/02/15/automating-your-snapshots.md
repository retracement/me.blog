+++
date = '2011-02-15T16:49:00+01:00'
draft = false
title = 'Automating your SQL snapshots'
categories = ['Technology']
tags =['SQL']
+++

A little while ago I was talking to someone about possibilities of reporting from SQL Server to avoid impacting the production server and the very first thing that sprang to mind was for them to use snapshotting from a mirror. They had already considered this option but it was not really viable because an almost real-time reporting solution was required. It wasn't until later that I wondered whether I had not really been precise enough in my explanation.

Since creating a snapshot can be pretty much instantaneous due to the way in which their mechanics work, as long as *real-time* means the time at any one instant, then there is no reason why snapshotting cannot be the solution for you. The only problem then that remains is how you are going to manage your snapshots, namely creation and destruction. Since the initial overhead of setting up (and then removing) a snapshot is relatively low, it makes perfect sense in my opinion to generate snapshots for reporting and then tear them down post report creation.

When you are creating a snapshot, since you must specify all datafile locations, auto-generation of them may at first seem a little more complicated than it really is. In truth, it is not particularly difficult and I have written some fairly useful code which can create a snapshot for you easily. Obviously you can customise the naming conventions of the snapshotted database to your requirements. Your reporting solution then could call a procedure to create the report snapshot (using the code below) on your mirrored database. All that would remain is to tear the snapshot down once the report has completed. I shall leave it to your imagination how you want to tear down your snapshots since depending upon your naming convention your method might differ slightly, but querying snapshots in existence is very simple as follows:

```sql
DECLARE @database_id sysname
SET @database_id = DB_ID('Northwind')
SELECT d.name,f.physical_name FROM sys.databases d JOIN
sys.master_files f ON d.database_id=f.database_id
WHERE d.source_database_id = @database_id
```

The procedure does however have an output parameter that returns the created snapshot name and you can use it in your reporting TSQL code to tear the snapshot down post report generation. Its up to you, but it is there if you want it!

We then move on to the main code itself to auto-generate your database mirror snapshot. The first thing you need to do is to decide which database you are going to deploy this stored procedure to. I personally prefer to have a dedicated database administration database that has all the custom written procedures, functions and the like housed in there. Once you have changed context into the database of choice your then need to execute the usp_createsnapshot creation script which can be found [here](https://bit.ly/RXYajR) to deploy.

Once this procedure is deployed, all you need to do from now on in order to create a quick snapshot on any of your databases or mirrors located on this server is simply call the proc and pass in the name of the database or mirrored database and execute and hey presto a database snapshot has been created for you 🙂
Below I have entered a few examples of how the procedure works and it's error handling and feedback. The final example is valid input.

```sql
EXEC dbo.usp_createsnapshot @help=1
GO
EXEC dbo.usp_createsnapshot master
GO
EXEC dbo.usp_createsnapshot invaliddbname
GO
DECLARE @ss sysname
EXEC dbo.usp_createsnapshot msdb,@ss OUT
PRINT 'Snapshot ' + @ss + ' created.'
```

Within this following screenshot are the results, and in particular the output within the red box demonstrates the correct use of the stored procedure.

![Snapshot output](/images/2011/snapshot_output.png)

Well that's the end of this post ladies and gentlemen. I hope you find this stored procedure as useful as I do, and remember... *happy reporting!*