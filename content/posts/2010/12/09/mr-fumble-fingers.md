+++
date = '2010-12-09T14:27:00+01:00'
draft = false
title = 'Mr. Fumble Fingers - a case against clustering'
+++

If you are not perfect (and I am guessing you are not!) you will have been in that situation where you have clicked your mouse button (substitute with your own horror story) and then almost immediately thought *what have I done!* followed by an intense feeling of self loathing, self pity and general hopelessness.

Actually as DBA's, being fallible is not really an option and is why our positions of risk are usually rewarded on many various levels -of which higher salaries only play a small part of it. In the early days of my long IT career I recall two scenarios that I remember that feeling of despair (stories which I'll save for another rainy day) but on both of those occasions they were ironically Cluster related and on both occasions I could argue the mistakes had been entirely preventable -and not just by myself. One issue was related to hardware and the other to the software setup.

![About to crash](/images/2010/driving_with_a_phone21.jpg)

There is a fallacy in IT departments with no experience of this technology that SQL Server Clustering automatically equals High Availability, and this belief is fundamentally wrong if your HA environment is in the wrong hands. Versions of Clustering prior to Windows and SQL 2008 was even more prone to failure due to mismanagement and misconfiguration.

A point I have labored over the years to get across to various middle and senior level management personnel was that the entire Clustering infrastructure (should it ever be implemented) needed to be treated as a completely separate entity and have highly experienced and trained support staff managing it. Administrative access to the environment would be restricted only to that specialized team and would comprise of only the best and most experienced DBAs, Network engineers, Windows, and SAN admins. If you fail to get the message across then the story *will not* have a happy ending.

![Naughty boy](/images/2010/spanking.jpg)

Unfortunately, in so many instances, IT departments integrate Clustering into their support infrastructure in exactly the same way in which they would any other new technology and this is when problems will strike and turn your HA solution into a smoldering piece of rubble (or as good as).

SQL Server Clustering is a fantastic piece of technology in the right hands and is getting better in every single iteration but it has to be understood and managed correctly. It is sensitive to change (and this could even be something as innocuous as firmware updates to your SAN) and when it goes wrong usually requires a wide generalized skill set and specialized SQL and Clustering knowledge to fix problems. Much of the time many problems could have been avoided had you put in place the right support structure.

Whilst I always think of SQL Server Clustering as *Higher Availability*, implemented incorrectly you might as well think of it as *Hopefully Available*.

Happy Clustering!