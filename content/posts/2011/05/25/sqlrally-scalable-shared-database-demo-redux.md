+++
date = '2011-05-25T23:24:00+01:00'
draft = false
title = 'Using SQL Server Scalable Shared Database'
categories = ['Technology']
tags = ['SQL','SQLRally','Scalability','Public Speaking']
+++

![Presentation slide](/images/2011/orders.jpg)

Several weeks ago now I had the honour of presenting at the very first ever SQLRally event in Orlando Florida and I gave a presentation around the very large (no pun intended) subject of scaling out your SQL Server data. At the end of one of my demos it unfortunately crashed and burned and the annoying thing was, that up to then it had worked flawlessly!

The problem that I hit in the demo was (I believe) related to a virtual machine crash that had happened the previous evening and I don't think I allowed the database to run through recovery the following day. I am presuming that my iSCSI targets were not connected to my SQL Server on startup which meant that the database in question which was located on my shared drive still needed to run through recovery and couldn't because I had (as part of the demo) changed the attribute of the drive to read-only in order to prepare it for the next stage. The error was suggesting that there was a version compatibility problem between the two instances and I had several members of the audience suggesting that this was the problem when I knew the versions were identical. Disappointingly, we were so short of time that I needed to move on.

![Scalable Shared Database](/images/2011/scalable.jpg)

I am definitely not the first presenter that a demo has not quite succeeded and I won't be the last, but the important thing is that I offered a solution to the audience...."I'll post a blog!". So here we are and here it is, all the information you really need to setup your Scalable Shared Database.

The first thing we need to do is to provision our shared storage that will eventually be presented to all our SQL Instance servers in order that they can be part of the SSD Cluster. In my demo I used an iSCSI target but in your production environment you will most probably be attaching to your SAN LUNS via HBA cards.

It is important that you only attach the storage to only one of the servers until you have performed all the actions necessary to prepare the database and storage so that it can become a Scalable Shared Database. Once you have created your database on this shared target, it is important that you stop any activity to it and you will note that I have checkpointed so that any dirty pages are gracefully flushed to disk. Next you can then set the database to read-only and finally can take it offline. Please note that the official instructions suggest you should detach the database. I am not a big fan of detaching databases (shan't go into that now) but if you would prefer to do that instead then it is a perfectly sensible approach.

![First database](/images/2011/1-first-database.jpg)<br/>
*Seed database needs to be set to read-only and then taken offline*

So once the database has been set to readonly and taken offline, this should mean that you should not have any open file handles to the volume. If you do then the next step will fail, what we need to do is set the volume to readonly and this is done by the diskpart utility.

![Volume readonly](/images/2011/3-set_volume_readonly.jpg)<br/>
*Set volume to read-only*

First query the volume id numbers by running the `list volume` statement. In my example above you will note that my shared drive has a volume id of 3. You need to select this volume in order that you can change its attribute, I therefore selected it by entering `select volume=3`. The final part of the preparation is to change the volume to read-only and we do so by entering `attribute volume set readonly`.

Now at this stage, if this attribute really has been set then any attempt to create new files or folders on it will fail as the screen cap below demonstrates.

![Test write protection](/images/2011/4-test_volume_for_write.jpg)<br/>
*Can no longer write to read-only volume... obviously!*

At this stage we are all ready to go! We can bring our SSD online on our very first instance and as you can see below, the database is open for business 🙂.

![Bring database online](/images/2011/5-bring_online.jpg)<br/>
Bring database back online

For all subsequent nodes you will need to connect the shared disk target to them. It is probably best that you do this one at a time only after you have added in the Scalable Shared Database to the instance, although theoretically it shouldn't matter a jot.

![Connect iscsi target](/images/2011/6-reconnect_target_on_next_server.jpg)<br/>
connect to target on next server

We are now very close to completing this exercise and the final stage is to attach to our SSD located on our read-only shared storage. Simply execute the CREATE DATABASE .. FOR ATTACH statement and voilà, we have my friends a lovely jubbly 2 node SSD cluster.

![Complete attach](/images/2011/8-complete_attach.jpg)<br/>
*Attach the read-only database on the read-only volume to your second instance*

As you can see, the procedure for creating a Scalable Shared Database is incredibly straightforward and has many benefits in certain situations and I will cover some of these in future posts as well as some of the drawbacks and possible alternative solutions. For now I hope you have enjoyed reading about something that is very rarely used, talked about or even known about. I only hope that Microsoft do build on this technology and give us something that really will set tongues wagging, a Read/Write Scalable Shared Database... *PRETTY PLEASE*!