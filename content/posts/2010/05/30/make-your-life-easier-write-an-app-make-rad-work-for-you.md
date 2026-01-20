+++
date = '2010-05-30T16:41:00+01:00'
draft = false
title = 'Develop query tool to return multi-query results'
+++

Quite often I come across scenarios where you know what you need to do, you know how to do them, but know the way to do it will be very repetitive in SSMS. In these instances the best way to overcome them is to create a few queries and join them up into a little applet.

One such example I had recently was that we wanted a way to very quickly check several servers for privileged database and server role membership.

So in simple terms I needed to see every login that was a member of any of the server roles and in addition I needed to see any database user that was a member of database owners database role or the owner of the database directly.

![Query results object](/images/2010/applet1.jpg)

This is actually something that you would have thought should be pretty easy to do in SSMS, but this is not really the case. The GUI requires that each principal is looked at one at a time in turn viewing the membership of each, and it is quite labourious and infuriating when there are many to look at.

The solution is relatively simple to code in T-SQL using the Dynamic Management Objects but running this script against more than a couple of servers in an adhoc fashion really could be simplified. In my particular situation I was asked to task a junior (non SQL operational) DBA with compiling this information and therefore the obvious solution to was to make this as easy as possible for them. It took me around 15 minutes or less to write a front end GUI which allows the user to enter the server name in a text box and both queries are ran against the server in question and the results are presented in a MDI child split pane window.

![MDI](/images/2010/applet2.jpg)

Maybe one day SSMS will provide more extensible functionality, but for now small applets in certain situations work well for me.