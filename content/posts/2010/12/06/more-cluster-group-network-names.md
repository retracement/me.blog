+++
date = '2010-12-06T13:39:00+01:00'
draft = false
title = 'How to rename a SQL Failover Cluster '
+++

Most recently I came across a couple of tweets on the #sqlhelp hash tag regarding changing a Cluster Group network name. Helpfully a knowledge base article was offered up to the person asking the original question. The article [How to: Rename a SQL Server Failover Cluster Instance](https://msdn.microsoft.com/en-us/library/ms178083.aspx) gave some very easy steps and advice to perform this action. However a comment was made regarding what exactly the Cluster Group Network name was. It was stated that *"It is merely a DNS name"* and this is not correct and may get you into hot water if you take that comment at face value.

![Not so easy](/images/2010/take-advice-carefully.jpg)

You may remember my post [Protect your failover cluster names](/posts/2010/10/22/use-protection-to-stay-available) you might recall me discussing why and how you should protect your Cluster Group network names. Firstly to reiterate, the names are simply Active Directory machine name entries -just like any other Windows Domain server or desktop. These also have a respective DNS entry for the configured Cluster Group IP address.

Performing this change actually does two different things.

1. Updates the Network name in Active Directory, renaming it to reflect the new name.
1. Creates a new A record (otherwise known as a host record) in DNS to map your new network name to it's respective IP.

The first step is of particular importance. The key term here is update (not create). Should you or your Active Directory administrator make the disastrous decision to remove the original network name prior to the change then nearly all is lost so don't do this! In a further post I will demonstrate one way in which to recover from this situation.

For the second step, the DNS entry is simply being created. This means that the old record will be left behind so you should ensure that this is tidied up post change over.

So without further ado I think I should demonstrate the Cluster Group network name change. Once we have determined the network name that we wish to change the first thing we will do is have a quick look within Active Directory to check that the account does exist (which of course it should).

![Cluster Group Name](/images/2010/activesdirectory1.png)

If you also look up this `A` record in DNS you will note that the IP address that it resolves to will be the same as your Cluster Group network name's dependent IP address. The process to changing the name is incredibly simple. Whilst the network name is active, bring up it's properties window.

![Name properties button](/images/2010/changename1.png)

So the next step is to enter your desired network name in the misleadingly titled textbox of `DNS Name`. Changing this will update your existing network name within Active Directory and create your new DNS `A` record as described earlier.

![Name properties](/images/2010/changename2.png)

Clicking the Apply button at this stage is not your last chance to back out however the next dialog is, so be sure this is what you want to do. You will be informed that proceeding will cause any existing clients to this network name connectivity problems (well what did you expect!). Click Yes and your changes will be updated. The network name cluster resource will go offline temporarily and once the changes have been made will automatically come back online under the new name.

![Confirmation to change](/images/2010/changename3.png)

So one final check, after going into DNS and checking that your DNS `A` record has been created (you can potentially remove the old one at this stage) we should then check active directory to review the network name change.

![Name name in Active Directory](/images/2010/activesdirectory2.png)

In conclusion I would just like to add that when listening to advise on forums, newsgroups, Twitter, websites and blogs (including mine) please ensure that you bear in mind *everyone is fallible*. Even though advise and information will be given with the best of intentions it can still be incorrect. if you blindly follow advise without your own thorough research and background validation you may land yourself in very hot water. Even the most highly skilled professionals can get it wrong sometimes (including MVPs), as I am sure they would themselves readily acknowledge, so it is important that you (if possible) test any recommended solution in a test environment before proceeding in a live one.