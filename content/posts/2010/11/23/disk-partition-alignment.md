+++
date = '2010-11-23T14:29:00+01:00'
draft = false
title = 'Understanding disk partition alignment'
+++

I'm sure by now most of you will have heard about Disk Partition Alignment and why it is so important (and not just for use with SQL Server IO Subsystems). I'm really quite interested in how many companies have actually sat up and smelt the coffee and addressed this issue? This problem is something that DBA's quite readily acknowledge is a potential area of concern, but we usually have to rely on our SAN and Server administrators being one step ahead of the game in actively addressing the situation.

Who do you think you are kidding!

I've experienced first hand on several occasions different DBA's flagging this issue as serious to those guys only to experience blank or puzzled looks or (at worst) a lack of desire to address the issue.

![Don't talk about it](/images/2010/sssh.jpg)

The reality is that Disk Partition Alignment sounds nuts. Right? It is really one of those problems that you really wouldn't expect to find in modern day computing and the work around sounds almost like it is an April fools joke. The lack of uptake in addressing this problem is probably the biggest reason why this issue has now been addressed in Windows 2008 automatically, but unfortunately the story will not end there.

Since a partition may be misaligned in previous versions of Windows at partition creation time, the problem will not be resolved until it is destroyed and recreated. This could be quite a painful exercise of moving data back and forth from old to new partitions and in those cases of very large full partitions may not actually be technically feasible until a secondary storage array has been obtained (and business downtime arranged) to make this operation possible. Another issue to bear in mind when you are recreating new partitions (after destroying the old) within Windows 2008 is that the Partition Alignment *may* have already been addressed by an on the ball SAN administrator at the SAN level. This will obviously mean that …yes you have guessed it…your new Windows 2008 partition will then be misaligned! This is something to watch out for.

Anyway without further ado, here are a couple of useful urls for a more detailed look at this subject. This first url is quite simply excellent and should tell you everything you need to know.

[Disk Partition Alignment Best Practices for SQL Server](https://msdn.microsoft.com/en-us/library/dd758814%28SQL.100%29.aspx)

The next two urls are supplementary:

[A Description of the Diskpart Command-Line Utility](https://support.microsoft.com/kb/300415)
[Disk Partition Alignment (SANs and Diskpart )](http://sqlblogcasts.com/blogs/grumpyolddba/archive/2008/04/03/disk-partition-alignment-sans-and-diskpart.aspx)

In conclusion, the next time your company is putting in a new Disk Subsystem whether it be SAN or even DAS, don't forget to bring up the subject of partition alignment with your administrators and ensure that you all address this problem from the beginning.

_**Update (24th November 2010):**
I've asked to clarify a couple of things in this post a little bit by Mladen Prajdic – the inventor of the great SSMS tools pack and although I can't promise it will make any more sense to others I shall do my best…here goes…. OK, so the particular area of confusion is the part where I discuss SAN admins addressing the partition alignment at the SAN level. What I mean by this is that it is possible for the SAN admins to do some SAN level magic to offset the partition which will not be visible to Windows. Therefore it is important that you ensure that you speak to them to determine that this has/ hasn't been done, although it would be unlikely since as my post states there seems to be a general apathy by the storage admins to partition alignment. Secondly, Windows NTFS filesystem cluster size should be compatible with the physical array stripe size. In both cases they should be 64Kb. The area of the SQL CAT article that touches on this is the section titled "Essential Correlations: Partition Offset, File Allocation Unit Size, and Stripe Unit Size"._

_Hope this was OK Mladen?_