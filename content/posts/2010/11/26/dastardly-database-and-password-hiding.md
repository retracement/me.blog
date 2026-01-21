+++
date = '2010-11-26T13:44:00+01:00'
draft = false
title = 'Dastardly database hiding'
+++

I've been recently going through the first part of one of Redgate's excellent free ebooks [The Red Gate Guide to SQL Server Team-based Development](https://www.simple-talk.com/books/sql-books/the-red-gate-guide-to-sql-server-team-based-development/) and was rather amused when I came across the section talking about creating what they term *shape* tables. Essentially what this means is to use the shape ascii codes in order to name a table. I remember a very very long time ago when I was still (almost) in short trousers reading an article suggesting that user passwords should incorporate various invisible ascii codes (such as escape codes) in their respective passwords, since this would render dictionary password cracking tools pretty much useless. It would also help if someone was looking over your shoulder – yes they have just observed you type *MyP@ssw0rd* but they didn't quite catch that ascii code you also typed did they?!

Anyway, all this got me thinking (yes sometimes this does happen) specifically I started wondering *if databases could be named using ascii/extended ascii for those invisible codes?*

![Invisible man](/images/2010/invisible_man1.jpg)

For those of you who are not aware of how to type out an ascii code it is fairly simple. All you need to do is hold down the ALT key whilst typing out the numeric decimal code and releasing the ALT key. Depending upon what the code and the text editor you are in, the code may or may not cause a character to be input. For instance typing ALT+032 will cause a normal empty space to be input to your editor.

After much trial, error and disappointment I was not having much success with invisible codes although I did find one immediate surprise -namely that you can call databases with no names! I'm sure I must have read this somewhere but my conscious self does not recall it. One interesting thing about this, is that irrespective of how many spaces you put between your squared brackets it is always interpreted as the same name so essentially you can only create one database using spaces (or ascii 032).

But much toying later I finally I had my eureka moment! I found that invisible ascii code 255 does generate an invisible character which isn't interpreted as an empty space. Even more interesting was that unlike ascii 032, multiple codes were being taken as a unique database name meaning that multiple databases could be created each with an apparent blank name – a good one for scaring your junior dbas :).

So is there any use for doing this? Actually I can think of a couple but would be interested in any of your suggestions. I may at some stage follow this post up to go through a few of them.

![Create and drop databases](/images/2010/magical-blank-databases.png)

I eventually decided to return back to the subject of user passwords. I again wondered if it was possible to use my magic ascii code of 255 when creating a SQL login. Yet again this took and the "empty space" was in fact being treated as a real character. In other words I couldn't substitute it for a SPACE. This is a very good thing because it means SQL login passwords can made be even more complex be using a combination of this code and real spaces. Actually I have a suspicion that you could probably use ascii 255 for naming any SQL Server object such as tables, columns, functions etc. or even use it in your data. I'd be very interested to here about any tests you perform so please get in touch if you do.

I can envisage several really interesting security uses of using this technique of using the hidden character within your database name and may at some point test these scenarios out, but I would be irresponsible if I didn't warn you to be careful. Firstly I do not know of the official stance on using this ascii code for object naming and it is just possible that future versions may break the way it currently works, although I think this is unlikely. Secondly because this code is invisible you might get into a situation where you cannot remember the sequence yourself particularly in the case of an old legacy database – although I cant currently think of a situation where you would not be able to recover from this but maybe further testing using certain scenarios and SQL technologies may reveal something and of course if I do find something…I'll let you know!