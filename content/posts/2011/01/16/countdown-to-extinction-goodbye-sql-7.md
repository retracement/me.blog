+++
date = '2011-01-16T20:06:00+01:00'
draft = false
title = 'Goodbye SQL 7'
categories = ['Personal Development']
tags =['PASS Summit','Learning','Public Speaking']
+++

I cannot let the week go by without a special goodbye to an old friend. If you hadn't already heard, [SQL 7.0 support officially ended on the 11th January](https://blogs.msdn.com/b/sqlreleaseservices/archive/2010/09/29/end-of-support-for-sql-server-7-0.aspx?wa=wsignin1.0) of this week. This means that as of now, legacy installations of SQL Server 7 are threatened with extinction and is only a matter of time before they become as scarce as SQL 6.5 installs.

If you are interested in how I got started with SQL Server and you haven't done so already, you might want to take a quick peek at my [about](/about) page. Here I mention how I quite insightfully (read luckily) took a training opportunity and put myself on a SQL 7 beta course rather than the one my company was expecting – one for SQL 6.5. That decision had a very big effect on my future IT career. Whilst at the training centre I remember that there was a guy there who claimed that he had been one of the first 100 people in the world to certify as an MCSE+i and his efforts to do so had been made a little easier because of the fact that he had apparently been based in Saudi Arabia for six months in a company that had not only a testing centre but all course materials available to him. He said of being out there *"well what else can you do for 6 months in Saudi? I thought I might as well do something productive!"*

![SQL7 logo](/images/2011/sql7b_scaled.png)

For those of you that might not remember that far back, the old style MCSE+i was on NT 4 and the primary difference (the +i) meant (if I can remember correctly) that you had to pass three extra exams and these were Implementing and supporting IEAK 4, Administrating Proxy Server 2.0 and Administrating IIS 4.0 -so in short if you have not already guessed the +i stood for Internet. Anyway, the take away from this conversation was that I thought to myself *I can do that*. From that decision to go on that beta course I was now set on a path of certification that would shape my learning and career to this very day. More importantly at no point in my career since the time that I first began using the slightly more awkward SQL Server 6.5, have I been more than a few pings away from a Microsoft SQL database engine. I believe this is testament to how good the SQL 7 product was and how it helped shape future releases, I strongly believe that without SQL 7, Microsoft's database offering would probably now just be MS Access.

![PASS 2010 Welcome](/images/2011/welcome.jpg)

So moving forward in time by around 13 years... *You're simply the best... better than all the rest...better than anyone... anyone I've ever met* and so on. 🙂 During the [SQL PASS 2010](https://www.sqlpass.org/summit/na2010/) keynote speech, after the infamous and most enjoyable Tina Turner impresario, we witnessed a video introduction of what happened to SQL Server in order to get where we are today. I am of course talking about the decision to throw away the entire code-base and start again, performing a complete product rewrite. We saw photos of the development team sprawling fast asleep over sofas after working into the early hours and it was quite insightful to realise exactly how much effort had gone into this version.

![Tina does PASS](/images/2011/tina.jpg)

One of the fundamental changes new to the SQL 7 engine was the change in the storage structures. Several weeks ago on Twitter someone asked why was a page 8KB in size, and I immediately looked back to SQL 7 and thought about the decision of the design team to choose 8KB page sizes instead of the previous 2KB. There must have been a good reason, most likely a performance benefit and not just a capacity one. In a nutshell I replied that I believed they had chosen 8KB pages because this meant extents would be (assuming the multiple of eight) 8 x 8KB = 64KB. So what is so special about 64KB? On the NTFS file-system, 64KB was generally considered as the optimal block/ cluster size for performance and this is particularly important now since it means that every extent allocation would use this optimal size. Kalen Delaney agreed that my assumption was probably correct, but if anyone else has any further information to add I would be very interested to hear it.

There are so many other great [changes](https://technet.microsoft.com/en-us/library/cc966493.aspx) that arrived with SQL 7, it didn't just look good, it was good. Goodbye SQL 7 I'll never forget you xxx.