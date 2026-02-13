+++
date = '2016-10-04T15:57:00+01:00'
draft = false
title = 'Transaction Log, Delayed Durability, and considerations for its use with In-Memory OLTP – Part I'
categories = ['Technology']
tags = ['Concurrency','Delayed Durability','Performance','SQL']
+++

![Warhol](/images/2016/warhol_smaller.jpg)<br/>

In this mini-series of posts, we will discuss how the mechanics of SQL Server's transaction logging work to provide transactional durability. We will look at how Delayed Durability changes the logging landscape and then we will specifically see how In-Memory OLTP logging builds upon the on-disk logging mechanism. Finally, we will pose the question *should we use Delayed Durability with In-Memory or not* and discuss this scenario in detail. But in order to understand how delayed durability works, it is first important for us to understand how the transaction log works -so we shall start there...

# Caching caching everywhere nor any drop to drink!

SQL Server is a highly efficient transaction processing platform and nearly every single operation performed by it, is usually first done so within memory. By making changes first in memory, the need to touch physical resources (such as physical disk IOPs) are reduced (in some cases substantially), and reducing the need to touch physical resources means those physical boundaries (and their limitations) have less impact to the overall system performance. Cool right?!

# A little bit about the Buffer Cache

All data access such as insertion, deletion or updates are all first made in-memory, and if you are a Data Administrator, you will (or should!) already be familiar with the Buffer Cache and how it works. When data pages are required to fulfill data access requests (and assuming they are not already present in the Buffer Cache), the Storage Engine first reads those data pages into the Buffer Cache before performing any requested operations on those pages in memory. If the pages in memory are modified, those dirty pages will ultimately need to be persisted directly to disk (i.e. to their respective physical data file/s) through automatic checkpointing and will contain changes performed by either committed or uncommitted transactions. In the latter case of uncommitted transactions, the presence of those dirty pages in the data file/s is why UNDO (found in the transaction log) is performed upon recovery to roll back those changes into the buffer cache. Conversely, any dirty (buffer cache) pages that are a result of committed transaction changes but have not yet been hardened to disk (through `CHECKPOINT`) would require the REDO portion of the transaction log to roll those changes forward on recovery. I will refer back to this again when we move on to talk about In-Memory OLTP in future posts.

# It's all about the log, about the log, no trouble!

![Lolcat](/images/2016/lolcat-interests-330.jpg)

But It is not the hardening of data to data files that we should really be focusing on here (since there is little relevance to delayed durability), we are more concerned about transactional durability in SQL Server.

There is a common misunderstanding with SQL Server DBAs that when transactions are making changes, the transactional changes are written immediately to the logfile. The durability claims of SQL Server make this misunderstanding very easy to understand, but requiring a physical IO operation to occur upon every single transactional data modification would clearly be a huge potential bottleneck (and dependency) upon the transaction log disk performance. The SQL Server architects recognised this challenge, so in a similar way that data page modifications are first cached, so are the transactional changes. In the case of the transaction log, the "cached" area is provided by in-memory constructs known as log buffers that can store up to 60 Kilobytes of transaction log records.

Now that you know that log buffers are there to delay (or *buffer*) transaction logging to maximize the efficiency of a single physical IOP to disk, we must now consider when these structures *must* be flushed to physical disk and stored within the transaction log in order to provide the Durability property of ACID that SQL Server adheres to. There are two main situations a log buffer is flushed:

1. **On transactional COMMIT.** When a client application receives control back after issuing a successful `COMMIT` statement, SQL Server *must* provide durability guarantees about the changes that have occurred within the transaction. Therefore all transactional operations must be written and persisted to the transaction log, ultimately requiring that the log buffer containing those changes must be flushed on COMMIT to provide this guarantee.

1. **On log buffer full.** Strictly speaking, as long as the first rule is adhered to, there is no logical requirement that SQL Server should flush the log buffer to disk when it becomes full. However, if you consider that it is paramount for SQL Server to never run out of available log buffers, then it is pretty obvious that the best way for it to avoid this situation is to hold onto the log buffer only as long as it needs to. If a log buffer is full, then it serves no further purpose in the buffering story and furthermore, given that the log buffer will contain a maximum of 60 Kilobytes of changes, there is good alignment to the physical write to the transaction log. When the log buffer is full it makes sense to flush it to disk.

![transaction log](/images/2016/tranlog_smaller1.png)

---

Now we have a better understanding how the transaction log provides transactional durability to SQL Server and how it delays physical resource writes to the transaction log disk to improve logging performance, we can look at how Delayed Durability changes the logging landscape, why we might use it and what we are risking by doing so in the second post in this series.