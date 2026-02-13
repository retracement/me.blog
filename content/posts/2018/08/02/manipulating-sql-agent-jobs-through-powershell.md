+++
date = '2018-08-02T12:14:00+01:00'
draft = false
title = 'Manipulating SQL Agent Jobs through PowerShell'
categories = ['Technology']
tags = ['PowerShell','SMO','SQL','Dotnet']
+++

In our last post titled [Loading and using SQL Server Management Objects in PowerShell](/posts/2018/08/01/loading-and-using-sql-server-management-objects-in-powershell), we demonstrated how to load and consume SQL Server Management Objects in PowerShell, which then allows us to access your SQL instance through an SMO instance object. Please make sure you read that post before continuing.

In this specific post, we will demonstrate and walk through how to use PSH and SMO to view and change SQL Agent jobs.

If you recall we had previously instantiated an object to our SQL Server instance and assigned it to our $sqlinstance variable as follows:

```powershell
$sqlinstance = New-Object `
   -TypeName Microsoft.SQLServer.Management.Smo.Server("server2\sql2016")
```

We can access the SQL Server Agent through its Jobserver property (returning an object of type [Microsoft.SqlServer.Management.Smo.Agent.JobServer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.sqlserver.management.smo.agent.jobserver)). This object has lots of available properties and methods available, but the one we are interested in is clearly the Jobs property (providing a collection of type [Microsoft.SqlServer.Management.Smo.Agent.JobCollection](https://docs.microsoft.com/en-us/dotnet/api/microsoft.sqlserver.management.smo.agent.jobcollection)). Obviously, this means we have to iterate over each one or access a specific element through the collections indexer.

For now, let's take a look at all the jobs in the Jobs collection:

```powershell
$sqlinstance.jobserver.jobs|Select-Object -Property Name, Description
```

This results in the following...

```text
Name                    Description
----                    -----------
My Maintenance Job      No description available.
syspolicy_purge_history No description available.
```

Through the indexer we can access our job in question using its collection numerator:

```powershell
$sqljob=$sqlinstance.jobserver.jobs[0]
```

Or via its named property:

```powershell
$sqljob=$sqlinstance.jobserver.jobs['My Maintenance Job']
```

Either way, we now have an object which we can retrieve or change properties on.

A word of caution, it is possible for the underlying SQL instance settings to change between the time that your SMO instance object was created and now, so if you intend on changing something it is advisable to make sure you are looking at the most recent change. I fully expected that calling the Refresh method on the Jobs collection would be enough to rebuild the collection with updated properties, but this does not appear to be the case. In fact, doing this on the parent JobServer object or even the sqlinstance object does not do that either.

The only guaranteed way to ensure your SQL job information is accurate appears to be calling the Refresh method on a specific job itself as follows:

```powershell
$sqljob.Refresh()
```

Querying our specific job now shows that we now have a description value (albeit misspelled!).

```powershell
$sqljob|Select-Object -Property Name, Description
```

Gives us:

```text
Name               Description
----               -----------
My Maintenance Job My descriptionz2
```

Let's fix that problem and change the description. Obviously, this should be a simple operation by setting the SQL job Description property:

```powershell
$sqljob.Description = 'My description'
```

Unfortunately, if we take a look at the SQL job through SSMS (feel free to right-click and refresh the Agent node first), we see that our change has not taken effect.

![SQL Job](/images/2018/myjob.jpg)

In order to write back your change/s to the SQL Server instance you must call the objects Alter method as follows:

```powershell
$sqljob.Alter()
```

And this is enough to update the SQL Job on the SQL instance!

This concludes what I hope was a fairly easy two-post demonstration of how to use SQL Server Management Objects and PowerShell and use them to programmatically manipulate your SQL Server instances. In this specific case we ultimately used SMO through PSH to query and manipulate a simple property on a SQL Job (the job description), however I hope you can visualize the potential for doing much more advanced operations using this method to the SQL job server and the entire SQL instance!