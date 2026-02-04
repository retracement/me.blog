+++
date = '2011-10-14T15:59:00+01:00'
draft = false
title = 'NOLOCK hits Mythbusters!'
categories = ['Personal Development']
tags = ['PASS Summit','Public Speaking','Concurrency']
+++

Exactly one month to this very day I wrote the article [When should you use NOLOCK?](/posts/2011/09/14/when-should-you-use-nolock) which explained exactly when the use of the `READUNCOMMITTED` isolation level was acceptable and when it wasn't. I also revealed that it not only takes out a *Schema Stability (Sch-S)* lock on the object (which is documented in Books Online for the hint) but also took out a special *Shared (S) Bulk Operation* lock when used on heaps. This latter behaviour is not even mentioned in Books Online for the hint and as far as I know, has never been documented or mentioned anywhere else before.

I therefore asked Paul Randal to check out my aforementioned blog post to confirm that I wasn't going mad, and thankfully he confirmed that I wasn't and that it was in fact expected behaviour. He also thanked me for giving him the idea for his final myth for his *More DBA Mythbusters(DBA-316-S)* session which he performed yesterday in which he gave an explanation of these behaviours. So I not only encourage you to check out my post, but to get hold of the PASS Summit 2011 DVD set and have a watch!

![NOLOCK hits Mythbusters](/images/2011/nolock_reduced.jpg)<br/>
*My idea hits Mythbusters!*