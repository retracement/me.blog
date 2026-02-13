+++
date = '2013-07-17T13:44:00+01:00'
draft = false
title = 'Why don’t you go and LOCK OFF!'
categories = ['Technology']
tags = ['Concurrency','SQL']
+++

No this is not a Hekaton post 🙂 -perhaps I should have saved this title for another rainy day.

SQL Server concurrency is a particularly difficult subject to master and understanding the result of your actions can be confounded by some of SQL Server's special behaviors and quirks that I so often like to talk and write about.

As some of my ~followers~ stalkers out there may remember, one of my favorite demonstrations I like to give is during my
[READPAST & Furious: Transactions, Locking, and Isolation](https://www.sqlpass.org/summit/2012/Sessions/SessionDetails.aspx?sid=2816) session -last performed at the [SQLPASS 2012 Summit](https://www.sqlpass.org/summit/2012/) and [SQLSaturday #229 Dublin](https://www.sqlsaturday.com/viewsession.aspx?sat=229&sessionid=14975) where in a little piece of code I discuss whether a transaction is really Atomic (or not). For anyone reading this paragraph who has immediately answered "ALWAYS" -then my friend you are in serious trouble with your code-base. And I am not even talking about the use of lower levels of isolation to perform dirty reads within your transaction here, isolation does not effect a transaction's atomicity.

No, what I am talking about how SQL Server (under certain conditions) can fail a statement in a transaction, but still successfully commit. If this is not the behavior that was intended for your application then congratulations! You now have an inconsistent dataset. You can read a quite exhaustive post on this subject from my post [Baby baby baby, where did our ROLLBACK go?](/posts/2012/10/23/baby-baby-baby-where-did-our-love-data-go).

![bad lock](/images/2013/craplock.jpg)

Yesterday I was talking with a colleague (and fellow speaker) about a particularly bad concurrency issue at our current place of work and he told me that one quick and dirty proposal was to limit the duration of problematic locks, it had been suggested to set the `LOCK_TIMEOUT` for each session. Straight away, my alarm bells started ringing and I replied that this is probably a very bad idea. I could talk for hours on why it is a bad idea from a concurrency perspective -it is a bit like asking someone to attempt to cross a busy road and return to the Kerb (Sidewalk for my American buddies!) each time if a car is coming. However I won't bore you with that detail, I will simply say that once you have read my post (referenced above) I will add that one such behavior (that allows the failure of a statement but does not cause a rollback) is using the LOCK_TIMEOUT session setting. Oh yes I know a lot of you out there are using it, and yes you had better start praying 🙂

In the following code snippet we demonstrate this behavior by updating one record in a table in an open ended transaction (which will take an Exclusive lock on this row). In a second connection we will run a second transaction in a session using a LOCK_TIMEOUT that first inserts a record into this table and then attempts to delete our locked record (and triggering the lock timeout).

Let's first create a table with a single record and exclusively lock that record out under an open ended transaction:

```sql
--connection 1
CREATE TABLE t1 (c1 INT)
GO
 
INSERT INTO t1 VALUES ('1');
 
BEGIN TRAN
    --update value in open ended
    --transaction to take exclusive
    --lock where c1=1
    UPDATE t1 SET c1 = 3 WHERE c1=1
```

Next we will run a transaction in another connection to insert one record into this table and attempt to remove the blocked record:

```sql
--connection 2
SET LOCK_TIMEOUT 10
BEGIN TRAN
    INSERT INTO t1 VALUES ('2');
    DELETE FROM t1 WHERE c1 = 1;
COMMIT
--after lock timeout view committed values
SELECT * FROM t1 WITH (READPAST);
```
The following error occurs...

>(1 row(s) affected)
Msg 1222, Level 16, State 45, Line 5
Lock request time out period exceeded.
The statement has been terminated.
>
>(1 row(s) affected)

Now you can quite clearly see that the LOCK_TIMEOUT worked, but let us now take a look at the committed contents of the table (skipping over the locked record using the `READPAST` hint):

```sql
--connection 2
SELECT * FROM t1 WITH (READPAST);
```

```text
c1
———–
2

(1 row(s) affected)
```

Yes transaction fans you will see that the row we inserted under an *Atomic* transaction committed successfully whilst the delete failed. I am not going to labour this point any more than to say, please be careful to understand any setting you enable for concurrency optimization and troubleshooting. If you are not careful, then you may live to regret it!