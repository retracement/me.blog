+++
date = '2018-09-01T14:38:00+01:00'
draft = false
title = 'In-Memory logging and log "compression"'
categories = ['Technology']
tags = ['SQL','In-Memory OLTP','Transaction Log']
+++

Introduced with SQL Server 2014, In-Memory OLTP (IMOLTP) provided a new optimistic concurrency model which implemented a lockless and latchless mechanism resulting in far better throughput for short concurrent OLTP workloads. Unlike the traditional on-disk concurrency model, IMOLTP stores table data as data rows in-memory (rather than in-buffer page structures) and this data (or changes to it) will eventually be asynchronously written into checkpoint file pair structures on-disk. Given that the speed of physical disk is no longer a major consideration for the speed of data changes when compared against the traditional on-disk concurrency model, you would presume that we no longer need to worry too much about their performance. Unfortunately, we have something else to consider – the transaction log.

The transaction log is almost certainly going to become your biggest concern for potential bottlenecks with databases enabled for IMOLTP since (for [SCHEMA_AND_DATA](https://docs.microsoft.com/en-us/sql/relational-databases/in-memory-oltp/defining-durability-for-memory-optimized-objects) In-Memory tables at least) the transaction log is still required to persist transaction log records for durability purposes. In other words, under default behavior, even In-Memory transaction log records must be flushed and written to the transaction log when those transactions commit.

Luckily IMOLTP provides several logging benefits including the following:

- No index logging overhead. Only index definitions are persisted meaning that they are rebuilt upon database recovery (should that happen!)
- No undo logging overhead. Undo is unnecessary because In-Memory data structures are not overwritten and instead, new structures are created (on commit, the old structures are marked for garbage collection). Rollback, therefore, only requires the old structures to remain.
- Log record ordering by Transaction End Timestamp. This removes the requirement for a single log stream (implemented in SQL 2016).
- Log record "compression". We will examine this in more detail below.

# Log record "compression"

It is perhaps important to highlight the fact that the term Log record *"compression"* is something that I have coined myself, and may or may not be the correct or official term for this feature. However, I have not managed to find any official terminology for it to date, but after reading further I think it will become clear why I am using it. Before we do that, first let's examine logging with on-disk tables.

**Logging with on-disk tables**

I will first create a simple table called *Assimilations*:

```sql
CREATE TABLE Assimilations
    (id INT IDENTITY,
    assimilation_date datetime DEFAULT getdate(),
    NewBorg INT,
    Details CHAR (50));
GO
```

Next we will execute a multi-statement transaction:

```sql
BEGIN TRAN
        INSERT INTO Assimilations (assimilation_date, NewBorg)
            VALUES (GETDATE(), 10);
        INSERT INTO Assimilations (assimilation_date, NewBorg)
            VALUES (GETDATE(), 15);
        INSERT INTO Assimilations (assimilation_date, NewBorg)
            VALUES (GETDATE(), 5);
        INSERT INTO Assimilations (assimilation_date, NewBorg)
            VALUES (GETDATE(), 7);
COMMIT
```

This committed transaction's log records should now be available to query using fn.dblog. We first find the most recent transaction id in the log for our operation, and then use that to pull back all log records for it:

```sql
DECLARE @TransactionID NVARCHAR(14)
DECLARE @CurrentLSN NVARCHAR(23)
SELECT TOP 1 @TransactionID =
        [Transaction ID], @CurrentLSN = [Current LSN]
    FROM    sys.fn_dblog(NULL, NULL)
    WHERE   Operation = 'LOP_INSERT_ROWS'
    ORDER BY [Current LSN] DESC;
 
SELECT  [Current LSN], [Operation], [Transaction ID]
    FROM    sys.fn_dblog(NULL, NULL)
    WHERE   [Transaction ID] = @TransactionID
    ORDER BY [Current LSN] ASC;
GO
```

This returns the following results:

```text
Current LSN             Operation         Transaction ID
----------------------- ----------------- --------------
00000024:00000378:0001  LOP_BEGIN_XACT    0000:00000384
00000024:00000378:0002  LOP_INSERT_ROWS   000:00000384
00000024:00000378:0003  LOP_INSERT_ROWS   0000:00000384
00000024:00000378:0004  LOP_INSERT_ROWS   000:00000384
00000024:00000378:0005  LOP_INSERT_ROWS   0000:00000384
00000024:00000378:0006  LOP_COMMIT_XACT   0000:00000384
 
(6 rows affected)
```

As you can see, we have a total of six log records returned. The first being the transaction `BEGIN` log record followed by four `INSERT` log records and terminating with one `COMMIT` log record. You will see that these quite obviously align with our transactional statements.

**Logging with In-Memory tables**

For this example I will create a similar but basic IMOLTP table called *AssimilationsIM*:

```sql
CREATE TABLE AssimilationsIM
    (id INT IDENTITY PRIMARY KEY NONCLUSTERED HASH
        WITH (BUCKET_COUNT=32) NOT NULL,
    Assimilation_Date datetime DEFAULT getdate(),
    NewBorg INT,
    Details CHAR (50))
    WITH (MEMORY_OPTIMIZED=ON,
    DURABILITY = SCHEMA_AND_DATA)
GO
```

And of course we still need to execute a multi-statement transaction (as before) but this time inserting values into our new in-memory table:

```sql
BEGIN TRAN
        INSERT INTO AssimilationsIM (assimilation_date, NewBorg)
            VALUES (GETDATE(), 10);
        INSERT INTO AssimilationsIM (assimilation_date, NewBorg)
            VALUES (GETDATE(), 15);
        INSERT INTO AssimilationsIM (assimilation_date, NewBorg)
            VALUES (GETDATE(), 5);
        INSERT INTO AssimilationsIM (assimilation_date, NewBorg)
            VALUES (GETDATE(), 7);
COMMIT
```

Again our committed transaction's log records should now be available to query using `fn.dblog`. As before we first find the most recent transaction id in the log for our operation (this time we are looking for the *LOP_HK* operation rather than the *LOP_INSERT_ROWS* operation), and then use that to pull back all log records for it:

```sql
DECLARE @TransactionID NVARCHAR(14)
DECLARE @CurrentLSN NVARCHAR(23)
SELECT TOP 1 @TransactionID =
        [Transaction ID], @CurrentLSN = [Current LSN]
    FROM    sys.fn_dblog(NULL, NULL)
    WHERE   Operation = 'LOP_HK'
    ORDER BY [Current LSN] DESC;
 
SELECT
    @TransactionID AS '[Transaction ID]',
    @CurrentLSN AS '[Current LSN]'
 
SELECT  *
    FROM    sys.fn_dblog(NULL, NULL)
    WHERE   [Transaction ID] = @TransactionID
    ORDER BY [Current LSN] ASC;
```

Interestingly this now returns the following results:

```text
Current LSN             Operation         Transaction ID
----------------------- ----------------- --------------
00000024:00000478:0002  LOP_BEGIN_XACT    0000:00000396
00000024:00000478:0003  LOP_HK            0000:00000396
00000024:00000478:0004  LOP_COMMIT_XACT   0000:00000396
 
(3 rows affected)
```

This time, we have a total of three log records returned. The first and last being the transaction `BEGIN` and `COMMIT` log records as before, but now we have a single log record sandwiched in-between. We can use an undocumented in-memory function (`fn_dblog_xtp`) to break this record open based up our transaction id:

```sql
SELECT
    [Current LSN],
    operation_desc,
    Operation,
    [Transaction ID]
    FROM    sys.fn_dblog_xtp(NULL, NULL)
    WHERE   [Current LSN] = @CurrentLSN
    ORDER BY [Current LSN] ASC
GO
```

This now yields the following:

```text
Current LSN             operation_desc      Operation    Transaction ID
----------------------- ------------------- ------------ --------------
00000024:00000478:0003  HK_LOP_BEGIN_TX     LOP_HK       0000:00000396
00000024:00000478:0003  HK_LOP_UPDATE_ROW   LOP_HK       0000:00000396
00000024:00000478:0003  HK_LOP_INSERT_ROW   LOP_HK       0000:00000396
00000024:00000478:0003  HK_LOP_INSERT_ROW   LOP_HK       0000:00000396
00000024:00000478:0003  HK_LOP_INSERT_ROW   LOP_HK       0000:00000396
00000024:00000478:0003  HK_LOP_INSERT_ROW   LOP_HK       0000:00000396
00000024:00000478:0003  HK_LOP_COMMIT_TX    LOP_HK       0000:00000396
 
(7 rows affected)
```

We can now see the same number of `BEGIN`, `INSERT`, and `COMMIT` log records as with the on-disk transaction. This means that IMOLTP has somehow *"compressed"* our multiple log records into one single log record. This is made possible by another optimization that we haven't yet mentioned, although it could have perhaps been inferred by the fact that *UNDO* is not logged. Basically, IMOLTP transactions do not cause any physical log IO until they have been committed and allows these multi-statement log records to be rolled into one.

On a final note, I have yet to figure out what exactly the *HK_LOP_UPDATE_ROW* record is recording, but it is my suspicion that this is related to an internal meta operation or something to do with the hash index on the table. I may update this post when this mystery is solved.

# Summary

In summary, IMOLTP has many logging efficiencies baked into it out of the box. If the log file of your database is one of your biggest bottlenecks, then using this technology in the right situation could be a very good idea!