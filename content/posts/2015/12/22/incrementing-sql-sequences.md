+++
date = '2015-12-22T12:40:00+01:00'
draft = false
title = 'Incrementing SQL Sequences'
categories = ['Technology']
tags = ['Sequences','SQL']
+++

[Sequences](https://msdn.microsoft.com/en-us/library/ff878091(v=sql.110).aspx) were first introduced in SQL Server 2012 and are a way to improve performance in situations that would have traditionally been implemented using the column based identity property. One of the biggest downfalls of the identity property is that new values are generated on inserts meaning that they are also transactionally dependent. Sequences on the other hand are not and allow for caching and many other benefits which you can compare and contrast in this really good SQL Server Pro magazine article by Itzik titled [SQL Server 2012 T-SQL at a Glance – Sequences](https://sqlmag.com/blog/sql-server-2012-t-sql-glance-sequences).

# Our situation

Recently our performance environment was undergoing load testing using pre-loaded synthetic data which (upon execution) started to result in failures from identity conflicts. It was fairly obvious that our current sequence seed values were much lower than the loaded data identities. Not so much of a problem you might think, since we can easily reseed the sequence number via the *Sequence Properties* dialog (below). Simply select the *Restart sequence checkbox* and type your new seed number into the entry box next to it and click *OK*.

![sequence properties](/images/2015/sequence-properties.jpg)

The only problem with this approach is that our database was configured (rightly or wrongly) with approximately 250 sequences! Since we could not be sure which sequences would ultimately cause us problems we decided to increment each one by 10,000.

Not being someone who likes performing monotonous tasks and also recognising the fact that this task would probably need to be performed again in the future I decided to attempt to programatically solve this problem.

---

# How to use it

_**Disclaimer:** Before using the following script, please make sure you understand what you are doing and where you are doing it. The responsibility for its use and misuse is all yours!_
 
The script below is fairly basic and generates a script to update every single sequence within your database (make sure you change context to the correct one) with a default increment of 10000 (feel free to alter as necessary). If you only want to update a range of sequences then obviously you should add a `WHERE` clause to this script and filter on whatever criteria floats your boat.

```sql
DECLARE @increment_sequence INT = 10000
SELECT
   'ALTER SEQUENCE [' + SCHEMA_NAME(seq.schema_id) 
   + '].[' + seq.name + ']'
   + ' RESTART WITH '+ CAST(CAST(seq.start_value AS INT) 
   + @increment_sequence as VARCHAR(max)) + ';'
FROM
   sys.sequences AS seq
   LEFT OUTER JOIN sys.database_principals AS sseq
ON
   sseq.principal_id = ISNULL(seq.principal_id, 
   (OBJECTPROPERTY(seq.object_id, 'OwnerId')))
ORDER BY
   SCHEMA_NAME(seq.schema_id) ASC,
   seq.[Name] ASC
```

It creates a script in your query results as below. Simply copy and paste this into a new query window and execute.

```sql
ALTER SEQUENCE [DB_Sequence].[LoanID] RESTART WITH 10033;
ALTER SEQUENCE [DB_Sequence].[TransferID] RESTART WITH 10000;
ALTER SEQUENCE [DB_Sequence].[AccountID] RESTART WITH 68719;
ALTER SEQUENCE [DB_Sequence].[CustomerID] RESTART WITH 1010006;
```

If you do need to update many sequences in your database I hope you find this script useful and it saves you as much time as much as it has me!