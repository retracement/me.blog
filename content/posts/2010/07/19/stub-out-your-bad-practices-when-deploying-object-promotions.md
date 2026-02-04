+++
date = '2010-07-19T12:18:00+01:00'
draft = false
title = 'Using STUBs for object promotions'
+++

When a script is deployed in your production environment, how is it deployed? Are the scripts written to account for the object already existing and if so what actions are taken? But what if the object doesn't exist, what then? All too often in these scenarios the usual style of deployment scripts I see being provided to database teams first detect the existence of said object and then drop it (should it exist). This then leaves the way clear for the script to perform its create statement. is there really anything wrong with this approach? Well yes, actually there is.

Every time an object is dropped from a database you are also removing database user permissions to it. This sounds (and ought to be) blatantly obvious, but from my experience it doesn't really seem to be the case or perhaps is thought to be not too important. In the cases where a developer is aware of what is happening, they tend to add extra DCL to their promotional script so that security is added back in. Obviously from a DBA perspective, having the security of your database and objects open to manipulation (albeit through a legitimate route) is not a good thing and therefore active usage of DCL should be discouraged or forbidden for standard promotions.

![Developers vs DBAs](/images/2010/johnsteelesmall.jpg)

For those developers who are not aware that dropping an object loses the permissions, I believe they must think that the permissions somehow remain intact for the database user itself so that when an object is recreated the permissions would become relevant again. *THIS IS NOT THE CASE!* Remember, when dropping and recreating an object, it results in a new objectid and so even if the permission set did remain on the database user they would not point to this new object anyway.

A different approach to your scripting is needed. Logically you must ensure that if an object exists then it is `ALTER`ed and if and only if the object does not exist should you issue a `CREATE`. At no time whatsoever should a `DROP` ever be issued unless of course a complete and persistent removal of an object is the desired consequence.

There are a couple of hurdles to overcome in order to achieve this. The first is that the `ALTER` and `CREATE` statements must be the first statements of a batch and secondly logical `IF...ELSE` constructs and `GOTO` operators cannot span batches, therefore how can it be possible to check to see if the required object already exists and to take the relevant action depending upon that result?

![Tearing your hair out](/images/2010/tearhair.jpg)

The way around this issue is to have the object modification script to always flow sequentially and once any logical branching has completed to continue along the same path. Also the object change script should always perform an `ALTER` statement, and this can occur post `GO` batch separator. This gets around most of the problems but we haven't yet accounted for the possibility that the object may not exist. Since we are always performing an `ALTER` for the change script, all we simply have to worry about is ensuring that we create the object if it doesn't already exist. In this instance we still face the issue regarding the `CREATE` statement requiring its own batch, but the problem here is that in order to know whether a `CREATE` is necessary, a logical test needs to be performed, and the solution to this is not so obvious since the logical test means that the `CREATE` would not be the first line in a batch!

The solution to this last problem is actually quite simple. We can execute the `CREATE` batch by utilizing the `EXEC` statement, and this is great since we can now nest this within a logical existence test. This solution meets all of our requirements. Permissions and objectids are retained, and the script is very simple to maintain. Only the `ALTER` code segment would need to be updated for future code revisions.

```sql
USE dbaadmin
GO
IF OBJECT_ID (N'usp_killspids') IS NULL
BEGIN
    EXEC ('
    CREATE PROCEDURE usp_killspids AS
    SELECT ''This is a stub procedure, implementation of it will
     be created by an ALTER statement''
    ')
    PRINT 'Procedure usp_killspids does not exists, creating a
     stub procedure before executing create script...'
END
GO

ALTER PROC [dbo].[usp_killspids]
.....
AS
.....

GO
```