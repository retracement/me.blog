+++
date = '2012-10-17T13:40:00+01:00'
draft = false
title = 'The Curious Case of the Non-Escalating transaction locks'
categories = ['Technology']
tags = ['Concurrency','SQL']
+++

SQL Server has many facets and behaviors that are often misunderstood and do not always work quite as we may have expected. At first you may believe that you understand the mechanics, only to realize after a little bit of delving deeper that your comprehension was wrong.

SQL Server Lock Escalation is one of these commonly misunderstood topics, and most recently after doing a little investigation into a few subjects related to escalation I was shocked and surprised in equal measure to find that things were not working quite as I expected.

As most of you will know, the reason SQL Server has an escalation mechanism is to provide a balance between concurrency and efficiency. On 64-bit SQL Server, since each lock consumes 128 bytes of memory and lock owner block (process holding the lock) consumes 64 bytes of memory, locking can start to consume an awful lot of memory and have an impact on system performance if escalation did not exist.

By default, there are really two reasons why escalation will occur:

1. A result of too many locks taken out on a resource by a statement (the ball-park figure is just over 5000).
1. The second is due to the amount of lock memory consumption (roughly just over 40% of the current SQL Server working set).

So let us assume that we have a transaction where we are making updates to many rows in a table.

For example:

```sql
BEGIN TRAN
UPDATE database1.dbo.table1
SET C2=1 WHERE C1 < 1000;
```

Now I don't think it will be a huge surprise to anyone when we see that we have taken 1000 Exclusive (*X*) locks and 3 Intent-Exclusive (*IX*) locks.

![Locks](/images/2012/lock1.jpg)

So now what happens if you run the following statement inside the same transaction:

```sql
UPDATE database1.dbo.table1
SET C2=1 
WHERE C1 >= 1000 
   AND C1 = 2000 
   AND C1 = 3000 
   AND C1 = 4000 
   AND C1 = 5000 
   AND C1 < 6000;
```

Just looking at the whole transaction statement you would assume that since around 6000 locks are being taken on the same resource we would get escalation occurring. Let's take a look:

![No escalation](/images/2012/locks2.jpg)

As you can see, escalation hasn't happened. This can however be explained by the statement I made earlier, that escalation will occur if the threshold is exceeded for a single statement on a single resource. I have performed tests and found that there does not appear to be any upper limited on the number of locks you can take on a single resource as long as each statement does not break the threshold and the memory threshold is not broken.

So now let us try to get escalation occurring. We shall first execute the following in the same transaction:

```sql
UPDATE database1.dbo.table1
SET C2=1 
WHERE C1 = 6000 
   AND C1 < 12000;
```

We have now issued a statement to update 6000 (new) rows, this statement is still being run within the same transaction on the same resource and as such we would expect it to trigger an escalation.

![Lock escalation](/images/2012/locks3.jpg)

This time escalation of all the Exclusive key locks and Intent-Exclusive page locks have been successfully escalated to a single object level Exclusive lock as expected.

---

# Conclusion

When I first saw this escalation behavior I was of the opinion that it was indeed a bug that I had uncovered. Over time and reflection I believe that it is not so much a bug as it is an oddity. The stated behavior of SQL Server to escalate due to the number of locks on the same resource by the same statement is being adhered to, although as I pointed out, if the locks have already been acquired, then these do not count towards the final threshold evaluation.

You could argue that perhaps SQL Server should have an optimization to cumulatively add the total number of locks owned by a SPID on a resource and compare this figure against the threshold limit, but I assume the Database Engine team must have ruled that out for concurrency optimization.

Can you use this oddity in a useful way? Perhaps...
Lets assume you need to perform a large data set change on a very large table or partition which is highly active. The data must be changed and performed whilst your environment is live, but you wish to minimize the impact to the table's concurrency. (I should also add here as an aside that escalation is not guaranteed if incompatible locks exist when it is attempted, but do you really want to run the risk that it could happen in this proposed scenario?)

Your first thought may be to change the data in batches, but perhaps you require that the whole operation is transitionally consistent? All of the dataset is updated OR it is not.

In this situation you could break down the updates into logical statement batches, ensuring that the number of locks taken by each does not cross the threshold. These statements can then be run inside an explicit transaction and escalation will not occur (or be attempted) due to thresholds not being crossed. Therefore you will only lose concurrency to those rows themselves and not require or attempt exclusive partition or object locks. And whilst it is possible to turn off escalation completely due to the number of locks (TF1224) or on the specific table itself (as of SQL 2008), this is not always possible or desirable in certain environments and this method could prove to be a far more satisfactory way around the problem.

---

# One Final Note

Hopefully this post has demonstrated that SQL Server mechanisms do not always work the way you expect (or have read in a book). With a little bit of testing and observation, it is usually possible to understand what is really going on. So don't delay and dive into your test environment!