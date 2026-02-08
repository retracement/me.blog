+++
date = '2014-03-05T09:30:00+01:00'
draft = false
title = 'ROLLBACK damn you!'
categories = ['Technology']
tags = ['Concurrency','SQL']
+++

A good question was asked on #SQLHELP the other day regarding the use of `XACT_ABORT ON`:

>If you don’t have a `BEGIN TRAN COMMIT TRAN` in a stored procedure, will SET XACT_ABORT ON do anything?

My immediate reply to this was:

>Yes it makes the transaction atomic. See my blog post [Baby baby baby, where did our ROLLBACK go?](/posts/2012/10/23/baby-baby-baby-where-did-our-love-data-go).

In response to this, the follow-up question was raised:
>wouldn’t executing the stored procedure by itself be atomic?

So I said:
>Not quite. An implicit or explicit transaction is not atomic by default in SQL Server depending upon errors” then “and a proc doesn’t equate to a transaction. Check out my READPAST & Furious presentation at *PASS SUMMIT* or *SQLBITS* for demos.

You really have to love Twitter sometimes for the 140 Character limit, and it really can cause a certain level of ambiguity in answers that can sometimes cause confusion or vagueness to replies, and I felt that it is probably worth elaborating further in a post where the only restriction is that of the editor (Moi) getting tired of typing!

Fellow Concurrency Junkie Roji Thomas ([Twitter](https://twitter.com/rojipt)|[Blog](https://sqlindian.com/)) later sent me a tweet which read:
>When there is no explicit transactions, are you suggesting that XACT_ABORT has an effect on single statement implicit transactions?

The answer is of course no, but I knew exactly what he was getting at and he was in effect quite rightly suggesting that there was an element of ambiguity in my reply.

---

So at this stage it might be a good idea for me to describe what XACT_ABORT does. If you want the official blurb on this directive you can look in [Books Online](https://technet.microsoft.com/en-us/library/ms188792.aspx), but I’ll boil it down in a nutshell. Put simply, setting `XACT_ABORT ON` has the effect on ensuring that any open transaction will be treated as purely atomic (all statements will complete successfully otherwise the transaction will completely rollback). By default `XACT_ABORT` is `OFF`, and more importantly there exist certain errors that will not cause a transaction to fail (even though that single statement might have). Examples of these are constraint failures and lock timeouts which have been discussed in detail elsewhere in this blog.

Setting `XACT_ABORT ON` is one way then to ensure that your transactions become all or nothing operations (yes, just like you thought they were already!) and there are certain considerations to bear in mind when using it. Again, rather than me regurgitate them, check out my other blog posts on this subject and my [READPAST & Furious presentation last recorded at SQLPASS Summit 2012](https://www.sqlpass.org/summit/2012/Sessions/SessionDetails.aspx?sid=2816) which describe its usage fully.

So going back to Roji’s comment, what he was getting at was that under the default behaviour of SQL Server, essentially the following set statements are essentially identical in possible behaviour and outcomes:

```sql
--transaction 1
INSERT INTO dbo.mytable VALUES(1)
 
--transaction 2
BEGIN TRAN
INSERT INTO dbo.mytable VALUES(1)
COMMIT
 
--transaction 3
SET XACT_ABORT ON
INSERT INTO dbo.mytable VALUES(1)
SET XACT_ABORT OFF
 
--transaction 4
SET XACT_ABORT ON
BEGIN TRAN
INSERT INTO dbo.mytable VALUES(1)
COMMIT
SET XACT_ABORT OFF
```

There are two important points of note. The first is that one of the reasons that these sets of statements are pretty much identical in behaviour is because we only have a single DML operation occurring in each. If we are not explicitly defining the transaction then the single DML operation would itself by default imply and begin and commit (or rollback) as a single unit of work (or transaction). The second point of note is that because we are running our connection using default SQL settings, `IMPLICIT TRANSACTIONS` is off (more on this in a second). So the point that Roji was raising, was that under these conditions, the setting of `XACT_ABORT` is irrelevant and I fully agree with this assertion.

This story becomes slightly more complicated when we start talking about implicit transactions. If we `SET IMPLICIT_TRANSACTIONS ON`, then the behaviour of transaction 1 changes slightly. Again, you can refer to the SQL DBA’s favourite resource ([Book’s Online](https://msdn.microsoft.com/en-us/library/ms187807.aspx)) for more information on this command, but essentially when implicit transactions are set to ON, it causes SQL Server to imply open ended transactions. In other words, now when a transaction has been implied, you will explicitly have to commit it (Oracle style). So in the case of transaction 1, there is the possibility that your T-SQL batch may contain further DML operations (which would all be automatically enlisted into this open transaction). In this scenario setting `XACT_ABORT ON` would have an effect and distinct difference to the same batch with `SET IMPLICIT TRANSACTION OFF` (the default).

For instance:

```sql
--transaction 5
SET IMPLICIT_TRANSACTIONS ON
INSERT INTO dbo.mytable VALUES(1)
INSERT INTO dbo.mytable VALUES(5)
COMMIT
 
--transaction 6
SET IMPLICIT_TRANSACTIONS ON
SET XACT_ABORT ON
INSERT INTO dbo.mytable VALUES(1)
INSERT INTO dbo.mytable VALUES(5)
COMMIT
SET XACT_ABORT OFF
```

In the example above transaction 5 is not quite the same as transaction 6. Transaction 5 could encounter a constraint violation by one of its statements, but still commit the other, whereas transaction 6 would automatically rollback in that situation.

In summary, you should always be careful when changing the default behaviours of SQL Server, and more importantly even when you haven’t, make sure you understand that the default behaviour matches that which was expected and required.

The setting of `XACT_ABORT` is irrelevant for single DML statements in isolation since they are *ALWAYS* atomic regardless, however the complications (and considerations) arise when a transaction spans multiple DML statements and could result in different behaviour to that which you were expecting. Setting `IMPLICIT_TRANSACTIONS` to `ON` will result in the duration of a transaction to last until it is explicitly committed and therefore the scope of your transaction might not be so obvious to you simply by looking at the code.

Good luck!