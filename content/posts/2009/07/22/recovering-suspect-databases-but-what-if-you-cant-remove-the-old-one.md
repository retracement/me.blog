+++
date = '2009-07-19T12:18:15+01:00'
draft = false
title = 'How to perform an emergency database restore when blocked by corrupt database'
+++
The other day a fellow dba was having a problem with removing a corrupt database from a SQL Server and was failing to remember that the physical architecture of SQL Server is actually quite simple. Every time he tried to drop the database or to fix the database allowing for data loss or even take the database offline, the server would complain that the database was in single user mode (even though it wasn’t). Anything he tried on this database was complained about, and he was at a loss on how to remove it.

One of the reasons I really like SQL Server as a product is the simplicity of architectural design which makes recovery in what would first appear to be bad situations relatively easy. Oracle for all its benefits is far trickier to troubleshoot similar situations.

A SQL Server database is essentially a physical set of files and metadata describing the database in the master database. In this situation, I presumed that SQL was incorrectly reporting the corrupted database from the metadata and it thought the database was live (which it was but essentially unusable). Therefore my fix was to stop the SQL Service which would release the file handles, rename the data and log files and finally bring the SQL Service back online. This time SQL Server detected that the database was (obviously) offline and suspect and allowed for us to `DROP` it. Issuing the `DROP` statement in this situation essentially removes a piece of metadata in the master database.

Finally, my colleague was able to perform a restore from a good database backup.