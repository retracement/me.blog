+++
date = '2010-12-23T13:20:00+01:00'
draft = false
title = 'Troubleshoot SQL Service connectivity'
+++

We had a fairly interesting incident recently where there had been some significant network issues which even once they had been resolved had left various connectivity problems to several of our SQL Instances. I thought it would be useful to take a few screen shots and demonstrate my thought process when this sort of reported incident occurs.

The first thing I am aware of is that our development support team has stated that an application is having problems connecting to the SQL Server. OK well the first thing I think is *lets try it for ourselves under my own system administrator security context*. The security angle is important since the problem may be related to access rights. The result of my attempt is that we at least prove that something is not quite working as it should and the instance cannot be connected to for whatever reason.

The very next thing I try is to ensure that the Windows Server is up and responding. Therefore you guessed it, we ping it! Yep that's not a problem, resolved and straight back at us.


![Remote connection](/images/2010/remotesqlcmdx.webp)

Next I want to make sure that the server IP being resolved is the correct IP for the server name. Secondly I want to have a look on this server to see that SQL is actually running. So once I have used Remote Desktop to connect to the server in question and am happy that the server name is indeed the one I expected, the next thing I want to know is *can I connect locally*. Straight away we connect in, and that at least shows that SQL is running. There are several things that can be wrong now, one thing we know for sure is that a remote connection cannot be made.

![Bad port](/images/2010/badport.jpg)

You might be thinking to yourself why would the local connection succeed and the remote one fail? Well there are several possible answers to that question, but I should state that local connections are more likely to succeed due to the default protocol configuration -more specifically that Shared Memory is specified as the first client protocol. Don't forget that TCP/IP is second in the list and more importantly, even though Named Pipes is third, by default it is disabled in the Server instance protocol. The next thing that I want to see is to take a look at all the SQL services that are currently running on the server. The reason why I want to do this will become clear in a while, but we already had proved that the SQL Server Service for the instance was running.

First I try to connect locally to the SQL instance:
```bash
sqlcmd -E -S localhost\sql01
```

But unfortunately the services still does not connect. So we view the SQL services that are in running state:

```bash
net start | find "SQL"
```

![View started services](/images/2010/servicesx.png)

So at the moment, everything looks normal on the SQL Server Instance. Usually in this scenario for remote connectivity problems I *ALWAYS* take a peek at the TCP/IP properties which can be found in the SQL Server Network Configuration node of the SQL Server Configuration Manager tool. Straight away I note something quite odd, that Dynamic Ports are enabled and therefore this server instance has not been configured correctly since I insist upon a fixed port range for all our SQL Instances.

![TCP Dynamic Ports](/images/2010/portsx.png)

I could perform a few more tests now if I felt it was necessary such as forcing a remote *sqlcmd* session to connect to the 1271 port for the SQL Instance, but I don't feel it is really necessary. I know that the *SQL Server Browser* service is running OK from when I listed all the running SQL Services and I know that the Browser services role is to send the port number of the running instances to the SQL Client for remote connections. I also know that we have had strange network problems, so I therefore take a guess that the Browser service is returning an incorrect port number (or at very least is not functioning correctly). I decide to restart it and see if that does the trick.

![Restart browser service](/images/2010/restartbrowserx.png)

After a graceful stop and start, I again go back to my remote session and try again. This time connectivity has been restored to me and the original application login fault is no more.

![Remote SQL connection](/images/2010/remotesqlcmdconnectx.png)

I should point out that there are a couple of things that I really should have followed up on, but sometimes post resolution investigation is not possible in busy environments. The things that I should have done were to have looked in the SQL Server error log, the Windows Application and Windows System log to identify any errors listed that may have been connected to this problem. For instance what we didn't prove was whether the Browser service had stopped working properly or whether the listening port had actually changed. These would have been very easy to look at and prove and the next time I see a problem like this I will go and have a look.

I would also stress that had a fixed port number been assigned to the instance correctly in the first place it would have at least removed one possible area for failure. I will discuss in more detail why you should use fixed ports for your instances at a later date, but if you are not already using them then you need to think carefully about why you are not.