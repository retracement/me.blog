+++
date = '2017-04-03T16:09:00+01:00'
draft = false
title = 'Resolving Windows Cluster creation problems in Powershell'
categories = ['Technology']
tags = ['PowerShell','HADR','Clustering','Windows','Installation']
+++

Creating a Windows Cluster is surprisingly easy, but every now and again you run into small problems which irritate and are not really helped by the error messages – a problem which is exacerbated by the do all in one go power of PowerShell.

One such problem is when you use the New-Cluster command to add all your nodes in one go.

```powershell
New-Cluster -Name magrathea -node server5,server6,server7
-staticaddress 192.168.1.70
```

Simple right? Well no. In this instance I ran into the following error:

>New-Cluster : There was an error adding node 'server7' to the cluster
>
>the node cannot be contacted. Ensure that the node is powered on and is connected to the network.
> 
>At line:1 char:1
> 
>\+ New-Cluster -Name magrathea -node server5,server6,server7 -staticaddress ...
> 
>\+ ˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜˜
> 
>\+ CategoryInfo : ConnectionError: (🙂 [New-Cluster], ClusterCmdletException
> 
>\+ FullyQualifiedErrorId : ClusterNodeNotReachable,Microsoft.FailoverClusters.PowerShell.NewClusterCommand

![Cluster create error](/images/2017/create-cluster.webp)

Ok, so the problem is obvious, isn't it? You have misconfigured the node IP address, Firewall is blocking, or even Remote Administration is disabled. Let's create the Cluster with only server5 and server6 and then try adding server7. 

We then get this error:

> Add-ClusterNode : Failed to access remote registry on 'server7.hitchhikers.galaxy'.  Ensure that the remote registry service is running, and have remote administration enabled.
>The network path was not found.
>At line:1 char:1
>\+ Add-ClusterNode -Name server7 -Cluster magrathea
>\+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>\+ CategoryInfo          : NotSpecified: (🙂 [Add-ClusterNode], ClusterCmdletException
>\+ FullyQualifiedErrorId : Add-ClusterNode,Microsoft.FailoverClusters.PowerShell.AddClusterNodeCommand

The error message is even more damming this time, and it is fairly clear you must have forgotten to enable remote administration -except you haven't. In fact, nothing seems wrongly configured with the node and everything looks identical to server5 and server6.

This time you decide to try adding the node directly from the problem server itself:

>Add-ClusterNode : The term 'Add-ClusterNode' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
>At line:1 char:1
>\+ Add-ClusterNode -Name server7 -Cluster magrathea
>\+ ~~~~~~~~~~~~~~~
>\    + CategoryInfo          : ObjectNotFound: (Add-ClusterNode:String) [], CommandNotFoundException
>\    + FullyQualifiedErrorId : CommandNotFoundException

Finally, all becomes clear! This time the error message fails, stating that the `Add-ClusterNode` cmdlet is not available and it is obvious that you have forgotten to install the *Failover-Clustering* feature on this node (doh!).

From the node in question we run the following in Powershell:

```powershell
Get-WindowsFeature Failover-Clustering|Add-WindowsFeature
```

Once installed, it is time to try again...

```powershell
Add-ClusterNode -Name server7 -Cluster magrathea
```

Let's query the cluster node:

![Get-Clusternode](/images/2016/get-clusternode.png)

**Success!**

---

# Conclusion

The moral of this story is that error messages are misleading, so when dealing with something as (potentially) complex as a Windows Cluster, it is always useful to break operations down into small fragments in order to understand the real reason for failure.