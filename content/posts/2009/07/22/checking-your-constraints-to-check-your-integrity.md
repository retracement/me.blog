+++
date = '2009-07-22T11:33:15+01:00'
draft = false
title = 'Check your constraints to check your integrity'
+++
Something that you probably don’t need to do every day is to change the data in the foreign key field. Generally, depending upon how your application is designed these would be relatively irrelevant and are simply just a mechanism to constrain the data on the key and logically join it together. In a situation where the primary and foreign key data actually describes a physical entity, it is feasible that when the physical entity changes name it makes perfect sense that the foreign and primary key data ought to change to reflect that.

For instance assume that there is an inventory database that has two tables, one called server (which has an entry for each server host) and one called instance (one for each SQL Server instance). If the PK->FK constraint is on the server host name itself then should the physical server every undergo a rename (perhaps due to new naming standard policy) then it this key constraint no longer works.

In order to update either of the fields in each table, you first need to alter the constraint so that it doesn’t check the data being entered.

e.g.
```sql
--disable the constraint
ALTER TABLE instances
NOCHECK CONSTRAINT FK_Instances_Servers

--first update the foreign key/s
UPDATE instances SET serverid = 'prd1a' WHERE serverid = 'frodo'
--then update the primary key/s
UPDATE servers SET serverid = 'prd1a' WHERE serverid = 'frodo'

--reenable the constraint
ALTER TABLE instances
CHECK CONSTRAINT FK_Instances_Servers
```

Great so you have now changed the data, but really having disabled the constraint you really ought to check that all constraints are good:

```sql
--check constraints in all tables
DBCC CHECKCONSTRAINTS
```

Now special mention should go to the CHECKCONSTRAINTS consistency check because if you still have a disabled constraint then this will not be checked. In order to make sure you check everything irrespective of whether a constraint is active or not you should issue the following command:

```sql
DBCC CHECKCONSTRAINTS WITH ALL_CONSTRAINTS
```

It is possible to check a specified constraint or table using this command see [Books Online](http://msdn.microsoft.com/en-us/library/ms189496(SQL.90).aspx) for more information.


To avoid the necessity of ever needing to change primary and foreign key field generic keys should be used, for instance an integer or random alphanumeric key. This would ensure that the constraint key data is disassociated from the real data.