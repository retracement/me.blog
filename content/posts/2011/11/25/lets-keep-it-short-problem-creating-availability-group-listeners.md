+++
date = '2011-11-25T00:55:00+01:00'
draft = false
title = 'Keep your Availability Group listener name short'
categories = ['Technology']
tags = ['Clustering','SQL','Availability Groups']
+++

Someone once said to me that he thought I should work as a GUI tester since I always seemed to find problems in the UI -after I had highlighted a problem to him that I'd found in Linux Gnome which he had never seen before *"after all his years working with the product"*. I guess there is an element of truth to that statement since I really do like clicking around and trying different things. However I suspect that my latest discovery won't necessarily be one of those obscure problems that no-one else comes across since it is such an obvious problem and fairly easy to run into.

Whilst playing around with SQL Server 2012 (Denali) Availability groups, I discovered that there is a restriction set on the size of the listener name that you are able to enter through the GUI.

Let me demonstrate...

First I will create a new DNS host name called *readpastandfurious* which you should note is 18 characters long.

![Create new A record](/images/2011/createdns.png)<br/>
*Create our "longish" DNS host record*

And of course there isn't a problem creating this record since the restriction upon the size of the DNS host name is (I believe 24 characters).

![DNS record created](/images/2011/dnscreated.png)<br/>
*DNS Server will now resolve our host record*

Next I decide to create an Availability Group Listener via the SSMS GUI wizard and am very surprised when I can only type in 15 characters! Obviously this text field must have been programmatically restricted since by default the GUI text fields would not be limited.

![Run out of characters](/images/2011/dialogbox.png)<br/>
*Oh dear we are out of characters*

My first thought was that it has been restricted on purpose perhaps for an odd NETBIOS reason -if for example NETBIOS was being used for resolution in any way in addition to DNS (daft but possible). Microsoft limits NETBIOS names to 15 characters, so this might have been a plausible explanation.

Next I decide to try and create the full 18 character availability group via T-SQL and I am a little surprised to find that it succeeded with no problems at all!

```sql
ALTER AVAILABILITY GROUP [READPAST &amp; Furious Optimistic]
ADD LISTENER 'readpastandfurious' (WITH IP ( ('10.0.5.1','255.0.0.0') ) )
```

![Successful create through T-SQL](/images/2011/queryworks.png)
*...but as I suspected, via TSQL is hunky dory***

---

My conclusion then is that this behaviour is almost certainly a bug and was introduced by a programmer who either doesn't understand the difference between NETBIOS and DNS or blindly assumed that 15 characters is enough?! In any event I believe this is a problem and have therefore raised a connect item for this issue [here](https://connect.microsoft.com/SQLServer/feedback/details/708320/availability-group-listener-restricted-to-15-char-via-the-gui) give it a vote up if you believe you might fall prey to this issue at some point in the future!