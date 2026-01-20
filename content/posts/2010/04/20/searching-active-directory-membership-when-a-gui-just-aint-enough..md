+++
date = '2010-04-20T13:14:00+01:00'
draft = false
title = 'How to search Active Directory membership'
+++

Bit of a quickie here but a goody. Nothing really new either but since I’ve just spent the last 10 minutes piecing the command together (the last time around 6 years ago) I might as well make a record of it.

So the problem is that you have a very long list of users and groups within an Active Directory group and screen shots don’t cut the mustard any more. The solution is to use the dsquery command line tool.

![dsquery](/images/2010/dsquerytemp.jpg)

The first query simply outputs all group members in an Active Directory group and the second query outputs all the group membership belonging to an Active Directory user. Easy, simple, quick and efficient.