+++
date = '2009-05-27T02:11:18+01:00'
draft = false
title = 'Recursive killing of SPIDs in a database for admin operations'
+++

One problem that I have found particulary annoying of late when performing any database operation that requires connections to it to be disconnected, such as setting it offline or a restoring over the top of itself is getting rid of these connections. Obviously this can be achieved by listing the current spids connected to the database and performing a kill for each spid. This can be quite laborious and obvious does not address any spids coming live whilst you are busy killing the existing ones, so you can end up going around in circles.

My solution to this is a script to effectively automate this manual process so that whilst the `SET OFFLINE` or `RESTORE` operation or such like are ticking away the procedure can be run specifying the name of the database and hey presto all live connections to this database are killed off one by one. The clever part (imo) to this script is the way in which it doesn’t attempt to block kill a batch of spids at the same time which might not be desirable should one of these spids close and be reused, but instead takes the first spid in the list and kills that and so on until no more spids are active for the specified database.

Obviously it is possible when altering a database to specify the option `WITH ROLLBACK IMMEDIATE` which has the same effect, but this option is not available for every situation and operation.

Anyway here is the stored procedure main body:

```sql
@dbname sysname, –required option (the database from which to kill of any connected user based spids)
@help bit = 0 –optional option (view help)
AS
SET NOCOUNT ON

IF @dbname NOT IN (SELECT name FROM master..sysdatabases) AND LEN(@dbname) = 0
BEGIN
PRINT @dbname + ' database specified cannot be found.'
PRINT 
RETURN
END

IF LOWER(@dbname) IN ('master','tempdb')
BEGIN
PRINT @dbname + 'invalid operation on database ' + @dbname + ' specify different database.'
PRINT 
RETURN
END

SET NOCOUNT ON
DECLARE @spid SMALLINT
DECLARE @str VARCHAR(255)

–just select the last spid and kill it and loop around until no more spids
PRINT 'Killing user processes in database ' + @dbname
PRINT 'Executing from SPID ' + CONVERT(VARCHAR,@@SPID)
WHILE EXISTS(SELECT TOP 1 spid FROM master..sysprocesses WHERE dbid = DB_ID(@dbname) and spid > 50 and spid @@SPID) –user spids are 51 and higher
BEGIN
SELECT TOP 1 @spid = spid
FROM master..sysprocesses
WHERE dbid = DB_ID(@dbname) and spid > 50 and spid @@SPID –user spids are 51 and higher

SET @str = 'KILL ' + CONVERT(VARCHAR, @spid)

PRINT @str;

EXEC(@str)
END

PRINT 'Complete.'