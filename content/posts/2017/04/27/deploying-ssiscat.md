+++
date = '2017-04-27T11:10:00+01:00'
draft = false
title = 'Bitesize SSIS - Deploying the SSIS Catalog'
categories = ['Technology']
tags = ['SQL','SSIS']
+++

I hate SSIS. It seems to me that it is full of certain nuances and unless you are regularly developing SSIS packages, they are easy to forget or it is easy to miss specific important steps. I first started using SSIS back in 2005 when it was directly introduced to replace DTS, but even today I am constantly going around in circles whenever I have to return to write certain functionality.Therefore I have decided to put together a “Bitesize” series of posts that encapsulate simple operations in order to help not just you, but more importantly, remind me! Hopefully, this will save me time in the long run...

---

The SSIS Catalog (SSISDB) was first introduced in SQL Server 2012 and enabled faster and easier SSIS package deployment (particularly in SQL Server failover clusters) since (by its use) all packages and metadata would now live inside its database, and any other remaining metadata within the instance. This is, of course, a huge improvement over traditional SSIS Package store deployment (and covered in future posts), but for now, we will focus solely on simple standalone deployment.

# Create the SSIS Catalog on your SQL Instance

Creating the SSIS Catalog for your instance is fairly straightforward, simply navigate to (and right-click) the *Integration Services Catalog* node in your instance via SSMS and select *Create Catalog...*

![Create Catalog](/images/2017/create-catalog3.jpg)

The Create Catalog wizard is a single page and all you really need to provide is a password which will be used to protect the encryption key which secures your SSIS Catalog secrets.

![Create Catalog Wizard](/images/2017/create-catalog21.jpg)

Note also in the wizard above, the panel message stating *"You can manage the encryption key by creating a backup. If you migrate or move the Integration Services catalog to another SQL Server instance, you can restore the key to regain access to encrypted content"*. The relevance of this operation will become obvious in later posts when we discuss deployment of the SSIS Catalog to an Availability Group.

You might also notice the option *Enable automatic execution of Integration Services stored procedure at SQL Server startup*. All this does is to automatically execute (or not) the SSIS cleanup job whenever the instance is started. You can view the automatic startup state of stored procedures by running the following code segment:

```sql
SELECT name,is_auto_executed 
FROM sys.procedures
```

If you want to change the automatic startup state of stored procedures then you can use the [`sp_procoption`](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-procoption-transact-sql) stored procedure.

Once your SSIS Catalog is created you should see the Catalog within the *Integration Services Catalog* node, a database called *SSISDB* under the *Databases* node, and a job called *SSIS Server Maintenance Job* under the SQL Server Agent *Jobs* node.

---

As you have seen, configuring your SQL Server for the SSIS Catalog is incredibly easy and regardless of your current SSIS package deployment strategy is something that you should use going forward (and also consider migrating your existing packages). In future posts, we will also cover SSIS Catalog deployment in a highly available configuration via an Availability Group.

---

Want more Bitesize SSIS tips? Then keep an eye open for the other posts in the series!