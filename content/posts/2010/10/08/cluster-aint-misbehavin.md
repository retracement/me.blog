+++
date = '2010-10-08T11:04:00+01:00'
draft = false
title = 'Recovering from corruption in SQL Failover Cluster'
+++

One of the great things about SQL Clustered Instances is that they generally give a much greater possibility for High Availability which is obviously the whole point of them. However one of the more commonly overlooked considerations is the fact that by their very design you are introducing extra complexity to your environment and this can be counter-productive and work against what you are trying to achieve in the first place. For instance if you allow unqualified and inexperienced people to access and even administrate your SQL Clusters then you could be in for a world of pain. I have seen it time and time again and yet those kind of classic mistakes seem to keep being allowed to happen. In order to highlight this point further, imagine your sysadmins, operators and DBA's going about their everyday management of systems oblivious to the fact that what they are working on is actually a clustered node. They could be creating a Network Share, stopping a Service or even installing software without realizing what they are doing and the potential risk they are taking. It is for this very reason that I believe that Windows Clusters in particular require a specialized team in order to support them because we want to maintain the High Availability don't we folks?

![Oh no!](/images/2010/ohno.jpg)

Another area of consideration is the issue of problem resolution. Your management team will think that just because you now have a shiny new Cluster you will never have any more problems or outages on these systems but this is obviously wrong and can add to the stress levels and expectations when trying to get a system back online. Also you should consider that resolving an issue can get a whole lot more difficult since your resolution must take into account what you are resolving. Making a change on a Clustered node can have a knock on effect to the whole of your Cluster configuration and secondly what you would have changed in a standalone environment may not even be possible in a Clustered Environment. So remember you are on a harder learning curve and *NEED* to remember this when disaster strikes because it *WILL* take you longer to fix and the worst thing you can do is panic.

A case in point happened to me the other day with a SQL Cluster that has effectively been open to every man and his dog (don't ask -it was political decision from long ago). I was experiencing very unusual behavior from the SQL instance whereby the Clustered Instance actually appeared to be coming online but when you attempted to connected to it via SSMS and expand out the Databases node you were presented with the hourglass forever. Next step was to connect via `SQLCMD` and various queries could be ran, even a list of the databases went through OK which did indicate that various user databases were "Recovering.." but if I tried to filter this list or even sort it then the prompt would just hang. I couldn't even perform an `sp_who2`! Error logs did not really highlight anything obvious.

![Problems](/images/2010/problems.jpg)

My biggest problem with all this was that I really wanted to know whether it was a Clustering problem or an Instance problem and it was hard to decide since I didn't really have much to go on. I decided that it must be the latter though based solely on the fact that the behavior being displayed appeared to be DB engine related and secondly would carry over to the next node on failover. Usually when the problem is Cluster based, the resource in question will not start. I personally believed that something was wrong with my System databases but I had no warnings and they were all coming online with no problems. 

# Testing for corruption

Trying to run a CHECKDB against the master database just appeared to be doing nothing -exactly the same behavior as some of the other commands so this didn't really tell me anything, I let it run for a while nether-the-less. Once convinced that this was a waste of time I decided that enough was enough at to start using "Gorilla Tactics".  First I decided to check the integrity of the disk that housed the system databases and was surprised to find there was a problem. DBA's please take note…I have very rarely ever seen another DBA check file system integrity (and many sysadmins too come to think of it) but it is an incredibly useful thing to check *yourself* and normally always fixes odd behavior with various apps should you find any. For instance a common one in SQL Server is where backups to disks are failing for no apparent reason. So once I detected that there was a problem in that area, I took the Cluster Group's SQL Services offline and also checked the remaining disks but they were OK. Next, once all file handles were closed to the drive with issues I then ran a fix.

![Filesystem corruption](/images/2010/filesystem_corruption.png)

I fully expected that to solve the problem having seen file system corruption be the cause of many a problem in the past, but after bringing the Cluster Group's SQL Services back online the same strange behavior remained! 

# Reparing master DB
No more Mr. Nice Guy now I thought and decided that something really must be wrong with the master database. I Decided that a restore of master database was needed but the issue was how to do this on a Clustered Instance? For example there was no obvious place to enter any startup parameters in the SQL Server resource to start it in minimal configuration mode -although I could have sworn I'd done this before on previous versions of Clustering (this one was Windows 2008). Attempting to directly change the Service parameters did not work either, they were not remembered. So what to do now?

I decided that it would be easier to start the instance through the command line but wondered whether this would work since we have to specify the SQL instance name (ours was the SQL Failover Cluster name) and in reality we were starting the local SQL Server Service on this local node. To my surprise, using the Cluster name and instance worked perfectly, the instance came online in minimal mode (and therefore also in single user mode). I ensured that I quickly connected to grab that one and only connection and I was now ready for the restore.

![Startup minimal mode](/images/2010/startup-minimal1.png)

So the restore was probably the easiest part of my whole shenanigans and simply typed out the command to do this, and once completed the instance automatically shut itself down.

A very convenient side effect of running from the command line using different startup parameters meant that I hadn't actually made any other configuration changes elsewhere, meaning that all I needed to do now was to bring the Cluster Group's SQL Services back online.

![Restore master DB](/images/2010/restoremaster.png)

# So what happened you might ask?

Yes everything came back online fine, I connected to the instance very quickly via SSMS and all problems had disappeared. The next step was to ensure I took a fresh backup of all the system databases and once complete I endeavoured to ensure the integrity of each and every system and user database. Thankfully all of these checked out fine, so the last step was to make sure a `FULL` backup of each and every database was taken and put to one side.

# In conclusion 

When you are performing database administration to a Clustered Instance, how you do it and what you do depends upon what the Cluster Configuration allows. SQL Clusters are much harder to troubleshoot than standalone instances and you should always keep reminding yourself that just because you know how to fix a problem on one does not necessarily mean that you will be able to do the same thing on the other.