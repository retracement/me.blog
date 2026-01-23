+++
date = '2011-02-17T19:38:00+01:00'
draft = false
title = 'Clustering nodes on different subnets'
categories = ['Technology']
tags = ['Clustering','HADR']
+++

It's rather ironic that the answer to a question that came my way recently, had been sitting in my feed-reader for the last month. Apart from the length of time that the item/s had been sitting there, I had opened them on numerous occasions and re-marked as *unread* because I believed they needed more than just a quick skim.

I consider myself very experienced with SQL Clustering but one cannot know everything about everything. I'm endeavouring to fill any gaps in order to be not just very good, but hopefully at some point a leading (or how about *the leading* expert in this field. Right now there are only a few people that I am aware of that I consider to be authorities in SQL Clustering -at least people I can learn from, I've even been using it as long as some of them. There will of course be more experts who I am unaware of but the ones I follow are Geoff Hiten, Allan Hirt and Symon Perriman of course! Whether I achieve my own personal aims will depend upon how many other areas of specialism I lend my eye to, but one can only try and right now I think I've been heading in the right direction.

![Not this kind of cluster](/images/2011/chocolate-cluster.jpg)<br/>
*NO these are not the kind of clusters we are talkin' about!*

Anyway, with respect to the question I am rather pleased it got asked, because it focused my attention to the two articles sitting in my feed-reader and although I may have digested them at some point I wonder whether the details would have stuck. Now those details have relevance to me and I doubt I shall ever forget. The question is "Can cluster nodes reside on different subnets?" -and this question was basically asked in the context of talking about Geo-Clusters. My answer was *no they can't, the nodes of a cluster must reside on the same subnet and to get around this problem on a Geo-Cluster a V-LAN needs to be created*. For now, lets not worry about Geo-Clusters for a minute to avoid overcomplicating things, so before you continue onto the links what do you think the answer is and why?

[Multi-subnet clusters? post 1](https://blogs.msdn.com/b/clustering/archive/2011/01/05/10112055.aspx)
[Multi-subnet clusters? post 2](https://blogs.msdn.com/b/clustering/archive/2011/01/19/10117423.aspx)

And now you've checked, tell me.... *did you get it right?*