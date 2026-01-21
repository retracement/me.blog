+++
date = '2010-08-22T09:26:00+01:00'
draft = false
title = 'How to query your database relationships'
+++

Last year I wrote about how to manage your table constraints in [Check your constraints to check your integrity](/posts/2009/07/22/checking-your-constraints-to-check-your-integrity) but one thing that is missing from this post is how do you determine what relationships exist between tables in order for you to temporarily disable them. This can be particularly tiresome on relationship heavy schemas.

Luckily there are two quite useful stored procedures that you can use to show you them and they are [sp_pkeys](https://msdn.microsoft.com/en-us/library/ms189813.aspx) and [sp_fkeys](https://msdn.microsoft.com/en-us/library/ms175090(v=SQL.105).aspx). The first stored procedure will display a primary key for a given table and the second can display a list of all tables with a defined foreign key relationship to the specified primary table.

One thing to be aware of though, is that if you specify an invalid table name or table owner you will still have an empty result set returned rather than an error, which is not ideal in my opinion, so just be sure that you specify those details correctly and if you get an empty result set then it is worth checking your statement again.

![Relationship stored procedures](/images/2010/pkfkkeys.png)

Another possible method to determine the relationships that you can use through SSMS is to create a database diagram and add in all the tables. This can work quite well for smaller schemas but generally can become quite unwieldy for larger ones, so by far an the easiest way is to use the stored procedures.