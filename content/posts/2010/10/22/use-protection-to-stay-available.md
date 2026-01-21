+++
date = '2010-10-21T12:17:00+01:00'
draft = false
title = 'Protect your failover cluster names'
+++

One of the biggest worries for us grumpy DBAs are the risks which are posed to us due to our reliance on other teams, specialists and technologies in order to maintain a working and efficient SQL Server infrastructure. This was one of the key concepts when I put together my Thinking outside the Box talk for [SQLBits](https://sqlbits.com) 7, and insist that we need to understand, get involved and help communicate best practices to our colleagues and fellow IT professionals.

An often overlooked area of our SQL Server infrastructure is the lack of protection afforded to the machine and service accounts that our instances and Clusters are using, and the dangers posed to these environments because of this is enormous.


Particularly in the case of SQL Clusters, should a well meaning Windows Administrator accidentally reset or remove the Cluster Group network names, you have a potentially very big problem to deal with. On deletion of the account you still have the possibility of recovering this account from tombstone using a third party tool (more on this another time) but if the account is removed, the first you will know of it is when your SQL Failover Cluster fails and refuses to start on any other nodes in your cluster.

New to Windows 2008 is the ability to protect your accounts and machine names from accidental deletion and it is very simple to do. Firstly as the DBA you should list all the objects that you wish to protect from your Cluster names, SQL Failover Cluster names (aka virtual servers), Windows network names (you should do this as well for all of those servers running standalone SQL servers) and all Service Accounts used within your Clusters and SQL instances. In short, for any object you *DONT WANT DELETED!*

Windows 2003 users can refer to this excellent article [Protect Objects in Windows Server 2003 Active Directory from Accidental Deletion](https://www.petri.co.il/protect-windows-ad-obects-accidental-deletion-recovery.htm) instead.

Once you have compiled your list all you need to do is get your friendly Windows Admin to load up the Active Directory Users and Computers snap in and for each object look at it's properties.

![Name object properties](/images/2010/protect_account1.png)

On the Object tab, ensure that the *Protect object from accidental deletion* checkbox is ticked.

![Protect object from accidental deletion](/images/2010/protect_account2.png)

A little bit of forward planning in this area could save you and your fellow professionals a world of pain. Be sure that you think ahead, *use protection and maintain your availability*.