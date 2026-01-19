+++
date = '2010-01-19T14:23:00+01:00'
draft = false
title = 'Disabling and Enabling Oracle Scheduled Jobs in sqlplus'
+++

The first thing you need to know is what jobs are scheduled and enabled. To do this we simply query the `dba_scheduler_jobs` view as follows:

```sql
SELECT owner, job_name, enabled FROM dba_scheduler_jobs;
```

![Query scheduled jobs](/images/2010/configschedjobs.webp)

Now we know what jobs are present and enabled/ disabled we can use the [DBMS_SCHEDULER](https://docs.oracle.com/cd/B28359_01/appdev.111/b28419/d_sched.htm) to change their status. The `DBMS_SCHEDULER` is a package which provides a collection of scheduling functions and procedures and in our case since we wish to either disable or enable a job, we simply call its Disable or Enable procedure as follows:

```sql
begin
DBMS_SCHEDULER.disable (name => 'STS.STS_OVERNIGHT_REFRESH_JOB');
end;
```

![Disable/ Enable jobs](/images/2010/disbenajobs.webp)

After you have run the PL/SQL batch to disable or enable the desired job, another check of the dba_scheduler_jobs view will prove that the job state is as you expect.

Enjoy!