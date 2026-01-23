+++
date = '2011-02-01T19:39:00+01:00'
draft = false
title = 'What is Disaster Recovery?'
categories = ['Technology']
tags = ['HADR']
+++

I was recently asked *"What is DR?"* when talking about SQL Clustering, Mirroring and the like to someone. The question was asked not because they didn't know but because they were interested in what I thought. The answer is quite simple right?

After a moments pause I answered, *Disaster Recovery is the ability to restore a companies IT infrastructure to a minimum level required in order for that company to continue operating*. OK so that wasn't a really great response and of course DR means different things to different people, so it is probably important that we can be as explicit as possible. When the business asks *"Have you got a DR strategy in place"*, I personally believe that we should already have established with the Business what they would class as *"Recovery"*.

Sometimes on failure of a system, recovery can of course be the process of bringing online a standby system (in an alternate location) and restoring the data to an agreed point in time but in other instances it could mean something entirely different. For example, I have supported systems that aggregate disparate data (think of something akin to a data warehouse) and all of this data could be regenerated. Therefore recovery of this aggregate system could simply mean bringing online a replacement empty aggregate server and regenerating it's data from either the source systems OR DR source systems of the data.

![Where is my car?](/images/2011/wheres_my_car.jpg)

But I think the bottom line is that it is not the role of the IT professional to decide what DR is, our job is to design and implement a solution to an agreed set of recovery point objectives (RPOs) and these in my humble opinion *need* to be decided by the Business.

Therefore if I was asked this question again my response would probably be something like "Disaster Recovery is the ability to restore a companies system/s to an agreed set of RPOs decided by the Business". You should note that I have used the word system to reflect the fact that the recovered system may not even be IT based depending upon the Business requirements of your implementation.

*The next time someone asks you whether you have a DR strategy in place, have a think about who decided what the requirements were. Perhaps it is time to talk to the Business?*