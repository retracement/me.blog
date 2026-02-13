+++
date = '2018-10-01T17:07:00+01:00'
draft = false
title = 'Custom Job scheduling, Remote queries, and avoiding false negatives using PowerShell and SQL Agent'
categories = ['Technology']
tags = ['PowerShell','SQL']
+++

Say what you want about the SQL Server Agent, it is the lifeblood of SQL Server for scheduling maintenance routines, data loading, and other such ongoing operations. It has been around since the dawn of time and has grown into a rich (but simple) job execution scheduler, yet at the same time, still lacks certain capabilities that mildly annoy me.

# Scheduling exclusions
One such annoyance is the inability to create intelligent exclusions for job schedules. In other words the *"Don't run this job if"* scenarios that you might commonly run into at retailers, education or health sectors, financial institutions and other environments that may have time-sensitive or complex execution logic.

Now I know what you are thinking. You are thinking that you have created plenty of job schedules in the past and managed to add exceptions to the schedule, and with this, you are partially correct.

![New Job Schedule](/images/2018/schedule_dialog.jpg)

In the SQL Agent Job Schedule dialog above, we have a weekly schedule that is set to run every Monday to Friday, running only twice a day, starting at 00:00:00. By very definition, this schedule will not run on Saturday or Sunday (and therefore we have an exception), but we are constrained within the bounds of the dialog interface and available fields and logic.

# Complex scheduling logic requirements

Consider an environment that consumes lots of ETL feeds from third parties on a (working) daily basis. The feeds, however, are not created during holidays, meaning that the dependent job/s would fail if they ran on those days. There are three obvious ways to work around this situation:

1. Allow failures to occur (rolling back changes on failure)
1. Embed feed existence logic <sub>*1</sub> (and/or exception logic) within the code/ package (to prevent the package execution from failing)
1. Prevent job execution on-condition within a job step

**Option 1** (quite clearly) is not a great solution and would involve *false negative* reporting (in systems such as SCOM since the error would be just that) and require human interaction to review. Jobs would fail, but the failure would only be a result of missing feeds and nothing more. Recovery in this instance would (hopefully!) only require allowing the job to run on schedule once the feeds are delivered the following day.

**Option 2** would address the problem of failure in this instance but does not implement schedule exception logic. In other words, the job (and its package) would execute successfully, but this in itself creates *false success* reporting (the package did not perform its usual workload). Implementing schedule exception logic inside the code/package is also far from ideal since its function is easily lost in the logic of the actual job step and maintaining and building upon this logic could become unwieldy over time.

**Option 3** is perhaps the best choice and provides isolation between the job schedule logic and the job function itself (because requirements can change) and improved visibility (feedback) of why the job did not run – but we still need to understand how to implement this for situations that are outside the capability of the Job Schedule dialog.

*<sub>\*1</sub> Whilst it is fair to argue that existence logic is not the solution to our problem, it should be obvious to the reader that creating packages that gracefully handle failure is a given. In reality, all package dependencies should be tested for, and reported on error -and I tend to favor the "bubbling up approach" to failure.*

# Remote queries for exclusion lookups

In this specific scenario, we need to implement our custom logic to perform our holiday test and raise an error if the current day is one (we will address the handling of this error shortly). In our environment, we maintain a centralized calendar (in a table of course!) of all important dates and public holidays up to five years in advance, and this makes for an excellent reference point for our test query. We want to store our table on a different (but highly available) SQL instance so it can be used by all SQL instance Agent jobs, but its location introduces yet another challenge to our problem -how do we query this remote data from our Agent job?

If you automatically jumped to the conclusion that we should use a linked server, then I can tell you are a bit of a masochist. One of the (several) problems with this approach is that we would have to create a linked server to our calendar data instance within every instance that needs to reference it. In an enterprise environment, this could be a problem to deploy and maintain.

SQL Server 2008 R2 introduced PowerShell Agent job steps, and this gives us a very easy mechanism to programmatically run our remote query without a linked server.

In the following piece of code, we are simply querying our remote server hosting the calendar table (`[DBAAdmin].[dbo].[FullCalendar]`) and searching for today's date.

```powershell
$serverinstance = "server1"
$sqlCommandText = @"
DECLARE @testdate DATE = GETDATE()
SELECT IsHoliday
FROM [DBAAdmin].[dbo].[FullCalendar]
WHERE DateKey=@testdate
"@
 
if ((Invoke-Sqlcmd $sqlCommandText -ServerInstance $serverinstance).IsHoliday) {
throw "Current date is a Holiday"
}
```

If today's date is classed in the table as a holiday (see the evaluation against the returned dataset's *IsHoliday* property), then an exception will be thrown and the step will be failed.

# On step failure success

The only problem with failing the job when today *is* a holiday, is that we are essentially raising a false-negative which is not much better than just allowing a job just to fail because a feed wasn't delivered for that day. In the ideal world, we need the job to quit but terminate in such a way that is uniquely different to a *"success"* (i.e. that today is not a holiday) or *"failure"* (i.e. that the job has failed for some other reason that needs investigation).

The trick here is to change the job step On failure action to *Quit the job reporting success* via the *Job Step/ Advanced Properties* as follows:

![On step failure](/images/2018/onfailure.png)

So your job steps are like this:

![Job steps](/images/2018/jobsteps.png)

By doing so, your skip logic will result in the job step failure showing the job execution as an alert (rather than failure). Only the step itself will show as a failure -which, I think you will agree, is pretty cool! This means that not only do you have a visual indicator that job was canceled by design, but you should be able to programmatically exclude these alerts in SCOM.

![Log file](/images/2018/errorstep-1.png)

---

# Summary

In this post, we have done several things in order to deliver a richer scheduling solution for our agent jobs. These are as follows:

We have prevented our scheduling custom logic job step from causing false negatives through setting its "On Failure: Quit the job reporting success" to allow the job to report only an alert but and only fail the job step.
We have created a centralized table to store custom metadata to allow all SQL instances to query it for scheduling logic scale-out (where you might need to!).
We have utilized PowerShell to query our remote metadata in order to avoid the need to create (and maintain) instance-specific linked servers. This is clearly a good thing for the deployment of standardized scripts.
I hope you can see from this very simple example that not only can we now implement very rich custom scheduling solutions for our jobs, but we can also use some of these techniques to avoid the unnecessary use of linked servers and other things. Enjoy!