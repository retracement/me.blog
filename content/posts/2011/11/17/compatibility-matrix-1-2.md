+++
date = '2011-11-17T22:29:00+01:00'
draft = false
title = 'Lock Compatibility Matrix 1.2'
categories = ['Technology']
tags = ['SQL','Concurrency']
+++

A couple of months back I published my version of SQL Server's Lock Compatibility Matrix under the post [Fancy a decent Compatibility Matrix?](/posts/2011/09/27/fancy-a-decent-compatibility-matrix) since the one provided by MSDN is really very poorly designed and difficult for the average Joe to understand.This is one reason I believe that lock compatibility is a very misunderstood subject.

One thing that has bothered me about my version from the very beginning was the use of green and red to respectively indicate compatible or incompatible locks. The reason this has bothered me from the offset is because I was aware that those two colours could be a very big problem for colour blind DBAs. However, for me they really DID look great together and *told the story* of the compatibility between the locks without even needing words -every (non colour blind) person understands the concept of GREEN -> GO (i.e. lock grant), RED -> STOP (i.e. lock wait). BUT that doesn't help colour blind people very much does it!

Well the time has finally come to redress that issue and provide a matrix that is (hopefully) colour blind friendly. I have made the following changes:

1. Lightened the green so that it wasn't so garish and made it slightly easier on the eye.
1. Substituted the garish red for a deep blue. The reason I chose blue was for the *coldness* to (hopefully) still *"tell the story"* through colours.
1. Minor changes to the Legends (for purely indulgent reasons).
1. Changed the legend descriptions for green and red to *compatible* and *incompatible* since I believe they are better than *No Conflict* and *Conflict*.

![Lock compatibility 1.2](/images/2011/lockcompat12.png)

I also have several MAJOR changes to the chart scheduled at some point in the future such as changing the cell sizes to squares -which should overcome the perceived distortion that the eye interprets when the chart is viewed at the wrong sizes. In order to accommodate that change I also need to make better use of the white space and I know how I will do it, but as everything in life it will take some time!

Without further ado, the Compatibility Matrix 1.2 can be downloaded via [here](/documents/compatibility-matrix-1.23-bluegreen.pdf). Enjoy!

p.s. Your comments or requests are very welcome...