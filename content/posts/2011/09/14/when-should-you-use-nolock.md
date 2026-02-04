+++
date = '2011-09-14T12:31:00+01:00'
draft = false
title = 'When should you use NOLOCK?'
categories = ['Technology']
tags = ['SQL','Concurrency']
+++

Well the quick answer to that question is never. And there you have it, the quickest blog post in history (perhaps!).

...Except that I should really have said *"When should you use `READUNCOMMITTED` isolation or hint?"* and then the question takes a little more of a surprising turn. Let me first start and explain why I have categorically ruled against the use of `NOLOCK`. The reason, is that at some point in the dim and distant future Microsoft may decide to remove this hint from the product. If they could do it today, they probably would; the only thing that is really stopping them is the numerous lines of external T-SQL code that dirtys the floors of many Development teams around the world.

Of course technically `READUNCOMMITTED` and `NOLOCK` are identical with the exception that the former can also be used to set the session level `ISOLATION LEVEL` as well as the statement level.

Now, moving swiftly onto a surprising find by myself whilst I was putting together material and investigating ideas for my *READPAST & Furious* presentation, which is based loosely on the excellent presentations that Kalen Delaney has given over the last 6 years or more on the subject of Locking, Blocking and Concurrency. It is a homage of sorts. During her SQL PASS Summit 2010 Pre-Conference day titled *Locking and Blocking and Row Versions, Oh My!*, there was a question posed about whether there was ever a good reason to use the `NOLOCK` hint. The response was (quite unsurprising) *"not really, maybe only for troubleshooting"*.

Another thing that is commonly said about `NOLOCK` is that it does not take out Shared locks and this is the reason why when using the `NOLOCK` (or `READUNCOMMITTED`) hint, we are able to read uncommitted data. Since a Shared lock is not taken or requested, there is no incompatibility detected by the lock manager and therefore no waiting for Exclusive locks to be released from any dirty pages.

Now imagine my surprise when I was playing with several `SELECT` statements and came across locks being acquired when using `READUNCOMMITTED` (or `NOLOCK` dont forget!) hint. So if I perform a simple `SELECT * FROM orders WITH (READUNCOMMITTED)` from an entire table we can see what locks (if any) are acquired...

![HOBT locks](/images/2011/readcommitted1a.jpg)<br/>
*Ooooh look Ma, Locks!!!*

Now perhaps less of a surprise is the acquiring of a *Sch-S* lock (otherwise known as a Schema Stability lock) on the table itself. This is (obviously) being acquired in order to prevent any *Sch-M* locks being taken whilst the `SELECT` executes so that the table structure cannot be changed, and this happens with every query. In fact if you look at Books Online it evens warns you that this is what will happen:

> READUNCOMMITTED and NOLOCK hints apply only to data locks. **All queries, including those with READUNCOMMITTED and NOLOCK hints, acquire Sch-S (schema stability) locks during compilation and execution.** Because of this, queries are blocked when a concurrent transaction holds a Sch-M (schema modification) lock on the table. For example, a data definition language (DDL) operation acquires a Sch-M lock before it modifies the schema information of the table. Any concurrent queries, including those running with READUNCOMMITTED or NOLOCK hints, are blocked when attempting to acquire a Sch-S lock. Conversely, a query holding a Sch-S lock blocks a concurrent transaction that attempts to acquire a Sch-M lock. For more information about lock behavior, see Lock Compatibility (Database Engine).

Much more surprising is the acquiring of the Shared lock which is classed as a *[BULK_OPERATION]* in the *TextData* column. Now this one was totally unexpected and if we take a look at the Type column we see the value *12=HOBT* (Heap or B-Tree). This table my friends is in fact a *Heap*, which is only of slight relevance to this investigation but is worth remembering. So to summarize so far, *using `READUNCOMMITTED` hint in our `SELECT` against a Heap causes a Schema Stability lock on table object AND a special [Bulk Operation] Shared lock on the Heap*!

Next thing I am now going to do is run the `SELECT` statement under `READ COMMITTED` isolation. Doing so we get the following results and they are just as we expected, a single *Intent Shared* lock on the table object and multiple *Shared* locks acquired and released on each and every Page.

![Read Committed on heap](/images/2011/rconheap.jpg)<br/>
*Normality*

Now then, let us forget for a moment the *uncommitted* argument when using `READUNCOMMITTED` isolation and just agree on one thing -locking was more efficient in the first example wasn't it?! From a resource perspective, having to acquire and release all those page locks on a very large table would take it's toll in a highly transactional system.

Now the next thing I am going to do is run the `SELECT * FROM orders` statement in `SERIALIZABLE` isolation, and again no real surprises with what happens. You see that an *Intent Shared* lock and a *Shared* lock is acquired (and subsequently released) on the table.

![Read under serializable](/images/2011/readserializable.jpg)<br/>
*Readings are normal Jim*

What I am going to do now is to create a clustered index on the table and repeat the `SELECT` using `READUNCOMMITTED` and the result (from a locking perspective) is even better as you can see below.

![Read under uncommitted](/images/2011/readuncommitted2.jpg)<br/>
*We only have stability*

So this time the `READUNCOMMITTED` lock gives us an even more efficient turn around once the clustered index is in place, it simply takes out a single *Schema Stability* lock on the table. We go back then to the original question, *"When should you use ~NOLOCK~ READUNCOMMITTED?"* and I shall tell you! *Whenever you are querying tables that are housed in read only databases, read only filegroups, read only file systems, Scalable Shared Databases (in short committed data that is read only)* it makes perfect sense to use a locking strategy that uses no locking due to the following reasons:

1. Improves concurrency
1. Reduces lock manager overhead
1. Reduces memory usage of locks

And what of the dirty data you might ask? If it is read only there is no danger of it being changed and therefore no danger of lost updates or dirty reads. In other words (think of the film The Matrix here)... *"Do not try and bend the ~spoon~ isolation. That's impossible. Instead... only try to realize the truth. There is no ~spoon~ isolation (required)"*.