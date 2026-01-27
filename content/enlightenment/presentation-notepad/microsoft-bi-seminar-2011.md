+++
date = '2011-03-29T00:00:00+01:00'
draft = false
title = 'Microsoft Business Intelligence Seminar 2011'
categories = ['Technology']
tags = ['Training', 'Business Intelligence']
+++

The seminar was given on the Microsoft Campus at Reading UK on the 3rd March 2011. The presentation was split into four sections and the end of each section represents a real world intermission. You would be advised to download the slides (download link in notes) and follow the notes as you step through the slides. I personally think there is some excellent information embedded in these notes, your job now oh dear Business Analyst is start “data mining” for what is important and what is not! Within these notes you will also find a video link to last years presentation. Unfortunately due to the *NDA* nature of some of the information talked about this years session, it was not recorded.

If you notice any inaccuracies, *please* let me know.

---

# Part I

Works for [Project Botticelli](https://www.projectbotticelli.com/).

BI will become the most important aspect in IT over the next 10 years. Will be putting up videos over coming months, send him an email to get on mailing list.

BI Analysis will become self-service.

The way we provision and manage data is changing.

For his May delivery of the Business Intelligence Roadshow apparently the video should be available somewhere for download – *think it is [here](https://www.microsoft.com/everybodysbusiness/en/gb/business-intelligence.aspx)*.

Slides are published at www.projectbotticelli.com/downloads/public

This session is not being recorded.

People tend to believe the results of their analysis and reporting if it looks right without even analysing whether it is right.

With self-service Business Intelligence we have now lost the control.

How do you keep control?
Insight vs oversight.
Balance agility with control.
*There is a Gartner White Paper concerning balancing flexibility with control. …not sure if this is it – [Business Innovation Will Come from Organisational Openness, Says Gartner(https://www.gartner.com/it/page.jsp?id=541907).*

There was a study showing that 99 percent of all successful people use Microsoft Excel *-not sure who made this statement*.

PROVISION, EMPOWER and MANAGE are the three keywords for the evolution of Business Intelligence.

Gartner -Business Intelligence platform magic quadrant.

## Provision
 
Access to information.
Heterogeneous connectivity.
Microsoft is the number 1 vendor of OLAP cubes.
Yahoo runs on Microsoft technology.
See the figures on the slide.

![Build on What You Already Know](/images/2011/build.jpg)

## Empower
Make is easy for them to access data and mash it up. SharePoint (he refers to it as *"Office Server"*!).
Need to install free [PowerPivot add-in](https://www.powerpivot.com/download.aspx) from microsoft.com to make it available to Excel.
Splicers and sparklines are probably 2 easiest ways to visualize data in Excel…
The trick of PowerPivot Excel is in its use of the memory.
PowerPivot de-dupes data and it manages to do this by using a binary index.

## Manage

With self managed Business Intelligence will come self created problems such as the Airline freight distribution story that described how a single Excel spreadsheet became innocently distributed throughout the Airline (was written by a single manager to speed up his processes) and the entire Airline became dependant upon it without even knowing it (…and then there was a problem with the spreadsheet on a leap year). This story was given originally by Donald Farmer.
SharePoint going to move into everything we do in the future.
Frictionless deployment is Microsoft’s new keyword and an example of this would be found in SQL Azure.
The Microsoft Cloud offering differs from all the other vendors direction in that the way you develop software for the cloud is the way you develop software for the database. But they are also changing what that direction is.
Microsoft want the next version of tools to provide *self service-ness*

Ti foundation (*not sure whether this should have been Team Foundation*!) server add in to Excel.

PowerPivot for SharePoint get gallery to choose from.

The Gallery was created so that users can view what they want to open.

Using SharePoint is a good idea so uploading Excel spreadsheet up there as more of a collaborative effort may have prevented airline problem – particularly in the sense that the activity to the spreadsheet (or dependency) could have been reported on.

SharePoint shows activity bubble-chart which would show popularity of documents. – again good for airline story.
*This is how we gain control.*

PowerPoint management data.

Office
SharePoint
SQL
…are the trinity of Business Intelligence.

![Microsoft Business Intelligence](/images/2011/ms_business_intelligence.jpg)

---

# Part II
Self-service Business Intelligence.
Cubes and PowerPivot are almost the same thing conceptually though, PowerPivot doesn’t use cubes.
Slicer is EASIEST to visualise data. It feels like PowerPivot but is not part of it.

Data mining getting easier.

PowerPivot supercharges Excel.

Excel services are part of SharePoint
Is more for Enterprise use.

Otherwise use Excel web application
It is collaborative and good just to go through browser which avoids client version issues

Get the Data Warehouse version of the AdventureWorks database.
For cubes in Excel…
Click “From other sources…” button

Cubes are aggregational information indicators.

Slicing is when you choose a dimension.
Dicing is when you choose an additional dimension but he says in reality to dice it we need to have 3d dimension :)!

Cubing IS advanced analytics and everyone is doing it.

Data-mining is correlation search engine to find meaning events.

[Kdnuggets.com](https://www.kdnuggets.com/) is a good data-mining website.

Can use data-mining in Excel if you have SQL R2 license. Also need to download data-mining add-in and at the moment it is currently only available in 32bit! ...*have found this good reference for this addin [here](https://social.technet.microsoft.com/wiki/contents/articles/using-the-sql-server-data-mining-add-ins-with-powerpivot-for-excel.aspx)*.

Adds in extra buttons to Excel.

In Excel select all data and click analyse button. Data is sent to SQL Analysis Services engine and will find data exceptions.

Exceptions are pieces of data that are outside centre of gravity -and this is based on *connectivity* of data.

Uses *Microsoft Clustering* algorithm.

Association rules algorithm good for finding association.

PowerPivot he calls it *"the server without a server"*.

To use PowerPivot click PowerPivot button then to load data into PowerPivot connect to data.

Windows Azure marketplace data-market

Reports can be data-sources!

When PowerPivot cube is prepared is when CPU will spike on client. At the time de-dupe occurs.

When adding extra data-sources it is possible to get Excel PowerPivot to auto find relations.

Can put sparklines even ontop of data although it doesn’t look very visually appealing.

BISM uses DAX’s language.
Business Intelligence semantic model.

PowerPivot today works on Excel or SharePoint for it to work on the later, you need SQL 2008R2 Enterprise and SharePoint 2010 Enterprise.

PowerPivot compression in Excel will give you a ratio of anything between 1 to 10 and 1 to 100

Excel will still open file if add-ins are missing but will be able to only show the results.

How do cubes scale? Microsoft solved problem by using cubes without cubes i.e. PowerPivot!

DAX means Data Analysis eXpressions.
DAX does not replace MDX since the latter is still needed for more complex operations.

The most important DAX function is calculate.

---

# Part III

Says today, SSRS is for powerusers.
Other types of reporting is for everybody else.

Performance Point Services is slightly easier than SSRS.

Visio is great to run with Performance Point Server.

Presentation last year covered Performance Point.

Bing mapping tool.
On Codeplex need to search for [Bing Data Connector](https://dataconnector.codeplex.com/).

Silverlight pivot tool.

Azure report has to have Azure data-source.

Note that there is functionality lost in SSRS if native mode is not used.

Says of SharePoint integrated mode that it is SSRS mode we should ideally consider.

Look at his native mode web farm. Using NLB.

SharePoint has a recycle bin that can have retention set to prevent deletion.

Report manager.
Reportviewer control.
Reporting services web parts and connect into SharePoint.

When building reports you can zoom in and out but once it is deployed then it is then static

People are happily using SSRS in their companies but all their data is sometimes held in Oracle or SAP etc.

---

# Part IV

Why MDM?
Track change in IT through 4 aspects.

How do we prevent duplicates virtual databases? For instance to have a DATABASE agnostic to SQL, Oracle etc. -but is never going to happen since this would effectively mean they will be all the same engine/ technology.

![MDM Processes](/images/2011/mdm_processes.jpg)

2 approaches
Federated

databases – he says if you are running things like BizTalk the should seriously consider using it
Or
MDM

What is Master Data? -it is critically important data that presents a “single truth”.
Can every transaction be master data?
Every dimension could be considered as master data.

MDM is related to federated data management.

MDM isn’t really Business Intelligence but its usually always Business Intelligence guys who look after it

Story about sales locations and that in future may be subscriptions to single truth data.

Choose the master data that is relevant to you and then move on!

Entity modelling is quite good conceptually to create an MDM model.
Object Oriented modelling is also good for doing this.

There is Data Warehousing 2.0 Modelling concept *...believe he is talking about the book “DW 2.0: The Architecture for the Next Generation of Data Warehousing”. This url I found is also relevant for this concept and Parallel Data Warehouse – "[Data Warehousing 2.0 and SQL Server: Architecture and Vision](https://msdn.microsoft.com/en-us/library/ee730351.aspx)"*

It sends an email every time validation runs and is there is a rule failure.

You can create your own hierarchies to sell to the commercial sector in Azure Marketplace

MDM is not about technology but about the people who use it.

Perhaps in the future MDM will be built into the Database and application design?

SSDQS may be built into SSIS.

He likes to have a data quality scorecard for data cleansing.

![](/images/2011/close.jpg)

---

Well dear reader, that concludes my notes. I hope you found benefit to them and your feedback would be welcome since I have lots of material like this up my little (or is that big) sleeve.