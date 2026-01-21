+++
date = '2010-10-11T11:58:00+01:00'
draft = false
title = 'How to generate failover cluster logs'
+++

Windows and SQL Clustering can be a tricky business, but fortunately most of the administration and troubleshooting can be performed through the GUI. Obviously the same argument can apply that the simplicity of any GUI can be it's downfall in the sense that it opens up something complicated to the inexperienced. This can be a recipe for disaster on Clusters but as mentioned in previous posts is really down to your IT structures to restrict access to them.

Beginner to moderate level Clustering administrators may not be aware of this, but there is much you can (and sometimes must) perform from the command line. In day to day running activities there is usually no need to do so but knowing that various possibilities are available to you from the command line can be an absolute life saver. I also want to add that there have been numerous posts recently from various professional's blogs about managing Clusters from Powershell, but this is not what I am discussing. The commands I refer to are the native executables that are installed through the setup of Clustering.

I recently came across a great post about one such command, thanks to being pointed in the direction of Joe Sack from a tweet by Gavin Payne ([blog](https://blog.gavinpayneuk.com) | [twitter](https://twitter.com/GavinPayneUK)). I immediately noticed the post from Joe (seeing the word Clustering always catches my attention) and decided to take a look and encourage you to [check it out](https://blogs.msdn.com/b/joesack/archive/2010/02/09/dba-101-collecting-and-interpreting-failover-cluster-logs.aspx). He effectively discusses generating the Cluster log files for Windows 2008 Clusters so I decided to try it out myself quickly. Trying is the only sure fire way to remembering in my humble opinion!

Running the command will generate all the logs for each Clustered node in your specified location. Should any node be offline then the command will simply generate an RPC error for that node.

So the first thing you need to do is run the command to generate those logs.

![Generate cluster logs](/images/2010/generate_logs1.png)

Once all those lovely logs have been generated (it appears to be one per node) you can review them for your enjoyment.

![View log](/images/2010/logs.png)

As you can see when a problem hits, just knowing how to do this could aid you in your quest to solve the issue much faster than otherwise. Hopefully in future posts I can have a look into what other things you might want to use the native command line for when configuring your Clusters.