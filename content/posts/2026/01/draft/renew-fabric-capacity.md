+++
date = '2026-01-19T14:23:00+01:00'
draft = true
title = 'How to renew your Fabric Trial Capacity'
+++

It is probably fair to say that Microsoft are usually quite generous in giving free access to many of their services and products when compared to other software and hardware vendors, and I think that this (in part) has helped elevate the adoption of their service usage by technical professionals much more than would otherwise be possible.

There is no substitute for hands-on learning, and where there is no availability to vendor pre-built readily deployable free temporary labs, the only other options left to us are to roll out our own, or cross our fingers that a full trial exists.

Take Databricks as an example. They provide a few options such as their [Free Edition](https://www.databricks.com/learn/free-edition) (otherwise known as the Community Edition), but it does have a bunch of [limitations](https://docs.databricks.com/aws/en/getting-started/free-edition-limitations).
Alternatively you could sign up for the Databricks [Trial Edition](https://docs.databricks.com/aws/en/getting-started/free-trial) and whilst this gives you a more functional cluster to play around with, it is still not without its [limitations](https://docs.databricks.com/aws/en/getting-started/free-trial#trial-limits). Most frustrating is the fact that the trial only lasts for 14 days!

Compare and contrast this with Microsoft Fabric. Their trial lasts for a 60 day period and usually deploys using an F64 capacity! That said, there are a few features for this capacity that are [not supported under trial](https://learn.microsoft.com/en-us/fabric/enterprise/fabric-features), and the most notable of these (at the time of writing) include:
- Workspace-level private links
- ARM APIs and Terraform
- Fabric data agent

I hope you would agree that Microsoft trial offering beats Databricks trial hands-down. However, even though 60 days is a long time in technology, it is not a long time when you are trying to learn technology. Whilst I have heard many people suggest that it is possible to reach out to Microsoft to extend your Trial Capacity period, Microsoft's official stance is that they cannot or will not do this. I have also heard from several people to indicate that sometimes you may get the option to extend your Trial period from within your Fabric profile (as it nears the end) -assuming that your trial has been sufficiently active, but this has not been my experience to date*1.

![Fabric user profile](/images/2026/Fabric%20Profile.png)

Instead the only assured way to "extend" is to sign up for a new Trial. We will describe that approach below and some of your considerations before doing so.

# Dealing with your expiring Trial Capacity

If you assume the worst (that your Trial Capacity is not extending), then you have an immediate problem to resolve. More specifically you are going to need to move any of your Fabric workspaces created under your trial capacity to another capacity. Technically it is possible to have up to 5 concurrent capacities under a single tenant *2

# Signing up for Trial Capacity again




*1 The option to renew apparently appears automatically when there are only 7 days left on the trial capacity. I will update this post as soon as I am able to validate this. With my recent experience (which fell over the Christmas break) I did not access my Fabric capacity during this time. Sorry Santa!

*2 Again I have not validated this, and would advise against too many sign ups against your tenant. Microsoft are reviewing Fabric usage patterns and clearly will not want their cloud services abused! Less is more (don't take advantage of a good thing).