+++
date = '2011-03-04T16:12:00+01:00'
draft = true
title = 'Categorise your SQL Agent jobs'
categories = ['Technology']
tags = ['SQL']
+++

Perhaps you are one of those people who's house is immaculate? The lid is firmly placed back on the toothpaste, shoes left in a neat pile, coat in the cupboard. You know the sort of thing? Well I'm not quite like that but I try to be. Organized effort is a good thing and although it might not be obvious at the time can make life so much more easier further down the line.

Now what I am about to say isn't going to change your life with respect to your SQL Server Administration but it might help you take a little control of your SQL Agent jobs. Currently I have a few rules that I try to have in place for every SQL Agent job and these are:

A job must follow a standard naming convention (defined in a policy document)
Each job step must have a text output file to a standard location using the name of the job and a .txt extension
The text output from the first step in the job should overwrite and subsequent steps append
The success and failure of a job must be clearly defined within the logic of the job steps. Therefore if a job "fails" or "succeeds" then I can be certain that this was the outcome of the process. In other words I do not allow a failure to be overwritten by a subsequent job step success.
A schedule must be clearly defined and named as (defined in a policy document) using the frequency within the schedule name.
I think this is about it, and apart from this organized approach, due to the high number of jobs we have running on each instance (our developers like jobs) it is at times a bit overwhelming.

![SQL Agent Jobs](/images/2011/jobs.png)

So if we just take a look at the SQL Server Agent jobs, here we have something similar to what you might see on any SQL Instance throughout your organisation. There is a mix of maintenance jobs and jobs related to business processes. The biggest problem is that it is not always easy to determine which is which. For instance, is Reorder a maintenance job or a business process job? -and here we only have a small number of them configured, imagine what it would be like if the jobs numbered 20 or more! Luckily there is an often unused feature and that is the ability to set a job "Category" to assist you in your organized administration. As you should see within the Aggregation Cert job there are already a selection of default Categories to choose from.

![Category list](/images/2011/default_list.png)

But what if we want our own? Well have no fear it is simple to do. Simply right click the Jobs node within the SQL Agent and select the "Manage Job Categories" menu option. Within here you can define and assign new Job Categories for agent jobs.

![Manage job categories](/images/2011/manage_cat.png)

For the purposes of this demonstration I have used one of the default categories and assigned it to my existing maintenance jobs and in addtion I've also created "Trading Application" and "Analytics Application" categories. As you will see below the jobs can be sorted or even filtered using the "Category" so this gives you great administrative power of tying up some of the loose ends in you SQL Agent Jobs and giving you a bit more of an idea of which jobs are related -and since these same categories can be deployed on other servers, this idea could potentially be even more useful in a multi-server environment.

![New category](/images/2011/post_cat.png)

So look, I don't pretend to have solved all your administrative nightmares here, but I think you'll agree assigning Categories to your jobs could save you at very least a minor administrative headache.