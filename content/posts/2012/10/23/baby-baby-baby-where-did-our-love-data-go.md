+++
date = '2012-10-23T22:51:00+01:00'
draft = false
title = 'Baby baby baby, where did our ROLLBACK go?'
categories = ['Technology']
tags = ['Concurrency','SQL']
+++

![Frankie Says Relax](/images/2012/frankiesays.jpg)

Possibly one of the very first things we learn as fledgling DBA's is that transactions are used to provide *"all or nothing"* operations. If you ever go for a SQL Server job interview, you are almost guaranteed to be asked exactly what constitutes the ACID properties (**groan**). The A in ACID is of course Atomicity and basically represents the fact that a given set of operations within a transaction either succeeds as an atomic unit OR it doesn't.

What I am about to discuss is probably not the biggest secret to many of the "more informed" of you, and over the years many fantastic speakers (such as Kalen Delaney ([blog](https://sqlblog.com/blogs/kalen_delaney/)|[twitter](https://twitter.com/sqlqueen)) have revealed the truth about SQL Server's transactional behavior.

I first presented this subject (and this behavior) at SQLBits 9 (Liverpool) in 2011, and have subsequently presented the subject a few other times elsewhere (most recently at SQLSaturday #162 Cambridge), and each time never usually fails to spark the alarm of at least one person in the audience.

When I was a relatively new DBA (and I do have to go back a considerable time) I struggled for a very long time to understand exactly what was the point of SQL Server's [SET XACT_ABORT ON](https://msdn.microsoft.com/en-us/library/ms188792.aspx) statement...

...I mean, as I understood it, if transactions failed (and therefore aborted) everything was rolled back anyway wasn't it? So what would be the point of setting XACT_ABORT ON then? This statement just didn't make any sense to me.

Then one day an incident happened at a company I was working for at the time, in which a massive amount of data had gone missing whilst it was being processed by an in-house application. The application would collect the data from a third party vendor, shred it and do funky things to it, before finally placing the data into a SQL database. Upon committing this data, the application would send an acknowledgement message to the third party vendor to provide a confirmation the data had been successfully received and could be removed from their queue.

When the missing data was detected by the application developers, the predictable finger pointing ensued. They blamed us for OUR crummy database and *obviously* misconfiguration server, and we blamed them for THEIR atrociously written application that was somehow (incorrectly) deleting data.

# A typical transactional scenario

We have an orders table in which we wish to remove a record that needs to be fulfilled and insert the (fulfilled) record into the fulfilled table. If the insert fails, it is important that the delete is rolled back and if the delete fails it is important that the insert is rolled back.

The code for this is as follows:

```sql
DECLARE @orderid INT
SELECT @orderid = MAX(id) FROM orders
BEGIN TRAN
   DELETE FROM orders WHERE id = @orderid
   INSERT INTO orders_fulfilled VALUES(@orderid, 4, @@SPID)
COMMIT
```

So in the code snippet above you can see that we are deleting the top orderid from the orders table and inserting it into the orders_fulfilled table. If we execute this code, it does exactly what it says on the tin – deletes the record from orders table and adds it into orders_fulfilled table all within a transactional context. Pretty basic huh?

Now if we rerun the code above, simply changing line 5 as below:

```sql
INSERT INTO orders_fulfilled VALUES(@orderid, 5, @@SPID)
```

The following error occurs...

>(1 row(s) affected)
Msg 547, Level 16, State 0, Line 5
The INSERT statement conflicted with the CHECK constraint "status". The conflict occurred in database "READPAST & Furious", table "dbo.orders_fulfilled", column 'status'.
The statement has been terminated.

So the error message itself is remarkably explanatory. It has told us that our insert statement failed due to the presence of a [CHECK constraint](https://msdn.microsoft.com/en-us/library/ms190377.aspx) on the orders_fulfilled table. Upon checking for open transactions, we note that there are none. Great, this must mean of course that our transaction will have automatically rolled back and undone our delete -right? *WRONG!*

Upon further inspection we note that not only did the insert not complete and that the transaction is no longer open, we observe that the delete did actually occur! The result if you were not expecting it is lost data.

Going back to our in-house application, the developers had introduced some code which was shredding vast volumes of data and performing a similar transactional delete and insert, but had not accounted for the fact that the constraint might be broken by the vendor data OR what would happen if it was. Because the application was acknowledging the receipt and commit of this data to the third party, not only was the data failing to insert into the database, but was also being removed from the vendor's transmission queue.

# Why is this happening?

This is not a bug in SQL Server, far from it. In fact this behavior is by design and occurs only when certain errors are raised within transactions. As you have seen, constraint errors are one such situation and the others that you may run into are (as far as I know still not documented in one single place).

I believe the thinking behind this behavior is to avoid unnecessarily undoing work performed by all the successful statements, which could come in useful in certain expensive transactional loading operations. Besides...if you didn't want this to happen, then you would have implemented an alternative right? 🙂

# What is the solution?
Well I've already mentioned the [XACT_ABORT ON](https://msdn.microsoft.com/en-us/library/ms188792.aspx) statement, and now it suddenly makes sense doesn't it? Since we have shown that transactions are not truly "all or nothing" operations by default, we need a way to direct SQL Server to make them atomic. `XACT_ABORT ON` informs SQL Server that it absolutely **must** rollback any transactions if there are ANY errors during their lifetime or commit everything.

There is a second and more common way of making transactions behave as expected and this is by using error handling. SQL Server 2005 introduced the [BEGIN TRY` and BEGIN CATCH](https://msdn.microsoft.com/en-us/library/ms175976.aspx) syntax which hugely simplified our transaction management. Within our catch-block we can simply perform any clean up actions that we want and [ROLLBACK](https://msdn.microsoft.com/en-us/library/ms181299.aspx) our transaction should we so desire.

So what happens if we use both methods together?
There is a particular problem to be aware of when using error handling in conjunction with `XACT_ABORT ON` and this is really due to the implications of setting this behavior on. By doing so, as we have already discussed, you have told SQL Server to [COMMIT](https://msdn.microsoft.com/en-us/library/ms190295.aspx) everything or roll everything back. When using error handling you are enabling functionality to take custom actions within your transactions. One such action could be a rollback to a [savepoint](https://msdn.microsoft.com/en-us/library/ms188378.aspx), or perhaps to insert logging information to another table.

Let's see what happens with the following code snippet:

```sql
SET XACT_ABORT ON
DECLARE @orderid INT
SELECT @orderid = MAX(id) FROM orders
BEGIN TRY
   BEGIN TRAN
      SAVE TRAN savetohere
      DELETE FROM orders WHERE id = @orderid
      INSERT INTO orders_fulfilled VALUES(@orderid,5,@@SPID)
   COMMIT
END TRY
BEGIN CATCH
   ROLLBACK TRAN savetohere
END CATCH
```

The following error occurs...

> Msg 3931, Level 16, State 1, Line 15
The current transaction cannot be committed and cannot be rolled back to a savepoint. Roll back the entire transaction.

The error we are experiencing here is known as an "uncommitable transaction" and occurs because `XACT_ABORT` is explicit in requiring that the whole transaction is rolled back on error. Performing ANY operation that causes log writes (such as an insert, delete, update or even rollbacks to savepoints) will result in error. It is not obvious from the error above, but rollback is automatic and no transactions are left open when this condition is hit.

So should we ever use `XACT_ABORT ON` then?

There are a couple of situations that using `XACT_ABORT ON` is the right or necessary thing to do (with or without additional error handling). One such situation is when you have unskilled (or lesser skilled) labour executing and promoting SQL scripts into your environments. You (or your developers) want to ensure that the scripts either work OR they do not. If you use distributed transactions then you will be required to use `XACT_ABORT ON`.

In the scenario that you do use both techniques, it is important that within your catch-block you detect any uncommitable transactions before attempting to perform any logging operations. This is performed by detecting the transaction state and the [XACT_STATE()](https://msdn.microsoft.com/en-us/library/ms189797.aspx) function enables us to programatically do this.

For instance:

```sql
-- if there is an open transaction, 
-- then roll it back!
IF XACT_STATE() <> 0 ROLLBACK TRAN 
-- Note that if tran state is -1, then tran uncommittable, 
-- if 0 then no tran open, else if 1 tran is committable
```

---

# Final Thoughts

Understanding SQL Server transaction management is far more difficult than you may realise and it is important that you know the implications of how your developers implement them. In the scenario presented here, we have demonstrated that if they/ you get things wrong then your database may be losing data without you even realizing it. The use of good T-SQL error handling and transaction state detection can be the difference and very simple to implement in the right hands.

Whilst it is common in organizations for Developers and DBA's to point fingers at each other when problems are experienced in your environment, it is important to understand that this usually happens due to poor communications between the two parties and most importantly because you both care. It is *YOUR* job as a DBA not only to protect your data but also ensure that you educate Developers where possible. Likewise it is the Developer's job to ensure performant and accurate application code and educate DBAs. Understanding each other's worlds better and communicating more efficiently can avoid some of the largest (and sometimes more costly) problems with very little effort.