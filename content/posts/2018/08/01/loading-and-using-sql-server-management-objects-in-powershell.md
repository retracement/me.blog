+++
date = '2018-08-01T17:37:00+01:00'
draft = false
title = 'Loading and using SQL Server Management Objects in PowerShell'
categories = ['Technology']
tags = ['PowerShell','SMO','SQL','Dotnet']
+++

I recently came across a snippet of code I had written for a client several years ago – one of those "by the way, how would you do this" type of requests. On the face of it, the task at hand seemed fairly simple to deliver since they wanted to query SQL Server Agent jobs through PowerShell. But because they also wanted to be able to change the SQL Server Agent jobs using PowerShell, I decided that the simplest way would be to programmatically access these jobs as Objects thereby providing *Methods* to call and *Properties* that can be easily returned or changed through their *get* and *set* accessors.

It was a no brainer that I would need to use [SQL Server Management Objects](https://docs.microsoft.com/en-us/sql/relational-databases/server-management-objects-smo/sql-server-management-objects-smo-programming-guide) (SMO) through PowerShell. In this post I will explain how to use SQL Server Management Objects in PowerShell.

# Loading SQL Server Management Objects

First, we need to talk about how to load the SMO assembly into the PowerShell session...

**Method 0, Using sqlps**

If we are going to use `sqlps` then we can go straight to the code since this automatically loads the SMO assembly into the sqlps session. You can do this by either importing the sqlps module into your PowerShell session:

```powershell
Import-Module sqlps
```

Or simply enter a sqlps session from your PowerShell session by running that command from the command line:

```bash
sqlps
```

Otherwise, we need do this using one of the following methods.

**Method 1, Using Reflection**
This method is generally considered the older way of doing this, though I personally think it is the simplest and probably less prone to issues.

```powershell
[reflection.assembly]::LoadWithPartialName("Microsoft.SqlServer.Smo")
```

However, you might want to visit the Microsoft Docs page for [Assembly.LoadWithPartialName Method](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.assembly.loadwithpartialname) and take note of the warning pane stating that:

>This API is now obsolete

This is referring explicitly to both method overloads of `LoadWithPartialName(String)` and `LoadWithPartialName(String, Evidence)`. So I suppose it is fair to say to avoid using this technique in the future.

**Method 2, Using Add-Type**

If you are using PowerShell 3.0 or above (which at the time of writing is probably nearly everyone since it was introduced in Windows Server 2012) we can use the `Add-Type` cmdlet.

```powershell
Add-Type -AssemblyName `
"Microsoft.SqlServer.Smo,Version=14.0.0.0,Culture=neutral,PublicKeyToken=89845dcd8080cc91"
```

You might be wondering which Smo version you should be using? You can find out what is installed by looking in the GAC at `C:\Windows\assembly\GAC_MSIL\Microsoft.SqlServer.Smo`

![SMO GAC](/images/2018/smo_gac.jpg)

In my case, I have a single instance of SQL Server 2016 installed but have four versions of SMO available. Whilst all four will technically work, it is possible functionality/ performance may be limited between versions, so we use the most recent (14.0.0.0).

As a final point, you can query your loaded assemblies for your session using the following command:

```powershell
[System.AppDomain]::CurrentDomain.GetAssemblies()
```

It is interesting (if nothing else) to try this after attempting each method listed above to see how each app domain differs.

Ok. So once we have SMO available we can start consuming it. First up, we will instantiate an object to our SQL Server instance.

```powershell
$sqlinstance = New-Object `
   -TypeName Microsoft.SQLServer.Management.Smo.Server("server2\sql2016")
```

Please note that you will not see an error or exception if the instance is not found. Instead, you could access one of the main properties to test existence for a value. For example, if we query the Product property, we should see "Microsoft SQL Server", otherwise that field will be blank.

Finally, to see what methods and properties are available we can view the public object class members as follows:

```powershell
$sqlinstance|Get-Member
```

Thankfully we can also see which properties have been defined with a read accessor (get) or write accessor (set) or both (get;set).

![Get/Set accessors](/images/2018/psgmaccessors.jpg)

---

In my next post titled [Manipulating SQL Agent Jobs through PowerShell](/posts/2018/08/02/manipulating-sql-agent-jobs-through-powershell) I will explain how to manipulate SQL Server Agent jobs using PowerShell and SQL Server Management Objects.