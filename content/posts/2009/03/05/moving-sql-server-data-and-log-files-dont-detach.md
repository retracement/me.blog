+++
date = '2009-03-05T02:13:46+01:00'
draft = false
title = 'How to move your SQL Data and Log files correctly!'
+++

Recently I came across the `ALTER FILE` part of the `ALTER DATABASE` command and started thinking about its use when moving physical datafiles.

I must admit that this particular part of the `ALTER DATABASE` command has largely gone unused. Whilst the `REMOVE FILE` part is infrequently used and the `ADD FILE` part is possibly more so, the intuitive nature of the SQL 2005 and above GUI (and even SQL 7 and 2000 to some extent) has meant that resorting to various statements is largely unnecessary. In fact, the only times that I can really recall having resorted to the `MODIFY` operator is when I have been renaming logical file names.

For instance normally when I have a situation in which a database has been created with its log and data file/s located at the wrong locations, the simple solution would be to detach, move files around, reattach primary file specifying the locations of all the other files. Alternatively, if you like creating work for yourself then a backup and restore back over the top of the database, changing the restore locations of the database files would also work. These two operations, of course, can be performed quite easily through the GUI or command line alike and therefore have not to date posed any real issues to me.

However, this is really a “closing the door after the horse has bolted” method. I have found that using the `ALTER FILE` command it is far easier to change the physical locations where SQL Server will look for the database file/s, take the database `OFFLINE`, move the files to the relevant locations and finally bring the database back `ONLINE`. Now the reason that I believe that this is an improvement over the detach move and reattach method is that you are not removing (and adding back in) the database from the system catalogs. This method should, therefore, be quicker to perform, and I believe less prone to manual error.

Update (27/05/2009) : Further to this article titled [Moving Database Files Detach/Attach or ALTER DATABASE?](https://www.sqlservercentral.com/articles/Administration/65896/) I have just come across the following article which goes into a little more detail and reasoning.

