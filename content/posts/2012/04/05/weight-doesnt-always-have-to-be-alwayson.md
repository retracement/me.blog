+++
date = '2012-04-05T15:56:00+01:00'
draft = false
title = 'Weight doesnt ALWAYS have to be AlwaysOn'
categories = ['Technology']
tags = ['Availability Groups','Clustering','HADR','SQL']
+++

One thing I keep hearing myself mentioning more and more in conversation (and most recently in a discussion group at [SQLBits](https://sqlbits.com/) a few days ago) is the ability to configure your Windows Cluster Quorum for situations where each cluster node may not be of equal importance.

In [SQL Server 2012](https://www.microsoft.com/sqlserver/en/us/default.aspx) we have various high availability enhancements and improvements and most of these are branded under the term [AlwaysOn](https://www.microsoft.com/sqlserver/en/us/solutions-technologies/mission-critical-operations/SQL-Server-2012-high-availability.aspx). In particular we have enhancements to AlwaysOn SQL Failover Clustering and a new technology known as AlwaysOn Availability Groups. Whilst I won’t bore you with any specific details about what these are or how to use them -since that is no doubt something for another day, you may probably be already aware that for both, each require the existence of a Windows Cluster in order to use them.

One of the biggest reasons why Windows Clustering has been adopted as a pre-requisite for AlwaysOn Availability Groups is to use the mechanism known as the *Quorum* that provides a voting mechanism to determine which nodes can collectively be considered to be still *"alive"* and which nodes can be singularly or collectively be considered to have *"failed"*. This mechanism is a very important concept to AlwaysOn since it prevents the classic *split brain* problem. With respect to Availability Groups it also means that the Database Witness server that is used for Database Mirroring is not needed to implement this technology and because of this, is more scalable and reliable than it otherwise would be. <sub>*1</sub>

![Not so cute clones](/images/2012/clones21.jpg)<br/>
*Quorum is not so CUTE!*

As you may also be aware, in Windows 2008/R2 there are four types of [Quorum model](https://technet.microsoft.com/en-us/library/cc731739.aspx) that you may use and these are:

- **Node Majority** -where more than half the nodes must remain active.
- **Node and Disk Majority** -where the disk acts as an extra vote and that more than half the votes must be available.
- **Node and File Share Majority** -where the file share acts as an extra vote and that more than half the votes must be available.
- **No Majority: Disk Only** -the disk quorum must remain available.

In all cases, should the required situations not be true, each Windows Cluster Node’s Cluster Service will automatically shutdown since *Quorum* is lost.

In situations where the total number of Cluster Nodes is an odd number then you would traditionally use the Node Majority Quorum model so that you could lose a total of half of your cluster nodes minus one before having a cluster failure. Otherwise the other three Quorum models should be considered (generally Disk Only Quorum could be a valid option when the node count is one or two but should otherwise only be considered in special cases).

What is not commonly known is that a Cluster Node does not *HAVE* to have a quorum vote. By default they do, but it is possible to set what is known as *Node Weight* to zero. Before we come onto that though, you are probably wondering exactly why you would want to do this? Well there are several scenarios that make this a desirable thing to do such as in situations where you have implemented a Geo-Cluster (a Cluster across Geographic locations).

Consider the following diagram :-

![Windows Cluster](/images/2012/cluster600.png)<br/>
*AlwaysOn FCI or AG nodes across sites*

As you can see we have Site A and Site B each hosting a selection of Cluster Nodes. It is possible that each node might be hosting a standalone instance of SQL Server and is simply part of a Windows Cluster because we are using Availability Groups (so each can house a Replica) OR it might be that we have implemented SQL Failover Clustered Instances perhaps because that we require the ability to fail SQL instances across sites.

Now in the scenario we propose (and assuming we choose a Quorum model of Node Majority) the total number of Quorum votes equals five. Imagine then that we suddenly have a failure of Node C. Since the number of votes would then equal four we would still have *quorum*. Now consider the prospect that we lose connectivity to Site B. This event would not only cause the Cluster Service for nodes on Site B to shut down (since their total votes of two would be less than the required majority of three) but would also cause the Cluster Service for nodes on Site A to shut down since there would also only be two votes available.

Although the Cluster Nodes on Site A could be forced to start, we have unfortunately lost availability (however temporarily) and more importantly requires manual intervention. Perhaps a slightly more elegant solution would be to set the Node Weight of  Node D and E to zero meaning that nodes A,B and C are the only ones that can cast a vote to make quorum (making a total Quorum count of three). In the event of a loss of one node on Site A and a loss of connectivity to Site B, two voting nodes will still be casting a quorum majority thereby keeping the Cluster available.

So lets now move onto how you are able to set your Node Weight for your Cluster nodes. Unsurprisingly we can set the Cluster node Node Weight through a property called *NodeWeight* but this is not accessible by default. This can be demonstrated by using the following PowerShell script (first we must import the failover clustering module):

```powershell
Import-Module failoverclusters
Get-Clusternode|Format-Table -auto -property Name, State, NodeWeight
```

We get the following result:-

```text
Name         State NodeWeight
----         ----- ----------
wonko           Up
wowbagger       Up
```

However upon installing [Hotfix 2494036](https://support.microsoft.com/kb/2494036) (which I should add requires a reboot to take effect) this makes the *NodeWeight* property accessible:

```text
Name         State NodeWeight
----         ----- ----------
wonko           Up          1
wowbagger       Up          0
```

As you can see from the above, I have already set the NodeWeight of wowbagger to zero and I did so by running the following PowerShell command:-

```powershell
(Get-ClusterNode "wowbagger").NodeWeight = 0
```

Before you get all Jackie Chan on me and set some of your Cluster Nodes node weights to zero you should first seriously sit down and draw up a design strategy as to whether this makes sense to do so. In the scenario I proposed “the Business” had stipulated that a loss of Site B or any of the Cluster Nodes within it should not in any way effect the availability of the primary Site A but by doing so we reduce the total number of possible failures on site A to a maximum of one failure in a Cluster containing five nodes! Therefore be very careful and cautious and only when necessary remember that your Cluster Node weight doesn’t always have to be AlwaysOn.

---

_<sub>*1</sub> This is a contentious issue for some (including me) since unlike with Database Mirroring, AlwaysOn Availability Groups (since it uses Windows Clustering) requires that a single Active Directory Domain spans each Geographic location._