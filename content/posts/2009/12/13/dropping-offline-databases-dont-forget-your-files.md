+++
date = '2009-12-13T23:04:00+01:00'
draft = false
title = 'Remember your database files when dropping OFFLINE databases'
+++

If I asked you the question, "what happens to the physical data files if you drop an offline database?" would would probably have to have a bit of a think about it.
There are really only three potential outcomes here:

1. The DROP fails
1. The DROP succeeds and removes the data files
1. The DROP succeeds and leaves the data files

Now if you think about the nature of the offline database the answer becomes a little more obvious. The `OFFLINE` database is simply meta data in the system catalog, there are no associations to files apart from that which exists within this meta. Therefore on this basis, if you remove an offline database, you are essentially removing the meta data. So on this basis you can deduce that the physical data files would remain – and this is exactly what happens (option 3).

Therefore it is very important to remember to tidy up your old data and log files on your operating system that will be orphaned over time due to decommissioning of old databases. For instance one technique I always use when a request has been submitted to remove a database (after the usual backup policies) is to firstly set the database to readonly for one week. If no problem records have not been raised during this time I will then set it to offline for another week, and then finally drop the database from the server.

Now it is at this point that it is important to remember to remove those orphaned files from the server otherwise there will be valuable wasted OS space and believe me, if you were not aware of SQL Server's behavior when dropping databases, and you are dealing with very many databases, it is very easy to forget.