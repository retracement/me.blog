+++
date = '2014-04-25T08:33:00+01:00'
draft = false
title = '10 reasons why I HA-te you!'
categories = ['Technology']
tags = ['Availability Groups','Clustering','HADR']
+++

Designing, planning, deploying, administrating and recovering (known from now on as **D**e**P**lan**D**e**AR** – and pronounced in the tone of a Caribbean Grandma 🙂 ) a SQL Server High Availability and Disaster Recovery (HADR) solution is really not an easy thing to implement and maintain. There are many reasons for this and whilst (ultimately) our business is only interested in an almost permanent unbroken connectivity to the Database Engine for its Applications or Middleware Clients, the reality is that our databases/s play just a very small part of the whole availability story when maintaining one of these solutions.

The skills and knowledge required for DePlanDeAR for HADR solutions generally span many different teams, require many different Subject Matter Experts (SMEs) and ideally at least one person to communicate and co-ordinate between them with enough insight and direction to achieve and maintain a robust solution. More often than not, I arrive on client sites only to find High Availability strategies that were thought to be fit for purpose when in fact the opposite is usually true.

In this article I am going to list ten of my favorite (if that is the correct term) reasons why I hate YOUR (yes your!) HADR strategy.

# 1. You have no idea what Quorum Model you are running under

Quorum is the mechanism used by your Windows Cluster Nodes and determine which can be considered part of the running cluster or which have lost connectivity. In this way Quorum aims to prevent split brain scenarios in which connectivity between nodes is lost. By default each node has one vote which makes up a Quorum maximum and a node will need to see a visible Quorum majority in order to continue running in the Cluster. A loss of a visible Quorum majority by a node would cause it to go offline (important to note in this scenario the Server itself does not shutdown!) and any cluster resources currently owned and running on it would fail over. Therefore as you might guess, Quorum is one of the most important concepts in Windows Clustering and is used by both SQL Server Failover Clustering and AlwaysOn Availability Groups.

Why then, do so few IT professionals from Windows Admins to Database Administrators still fail to make the effort to find out what exactly your Windows Cluster Quorum Model is? Time and time again I have seen the legacy Disk Only Quorum (a throw back from Windows 2003 and earlier and is a single point of failure) configured in Clusters that are not just two nodes but consist of many nodes. There is no excuse, not in any situation to use Disk Only Quorum these days. You should ensure that:

1. You understand the concept of quorum and the effect it has to your cluster's availability, regardless whether you are a DBA or Windows administrator.
1. Ensure it is changed immediately to a more appropriate model!

There have been significant Changes to Cluster Quorum in Windows 2012 and further advancements in Windows 2012R2. These changes will help make Quorum configuration and Cluster availability significantly easier and more efficient. I shall cover these another time but for now you can read a bit more about Quorum in my post [Weight doesn't ALWAYS have to be AlwaysOn](/posts/2012/04/05/weight-doesnt-always-have-to-be-alwayson).

# 2. Your Windows, Network, Storage and Database teams work in isolation

There are not many technologies that require the crossover skills in the way that SQL Server High Availability solutions do. Not only do you have to worry about SQL Server functionality itself (and let's not forget that it does help if you understand some of these HADR subjects quite deeply to avoid their nuances), but you also need to have a good understanding of Windows, Active Directory, Networking and SANs to name a few others. Probably one of the most common scenarios I encounter is teams working in Silos and communication across them being very poor. There is only ever one outcome to silo based designed HADR strategies -they ALWAYS result in bad designs, bad implementations and unconfident and ill-informed support teams.

Having technical cross-over is good. It gives you perspective, an appreciation of another team's challenges and the ability to communicate in their language. Nobody ever said you couldn't specialize in your area of choice and be a technical expert did they? You won't forget your existing skill set just because you have learnt something new. No, it will give you a foundational platform to build your knowledge. The dots start to connect and you smarter!

# 3. Your Entire Production infrastructure is all LIVE

So you have implemented a HADR strategy and all seems to work well right? Eventually you will make it live and run with success for a period of time, giving yourself a big pat on the back. Your fantastic design exists purely because you are amazing and no one else could have achieved such a feat of engineering!

Eventually the time comes to patch your Windows or SQL Servers. Then and only then do you realize that your solution requires you to deploy these to the systems that are currently running as LIVE. This mistake is more common than you would believe and your coupling between your *DR* solution and your *HA* solution is so tight that in order to patch anything in your Disaster Recovery site you have to initiate failover in Live! Sometimes it is possible to get around these situations by temporarily workarounds (such as breaking SAN replication and re-establishing later) but most probably your design only accommodates failover.

If it needs saying, ALWAYS try to decouple any strategies you implement as far as possible. There is nothing wrong with using complementing HADR technologies as long as the use of one does not compromise another.

# 4. You do not run similar HADR infrastructure in your (tick where applicable) UAT/ QAT/ Systest and Dev environments

In most organisations, Highly Available strategies are only seen as something worthy for production. I have been lucky to work for an organisation that employed nearly 100 developers and yet even there, almost zero thought had been given to the Development environment's availability. After I spent some time calculating the cumulative cost of man hours that would be lost in the event of any of the development database servers failing, it was a fairly obvious thing to suggest to them that:  in this scenario their Development environment was more important than Production!

What is more, running similar HADR deployments in other environments allows your Developers to design code that is more likely to be suitable for these platforms, allows your Testers to find platform related problems **before** code hits production and empowers you to accurately trial changes before risking doing so in live. I could go on but...

# 5. Your management think HADR is easy

Your DBAs understand SQL Server and your Windows Administrators understand Windows? You might almost be as bold to suggest that in each area of specialism, there are some real experts in those teams. Unfortunately HADR implementations span a whole stack of technologies and skillsets ranging from the obvious (Log Shipping) to the not so obvious (and seemingly unconnected) such as SAN replication or Virtualization. Understanding how all of these offerings can be used and knowing how they interoperate and play with each other can be bewildering, even for SMEs. Yes HADR will take you out of your comfort zone, but you will learn a lot from travelling to it and provide more robust solutions and more stable systems.

Remember to explain this necessity to management and make sure you can help them understand the obvious! If you need training, then explain to them **why**.

# 6. YOU think HADR is easy

You have been using HADR strategies for quite some time now and many of your peers believe you to have almost Jedi like skills. Heck you may have even started to believe your own hype and think you have every base covered.

...If only things were as simple as that.

It is always important to try to eliminate any single point of failure where possible in any HA solution, but there are too many variables to address when considering your designs. It is impossible for you to ever understand the impact of an Operating System (or firmware) patch to various parts of the environment, but ultimately one day your strategy is going to fail. How long the recovery from the system outage is going to take will depend (in part) on how well you actually understood the solution that was implemented. Having administered a working system over a very long period of time with no failures or downtime does not mean you are capable of restoring operations back to normal should they now go belly up.

Do you have confidence that (if you are given point in time sql backups) you would be able to rebuild a system from the ground up in a disaster scenario within the expected service agreements? If not, then you are running at risk.

# 7. You are not in possession of an SLA, RTO, RPO or system definition

When failure or disaster strike (and believe me, if they haven't yet, sooner or later they will), time and again I see people in positions of power, influence or command start flapping and demanding that operations absolutely have to return to a full working service immediately otherwise no Widgets can be sold by ACME Corp, and repercussions will be serious! Yet these highly charge stressed individuals are the very same people that you have approached on numerous occasions to ask for your Sytems Recovery Point Objectives (RPO),  Recovery Time Objectives (RTO) and Service Level Agreements (SLA). With a shrug of the shoulder they calmly tell you that there isn't yet any defined agreements but they are "working on it". Or perhaps even worse, they have given you documents which whilst defining what the RPO, RTO and SLA is for a particular system, they fail to **DEFINE** the system.

I have often seen people responsible for Business Availability go to great lengths to define the agreements for RPOs, RTOs and SLAs for particular Systems, but fail miserably in defining what *exactly* constitutes "The System". Every Business system is composed of many different moving parts and subsystems, both technical and non technical platforms. All of these things (as we have discussed already) are generally supported by groups of diverse teams that rarely communicate between themselves. At a higher level there are Business processes sitting on top of these platforms that will have their own nuances and quirks and require specialist knowledge.

Therefore is your "System" the entire thing that is being described above OR are you going to break it down into component parts for your availability agreements? Do you even know whether it is possible to meet objectives *if* you were forced to run recovery in a serial nature (which is so often the case in situations like these). You may find there is not enough time...

# 8. You do not regularly review OR test your HADR strategy

Your HADR plan is only as good (and no better than) the competency of everybody involved in its design and those who will execute that strategy in the event of failure. Throw into the mix a whole host of ever changing variables, technologies, services and business processes and suddenly you have a moving target to worry about. On too many occasions I have been witness to scenarios in where "The Business" would never allow a regular fail-over policy and believed that any solution currently in place would (if called upon) just work. Your problem is this; the longer it has been since you last tested your HADR plan/s, the more likely your moving targets will have compromised your solution -right? And if you agree with me; it is far better to experience a failed HADR plan when you don't have to rely on it than when you do.

It only makes sense then, to regularly review your strategies and try to reduce the risk of failures at any time, whether they occur through a managed test or because of an unseen event. I should also widen the scope further and say that if your company has a solid set of change control processes and procedures in place managing (and publicising) changes across your Enterprise, then it is far more likely that your HADR reviews are going to flag potential issues.

Nuff said!

# 9. You have no documentation (or your documentation is worthless)

By now it really should be self explanatory that if you do not have any documentation for your recovery strategy and these plans only exist in yours or someone elses head then you are destined to run into big trouble on failure. But more commonly documentation will exist, but it is unnecessarily large and difficult to follow. Maybe you wrote it with a baffoon in mind, but honestly, you do not have to describe in gory detail how to do operations that your specialist technical staff should be able to perform. If you are trying to document an operation such as *Restore database AcmeCorpBigDB and all logs including tail backup with norecovery from the most recent taken on Production server AcmeCorpProd1 to AcmeCorpDR1* then that is all you need to say. You do not have to explain which buttons to press or go into detail about how to do it in TSQL versus a GUI based restore (or even use that funky font you have recently discovered to make it look nice), just get straight to the point. Putting sidenotes that might assist in speeding up the process is just about acceptable, but any detail (for dummies) should be referenced through footnotes to other easy to find documents.

Assuming that you have written (in your opinion) the *World's Greatest Recovery Plan*, make sure that someone else gets to appreciate your good work by actually getting them to put it to the test and ultimately give it a quality assured stamp of approval. Choose your most junior member of the team and if they struggle to achieve recovery without asking questions or are doing something wrong, then either your documentation is not fit for purpose OR they need further training. In any event, you should always look toward the documentation as being imperfect before you assume that your Junior DBA needs a brain transplant. Remember that they wowed you in that job interview, so the likelyhood is that you (or your documentation) is at fault.

A final point worth mentioning on the above is that when your most Junior DBA is given the task to perform recovery, make sure that your most Senior DBA is given the task of shadowing them. Make sure that both parties understand that no help will be allowed and that the Senior DBA is simply there to protect the Junior from themselves OR the poor documentation. Remember to emphasize  that the documentation is being tested here NOT the Junior DBA.

# 10. You have no 24×7 Support for your 24×7 Operations

How many of you these days work within an oncall rota? That's great isn't it? The main problem with oncall is that every team will have ever so slightly different arrangements and understanding what **exactly** the oncall rota really means when you are oncall. Furthermore, since there is nobody actively watching and monitoring the systems availability, by the time you get to hear about a problem, several hours will have already passed -so much for your Highly Availability Service Level Agreements!

Usually an even bigger threat to recovery of your systems during your oncall hours is the time it normally takes to mobilize all the necessary teams to fix the problem. That is, if you have even managed to identify what **is** causing the problem. Communications across teams seems so much harder and takes so much longer when you should be getting your beauty sleep...

You may now be thinking that I am suggesting off-shoring your night-time support operation? Personally I would only ever suggest doing this if your offshore support have the knowledge and capability to actually fix problems themselves when they happen. If all they are there for is to escalate problems to you when issues are seen, then all you have achieved is added yet another element of perplexity to your support.

# Bonus: You are at the mercy your outsourced service provider

I could tell you stories about this that would chill you to the bone. But I shall spare you from the horror and simply say that if you are fortunate (or unfortunate) to outsource any of your IT service or infrastructure to a managed service provider, you had better make damn sure they can deliver on any promises that have been made within your SLAs, RPOs and RTOs. If you ever need a new server or new SAN provisioned instantly for whatever reason, can they deliver it within an acceptable time frame? No, of course they bloody can't!

Have you even bothered to formulate specific SLAs, RPOs and RTOs with them? No I thought not...

---

Well thank you for taking the time to read my list and I hope you have enjoyed them.