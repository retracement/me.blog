+++
date = '2010-09-16T15:58:00+01:00'
draft = false
title = 'An approach to dealing with database corruptoin'
+++

After attending Paul and Kimberly's excellent Masterclass in London a few months ago, I made a scribble in my notepad to play around a little more with self induced corrupted DBs (yes you read that right!). Part of Paul's demonstration showed various recovery techniques and gotchas when you are unfortunate enough to be hit with corruption problems. Having been exposed to quite a few situations in live environments, I am not exactly a stranger to DBCC CHECKDB and fixing corruption issues, but the session was definitely enlightening and taught me some useful things.

It also made me realize that the biggest problem your average production DBA experiences with corrupted databases. The DBA normally always first encounters corruption in a live scenario, which means that they have to be incredibly careful with how they proceed and not to take any risks. It is a high-stress low return situation because is very rare that the business is going to thank you for a successful recovery. In fact they will be much more interested why the corruption happened in the first place.

Once recovery has been achieved, it is very unlikely that the DBA will go back and then spend lots of time trying out various recovery techniques using a copy of the corrupted database on a test server to see how things could have been performed faster and more efficiently, but this is something that we all really should do. Its probably nearly as important as the actual recovery itself, because we can try out various scenarios in a stress-free environment and will learn an awful lot from doing so. We all know why we don't usually do this though, don't we? It is usually a time thing. For those brief periods where we have a lot to do, there is usually very little time to do them. Non critical pieces of work such as this will be left and then finally forgotten about.

# I'm gonna corrupt you!
This is why I decided to forcefully corrupt a database in my test environment during one of my quiet periods and how I seem to have stumbled upon slightly odd behavior with the GUI.

Firstly I decided to create a blank database as my plaything. I then took it offline and decided to trash a small part of the datafile header, after first taking a quick copy it.

Next I attempt to bring the database online through a Query window and the result was what you would expect (complaints all around).

I then tried to bring the database online through the SSMS GUI and the result was again more of the same.

OK, all well and good, so this time I decided to revert the datafile back to normal and bring it back online. Unfortunately the datafile appeared to still be in use. I didn't quite understand how this was so because my broken database was still offline. After a quick check I confirmed that YES there were two open file handles and these were being held by the sqlsrvr process!

# ..Scratching my head
To quickly close these file handles I decided to stop the SQL service, revert the datafile and bring it and the database back online. Next I decided to take the database offline again and put back the corrupted datafile. I then checked that the database file handles were closed (which they were) and attempted to bring the database back online through the Query window. After receiving the expected error I then checked that the file handles were still closed (they were). Repeating this using the GUI and receiving the same error I again checked the file handles. This time they were open again! I could therefore be sure that bringing the database online through the GUI was causing this issue! Is it happening because the GUI is trying to bring the database online differently? Checking a profiler trace, this certainly doesn't look to be the case, but something is obviously different. I have subsequently tried various other corruptions, and the behavior between the GUI and through the Query window has been identical. The behavior must therefore be related to where the corruption is in the datafile, and I think I'm going to do a lot more of this type of testing in the future. Most interesting.

# The moral of this story..
We have all seen lots of operations that when performed through the GUI are performed differently and less efficiently than when we did the same thing via a script -such as adding a new column to a table.
My concern about this situation is that although we could forcefully close the file handles of the DB left open in the SQL server process (trying so appears not to cause a problem), but we are still taking a big risk on the stability of the server, and it is probably not something you should try on a mission critical server. Therefore to avoid issues like these, avoid the GUI where possible!