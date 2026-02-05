+++
date = '2012-01-30T14:29:00+01:00'
draft = false
title = 'What are the impersonation rights for my database users?'
categories = ['Technology']
tags = ['SQL','Security']
+++

From time to time I get thrown the odd request to provide various bits of information from the SQL Server environment and most recently I was asked about a group of databases that required migrating to another instance. For some reason there had been a little bit of a panic about the impersonation rights in each of the databases and their configuration. The request was _\*ahem\*_ kindly passed to me. The number of database users was huge in each database and performing a manual lookup through the SSMS GUI got rather tiresome very quickly (after about the second user).

My reply was that as long as we ensured each database was migrated with associated logins and mapped correctly (to prevent orphaned users) we didn’t really need to worry about the impersonation, since it would be contained within the databases and unaffected by the move. Unfortunately, there was no getting away from it, they still wanted a report.

I first decided to perform a web search for the term *list impersonate rights* which led my to quite a useful post by Kendal Van Dyke ([blog](https://www.kendalvandyke.com/)|[twitter](https://twitter.com/SQLDBA)) called [Hey Mr. DBA, What Permissions Do I Have On This Database?](https://www.kendalvandyke.com/2008/12/hey-mr-dba-what-permissions-do-i-have.html) but I was already familiar with the examples listed, having previously come across the [fn_my_permissions](https://msdn.microsoft.com/en-us/library/ms176097.aspx) and [fn_builtin_permissions](https://msdn.microsoft.com/en-us/library/ms186234.aspx) functions in [Books Online](https://msdn.microsoft.com/en-us/library/ms130214.aspx). They did what I wanted, but really the wrong way around. To use them I would have to write a cursor based solution which would be a little long winded.

Thankfully most SQL Server problems are relatively easy to solve by applying a very small amount of grey matter to them. I decided to turn my attention to the system catalog views and started off with [sys.database_principals](https://msdn.microsoft.com/en-us/library/ms187328.aspx). Using the [Microsoft SQL Server 2008 R2 System Views poster](https://www.microsoft.com/download/en/details.aspx?id=722) which is helpfully stuck to my office wall I focused in on this view and my attention was drawn to the [sys.database_permissions](https://msdn.microsoft.com/en-us/library/ms188367.aspx) catalog directly underneath. Surely this would be the most likely candidate for the `IMPERSONATE` grants?

![Principals](/images/2012/dbprincipals.jpg)<br/>
*Wall posters can come in handy!*

After a very quick query of several of the databases provided to me I noticed that in one of them under the column *permission_name*, several rows contained the entry *IMPERSONATE* with a *state_desc* of *GRANT*. Bingo! That was just what I wanted. Time to write the query...

So we can test the query, let us first create a new database, several logins, several users from the logins and grant impersonation rights to some of the users to some of the others:

```sql
CREATE DATABASE Impersonation;
GO
USE Impersonation;
GO
 
CREATE LOGIN login1 WITH PASSWORD = 'Password1';
CREATE LOGIN login2 WITH PASSWORD = 'Password2';
CREATE LOGIN login3 WITH PASSWORD = 'Password3';
CREATE LOGIN login4 WITH PASSWORD = 'Password4';
 
CREATE USER user1 FROM LOGIN login1;
CREATE USER user2 FROM LOGIN login2;
CREATE USER user3 FROM LOGIN login3;
CREATE USER user4 FROM LOGIN login4;
 
GRANT IMPERSONATE ON USER::user4 TO user1;
GRANT IMPERSONATE ON USER::user4 TO user2;
GRANT IMPERSONATE ON USER::user1 TO user3;
GRANT IMPERSONATE ON USER::user1 TO user2;
```

Next we should connect under login1 and change context to our Impersonation database; then attempt to impersonate *user4* to test that everything works:

![Impersonation](/images/2012/sqlcmd.jpg)<br/>
*user4 impersonation works*

```sql
USE Impersonation
GO
EXECUTE AS USER = 'user4';
SELECT USER_NAME();
```

OK so we know all this works and we know how the rights are assigned. Lets write our query:

```sql
USE Impersonation
GO
SELECT DB_NAME() AS 'database'
    ,pe.permission_name
    ,pe.state_desc
    ,pr.name AS 'grantee'
    ,pr2.name AS 'grantor'
FROM sys.database_permissions pe
    JOIN sys.database_principals pr
        ON pe.grantee_principal_id = pr.principal_Id
    JOIN sys.database_principals pr2
        ON pe.grantor_principal_id = pr2.principal_Id
WHERE pe.type = 'IM'
```

This query produces a rather simple result set as you see in the next picture:
![impersonation results](/images/2012/impersonation.jpg)

And there we have it -all impersonation rights are listed for the database in question. I was able to go onto query the five databases in question and provide a very simple report which would have taken me a very long time to perform using the SSMS GUI.

Whilst nothing I have demonstrated in this article is particularly difficult, it does highlight the multitude of different ways to perform operations in SQL Server. Just make sure you choose the easiest route!