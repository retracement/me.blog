+++
date = '2011-06-14T08:46:00+01:00'
draft = false
title = 'Database corruption - Your mistake, my problem!'
categories = ['Technology']
tags = ['SQL','Corruption','TSQL Tuesday']
+++

![TSQL Tuesday](/images/2011/tsql2sday150x15032.webp)

This month's T-SQL Tuesday is brought to you by Allen Kinsel ([blog](https://www.allenkinsel.com)|[twitter](https://twitter.com/sqlinsaneo)) who contributes a great amount of his time to the SQL Community and so we can only say a huge thank you for your efforts and keep up the good work! I've not been able to participate in T-SQL Tuesday for a while so it is good to catch this one which is made easier by keeping a close eye on what Rob Farley ([blog](https://sqlblog.com/blogs/rob_farley/)|[twitter](https://twitter.com/rob_farley)) is up to because he is always on time every time! Here's my entry...

Many years ago I started a position at a company that did not understand the value of testing whether a SQL backup was valid. Within my very first few days a problem was discovered in a *very* important database holding extremely important information and on inspection there was found to be serious corruption throughout.

An existing DBA had looked through the archives of backups from tape for this database going back to around three months and there was not a valid backup to be seen (realised after numerous test restores). This was clearly not good and by now the business had been informed, and even more importantly were aware that the problem had (for some reason) been handed onto me! I guess I should have been flattered by this gesture of faith in my abilities, but it felt very much like passing the buck to the new guy. And the buck was firmly in my lap.

![dba against the world](/images/2011/dba-against-the-world.jpg)<br/>
*Fighting back the Business*

In scenarios of corruption on a database I always try not to work on the source material (or even server) if space and resources allow so the very first thing I did was to close down and disable all activity to this problematic database and take a reference backup. Once a reference copy had been taken I could now use it to set about restoring several databases on a different server so that I could have a purely faulty copy and a fixed copy. Therefore on the first copy I ran through a `CHECKDB` (without fix) and used this to identify *where* exactly the problems lie and *what* exactly I was going to do to repair them. The situation was bad I'm afraid, but at least localised in one particular table but seemingly throughout it -from top to bottom. Whenever I tried to run a SELECT statement to query this table it would fail very quickly.

The next thing I tried was to attempt to skip over the problem rows or pages by restricting the query using a `WHERE` clause upon the primary key which allowed it to continue to return some rows until failure at the next point and so on. I then had an idea that I could try to reconstruct the table by querying only the data that was good and worrying about the data that was bad later. So the next thing to do was to query the entire primary key column alone in a `SELECT` query and I was pleased to see that this entire column was good. Upon querying each of the other columns in the table I determined that corruption was not specific to any one column.

My plan therefore was to build up and re-materialize the table by querying it in a checkerboard style using a multi-cursor based solution so that if corruption was found upon a row then the column that caused the error would be skipped over in the next iteration and this worked like an absolute charm. What it did mean of course was that I had resulting holes in the table within specific columns of specific rows, but from a table consisting of millions of records I only now had problems on several hundred rows. The next step was to repair the second copy of the database and for all the datapages that were repaired successfully without data loss I could again use this data to fill holes into my new table where possible.

The problem rows had now reduced in number to around 14 or so, and it was decided to refer to a heavily updated database copy of this production database that had been refreshed to UAT environment several weeks previous and determine whether or not these problem rows had in fact been altered. Our investigations gave us confidence that they hadn't and so these final pieces of the jigsaw were used to fill the final gaps. The Business gave the solution sign off after checking through the data and solution and we put a repaired database with our new table back into production. A success for me and the Business and a lesson learnt for the team in terms of the importance of ensuring backups ARE validated.

---

Sometimes you'll find yourself in situations that have arisen through no fault of your own. Remember that you can't always choose your disaster scenarios or even do anything to prevent them. A disaster is a disaster no matter who caused it or who could have prevented it. When disaster strikes it is important that you are prepared to stir into action, remain calm, focused and most of all try to be inventive in situations that could benefit from this approach.

Good luck!