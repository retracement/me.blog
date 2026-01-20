+++
date = '2010-05-20T14:05:00+01:00'
draft = false
title = 'SSMS query pane bug in Windows 7'
+++

Are you familiar with this menu? 

![Menu](/images/2010/right0.jpg)

Are you sure? 

Do you use it a lot? 

Yep me too...

Most of you will already know how to get this menu within SSMS, and one such way (the way I always use) is to right click the white space within a Query window. One of the reasons you might wish to do this could be because you are copying text. Well in Windows 7 this does not seem to work any more. I initially noticed the problem when using my spanking new SQL 2008 R2 client installation. After several installations on a couple of platforms it appears that the issue is not specific to SQL 2008 R2 but is actually related to running on Windows 7. The following all display this behaviour :

- SQL 2005
- SQL 2008 with or without service packs
- SQL 2008 R2
- (SQL 2000 not tested)

To demonstrate what I mean, lets launch a query window within SSMS and type some text and execute the query. OK so now lets say we want to copy a selection of text. After selecting, and right clicking the mouse….nothing.

![No right click](/images/2010/right1.jpg)

Strangely next I’m going to do something slightly different. With the selected text I’m going to right click and hold the mouse over it and drag across the screen.

![drag](/images/2010/right2.jpg)

Upon release this mouse we still get a context menu coming up which allows up to move or copy here. Hmm, right.

![menu after drag](/images/2010/right3.jpg)

Let’s now move into the Messages (Results pane will do also) pane of the query window and right click.

![message right click](/images/2010/right4.jpg)

Great it works!

My final tests included using the keyboard short-cuts to copy and paste texts within the upper query window and there is not a problem there either.

In conclusion the only issue being affected appears to be purely the static right click within the upper query pane window. It is not a show stopper since this menu can be accessed through the main menu bar, but it is darned annoying and makes me wonder what other little quirks I might uncover within SQL applications running on Windows 7.
