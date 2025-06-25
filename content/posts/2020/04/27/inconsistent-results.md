+++
date = '2020-04-27T12:05:21+01:00'
draft = false
title = 'Inconsistent result sets and another case against pessimistic isolation'
+++
I’m a big fan of demonstrating when transaction processing goes bad in Database Management Systems because it is a reminder to us all to ensure that we know exactly what we are doing when we write our code. Not only should you test and execute your code multiple times in serial (for instance running in sequence in isolation), but you should also execute in parallel with itself or other potentially dependent transactions.

Elsewhere in this blog you will find many other concurrency and consistency examples which will make your hairs stand on end with fear and dread of the dangers that might lurk in your system, and whilst I would love to claim full credit for this latest toe-curling installment, I feel it is only fair to blow smoke towards the mighty Erland Sommarskog ([b](http://www.sommarskog.se/) for first raising a similar observation within a private forum many months ago. The comments and code are mine.

As we all know, Database Management Systems are designed with consistency in mind. That is, the level of consistency we observe should adhere to the level of isolation we chose. You will all no doubt be aware by now that the use of `NOLOCK` locking hint (or more specifically the `READ UNCOMMITTED ISOLATION`) is our way of telling the DBMS to favor concurrency over consistency and lead to reading result sets that are either incomplete or have not yet been committed. In other words, we are able to read in-flight transactional changes.

Both (on-disk) pessimistic and optimistic isolation in SQL Server implement pre-defined behaviors with the use of row, page, or object-level locks for the non-in-memory (or traditional/ on-disk) levels of isolation. Even on-disk “optimistic” isolation takes out write-level locking (but uses row versioning for reads). Locking (and latching) is often the root cause of many transaction processing bad behaviors that we find in SQL Server.

---

# Scenario
We have a table called Cars which is pre-loaded with 10 records.

We will have two user sessions, both running under (the default) `READ COMMITTED` isolation level. The first user session is continuously looping whilst each time executing a single transaction that deletes all records in the Cars table before inserting 10 records and finally committing. The second user session is continuously looping over a select statement to query our table.
Therefore, given that the delete and insert during session 1 are isolated under a transaction, we would expect that the query from session 2 would either be temporarily blocked (by session 1) OR return 10 records.

![Cars Concurrency](/images/2020/cars-graph.webp)

# Implementation
In this section we will detail how to implement these tests

**Setup**

My examples utilize the Cars table. You can set this up as follows:

```sql
CREATE TABLE Cars (
   id uniqueidentifier DEFAULT NEWID(),
   carname VARCHAR(20),
   lastservice datetime DEFAULT getdate(),
   SpeedMPH INT, Details CHAR (7000) CONSTRAINT [PK__Cars] PRIMARY KEY CLUSTERED ([id])
)
```
Now we need to configure two sessions that will run continuously until we hit a failure.

**Session 1**

In our first session, we start with 10 records in the Cars table and all subsequent changes are made within a transaction scope deleting and inserting 10 records at a given time. We understand there should be an atomic guarantee that success or failure of the transaction will ensure that 10 records should only ever be visible outside of Session 1 regardless of that outcome.

There is a tiny delay implemented within this first transaction, and whilst it is not the root cause of this problem, it helps to exacerbate the issue in testing. Feel free to do testing with our without.

Our Session 1 code is as follows:

```sql
WHILE 1=1
BEGIN
   BEGIN TRAN
      DELETE FROM Cars
 
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Ferrari', 170, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Porsche', 150, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Lamborghini', 175, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Mini', 110, '')  
      WAITFOR DELAY '00:00:00.02'
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Datsun', 90, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Ford', 125, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Audi', 138, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('BMW', 120, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Honda', 87, '')
      INSERT INTO Cars(Carname, SpeedMPH, Details) VALUES('Mercedes', 155, '')   
   COMMIT TRAN
END
```

Remember that out of the box (under `READ COMMITTED` isolation level), SQL Server uses exclusive locks to serialize data access to the table data and block access by other sessions to it until they are released. The exclusive locks are of course held until the end of the transaction.

**Session 2**

In our second session, the code will break if we manage to query less than 10 records from Cars table. If we successfully read 10 records we increment a `@ConsistentResults` counter and attempt another read. We expect session 2 to continue running, but if it breaks we will see how many times we managed to read 10 records.

It is worth pointing out that when we query session 2, the results are inserted into a table variable. The only reason we do this is so that we can count how many records are queried and return those rows back later.

Our Session 2 code is as follows:

```sql
DECLARE @Cars TABLE (id uniqueidentifier DEFAULT NEWID(), carname VARCHAR(20), 
    lastservice datetime DEFAULT getdate(), SpeedMPH INT, Details CHAR (7000));
DECLARE @ConsistentResults INT = 0
WHILE 1=1
BEGIN
    DELETE FROM @Cars
    INSERT INTO @Cars SELECT * FROM Cars
    IF @@ROWCOUNT <> 10
        BREAK
 
    SET @ConsistentResults = @ConsistentResults + 1
    WAITFOR DELAY '00:00:00.013'
END
SELECT @ConsistentResults AS SuccessfulPriorRuns
SELECT * FROM @Cars
```

# Test results

Our first run of Session 2 code gives us:

```text
SuccessfulPriorRuns
-------------------
4
 
id                                   carname              lastservice             SpeedMPH    Details
------------------------------------ -------------------- ----------------------- ----------- -------
4587384F-E79A-4D4C-9A69-1D969F31E431 Porsche              2020-04-19 18:58:25.627 150
F9AC6FFF-51B7-44E6-B773-311B17F0C5D7 Lamborghini          2020-04-19 18:58:25.627 175
3C0C196A-95DE-40F5-B6A3-432574CECBA5 Ford                 2020-04-19 18:58:25.640 125          
5F844E89-13A4-4D46-874A-55513F389A3C Audi                 2020-04-19 18:58:25.640 138    
BADB1001-C4F2-4C7E-A65C-5A4099C43244 BMW                  2020-04-19 18:58:25.640 120                      
E0E4EA82-DF24-44A9-93BB-89BB43B8FEB6 Datsun               2020-04-19 18:58:25.640 90                              
DD29F38F-01E2-459C-A14E-DE8D456B21FE Mercedes             2020-04-19 18:58:25.640 155                     
32BF996D-6077-4AAE-B23F-E5BB3A99365F Ferrari              2020-04-19 18:58:25.627 170             
B5E854C9-382B-4EB8-8E63-FB76F45B6EB5 Mini                 2020-04-19 18:58:25.627 110          
 
Completion time: 2020-04-19T18:58:25.6962254+01:00
```

As you can see, we managed to successfully read 10 records in the Cars table **four times** before reading only **9 records**.

---

Our second run of Session 2 code gives us:

```text
SuccessfulPriorRuns
-------------------
35
 
id                                   carname              lastservice             SpeedMPH    Details
------------------------------------ -------------------- ----------------------- ----------- -------
0B537CEF-6C8A-49A4-9295-054D29FEADC2 Porsche              2020-04-19 19:01:01.947 150
DFC2990C-E410-45AF-A225-34405AF83E17 Ford                 2020-04-19 19:01:01.993 125
9AC2AF9B-25C1-44BD-B503-38BE8D1C656A Audi                 2020-04-19 19:01:01.993 138
65D57774-BDE8-4072-BA3D-59A093AC4103 Honda                2020-04-19 19:01:01.993 87
3BF7C449-7FCA-4A93-B2CD-6688141410D2 Lamborghini          2020-04-19 19:01:01.947 175
1D5D62B3-11C7-4EC5-984F-67FB87296021 Mini                 2020-04-19 19:01:01.947 110
68A32AE3-F2C7-41DA-BFAD-8FA0276F543A Datsun               2020-04-19 19:01:01.993 90
 
Completion time: 2020-04-19T19:01:02.1127856+01:00
```

This time we managed to successfully read 10 records in the Cars table **thirty-five times** before reading only **7 records**.

---

Our final run of Session 2 code gives us:

```text
SuccessfulPriorRuns
-------------------
0
 
id                                   carname              lastservice             SpeedMPH    Details
------------------------------------ -------------------- ----------------------- ----------- -------
85227E5F-4C0F-4863-9895-21BA0232CFF5 Porsche              2020-04-19 19:05:43.380 150
F9AE7CE7-F092-4F09-9A69-25423CD0D9F4 Mercedes             2020-04-19 19:05:43.410 155
8E0F5C2D-3744-422A-A2B0-27D50BC89A96 Datsun               2020-04-19 19:05:43.410 90
D48E0EBD-9171-4278-A76B-32E2BCD64260 BMW                  2020-04-19 19:05:43.410 120
 
Completion time: 2020-04-19T19:05:43.5190517+01:00
```

The results this time were even worse. Our code **failed on the first read** of the Cars table reading only **4 records**

# Isolation Levels and In-Memory OLTP
If we run session 2 under all Pessimistic Isolation Levels (using on-disk), we get the following results:

|Isolation Level|Outcome|Description|
|-|-|-|
|READ UNCOMMITTED|FAIL|result set size varies widely|
|READ COMMITTED|FAIL|this was the default used in the examples|
|REPEATABLE READ|PARTIAL FAIL|result set size varies and the occasional deadlock|
|SERIALIZABLE|SUCCESS|either a successful run OR deadlock|

If we now run session 2 under the Optimistic Isolation Levels (using on-disk), we get the following results:

|Isolation Level|Outcome|Description|
|-|-|-|
|SNAPSHOT|SUCCESS|all runs successfull|
|READ COMMITTED SNAPSHOT|SUCCESS|all runs successfull|

If we implement the Cars table as an In-Memory OLTP table and run session 2 we get the following results:

Isolation Level|Outcome|Description
|-|-|-|
|*|SUCCESS|all runs successfull|

---

# Summary
As with nearly all cases of inconsistencies and strange transactional behaviors, they are usually displayed in the lower transactional isolation levels and this situation is no different. As is also demonstrated in our testing, we can usually fix these bad behaviors by using more restrictive levels. As you can see in our results, the problem still exists in `REPEATABLE READ` -although it appears to partially reduce the problem through the occasional deadlock (deadlocks exist for consistency reasons – so you must obviously handle them). `SERIALIZABLE`, which is the most restrictive isolation level of all, fully resolves the issue by deadlocking every single time that a successful run isn’t made. This clearly will be terminal for performance if your application handles and replays transactions on deadlock.

`SNAPSHOT` isolation comes (as expected) to the rescue and we do not see any consistency bad behaviors for any read, but quite surprisingly so does the optimistic implementation of `READ COMMITTED` (known to you and me as `READ COMMITTED SNAPSHOT`). I wasn’t really expecting that to work.

Finally, when we look towards In-Memory OLTP (IMOLTP), given that under the covers IMOLTP uses an entirely different (and optimistic) concurrency model and is completely lock and latch free, it is perhaps no surprise that no issues were experienced there. These tests add yet more weight (if you even needed it) to a move towards (on-disk) optimistic or In-Memory OLTP.

When this issue was first reported by Erland, Paul Randal ([b](https://www.sqlskills.com/blogs/paul/)|[t](https://x.com/PaulRandal)) of SQLSkills confirmed that was a known behavior and offered up his post titled [Read committed doesn’t guarantee much…](https://www.sqlskills.com/blogs/paul/read-committed-doesnt-guarantee-much/). While a quick read of Paul’s post will on the face of it look like the same issue as the one I describe, the one thing that does not align is the use of a HEAP table -since we have added a clustered index our example.

That aside, I believe the biggest trigger for this specific problem is very similar and that the following are pivotal to things going wrong:
- The use of a table Cluster GUID
- The use of mixed extents - and how SQL Server accesses them

# Final Thought
I have been saying for a very long time now that no-one should be using pessimistic isolation on their SQL Servers (despite it still being the default) with the only caveat being a 3rd party support contract requirement (and if the 3rd party doesn’t support it, you should look to move to other software). Furthermore, in SQL Server 2016 and onwards, mixed extents are disabled by default so I would expect this issue not to raise its ugly head – however, that is the only scenario I haven’t tested so I cannot confirm this.

This issue WILL NOT occur with optimistic isolation nor with In-Memory OLTP -so perhaps it is time for you to change!