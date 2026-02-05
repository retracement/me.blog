+++
date = '2012-03-14T12:43:00+01:00'
draft = false
title = 'How to remove your VirtualBox machines BLOAT'
categories = ['Technology']
tags = ['VirtualBox','Storage','Utilities']
+++

Well I guess its been a long time since I have offered you a useful nugget of information or help, so I have decided to haul myself to the keyboard and provide a useful tip that I have been saving for a rainy day. Regular readers to this blog will certainly be no stranger to my positively strong feelings towards Oracle’s VirtualBox and I have been a user of this product for more years than I can remember. The tip I am about to describe will use VirtualBox in the examples but it should be transferable to any virtualisation product that has the ability to shrink back virtual disks.

Of course in the world of most virtualisation products there is a concept of *dynamically expanding* virtual disks and *fixed-size* virtual disks, and VirtualBox is no different having had this ability for many years. There was always the question of whether to choose the dynamic disk to save real disk space or go for the fixed-size for performance gains. In actual fact this argument is a little bit of a red herring these days (more perhaps on this another time) and the reality is that unless you have a virtual disk which is frequently having to allocate space and expand (in the same way database or page files can) you are always better going for the dynamically expanding virtual disks.

Using a dynamic disk provides the ability to have a virtual disk that is only the size required for the files (and some overhead) that it contains. But what happens should your virtual machine disk allocate lots of space to allow for that 10GB SQL Server Database that you were playing around with (and then dropped)? The biggest problem with a dynamic disk is that it slowly creeps up in physical size over time in very much the same way that transaction logs can do in SQL Server if not truncated regularly by taking frequent log backups.

In VirtualBox and competing products, there is the ability to shrink back these dynamic disks to reclaim space back to your host machine. For those of you who have tried doing so will already know that it is usually a hit and miss affair and leaves you scratching your head wondering why your virtual disk shrink has only given back 1GB of a possible 10GB of free space. One of the problems with this type of operation is that the shrink operation will only release space that it knows is free and this is not always a clear cut thing if the disk block has at one time had writes. The Operating System knows they are reusable but the virtualisation shrink may not.

Lets take a quick look at the space consumption of my virtual machine’s virtual drive sizes…

![Before shrinking](/images/2012/before-shrinking.png)<br/>
*wonko is a bit big*

So the largest virtual drive belongs to a virtual machine I call wonko and it stands out to me straight away as being an oddity because it (like two other virtual machines) is running on Windows Server Core but is around twice the size of the others. After I take a look inside Wonko, it becomes very apparent to me why it is so large. When I reinstalled Windows Server Core on this VM, I forgot to reformat the hard drive. The installer therefore decided to rename the old Windows directory for safe keeping... Oh did I mention that I did this operation twice since my first re-install was wrong!

After completely removing the two offending archived Windows directories I take another look at the virtual disk space consumption...

![A lot of free space](/images/2012/space-in-wonko.jpg)<br/>
*My goodness what a lot of free space!*

Ok so this is now where the magic happens…

There is a rather fabulous tool from those wonderful Sysinternals chaps called sdelete which will help you mark all your free space as clean. This makes it easier for the virtualisation shrink operation to see that the space can be reclaimed back and the virtual disk can be shrunk.

Running a `sdelete /?` from the command line gives us the following list of options:

```text
SDelete – Secure Delete v1.6
Copyright (C) 1999-2010 Mark Russinovich
Sysinternals – http://www.sysinternals.com

usage: sdelete [-p passes] [-s] [-q] …
       sdelete [-p passes] [-z|-c] [drive letter] …
   -a         Remove Read-Only attribute
   -c         Clean free space
   -p passes  Specifies number of overwrite passes (default is 1)
   -q         Don’t print errors (Quiet)
   -s or -r   Recurse subdirectories
   -z         Zero free space (good for virtual disk optimization)
   ```

And the option we want is pretty obvious isn’t it? Yes let’s choose the `z` parameter...
![sdelete](/images/2012/sdelete.jpg)

Right, now our virtual disk’s free space is now zeroed out, it is time for us to run our shrink operation. In VirtualBox this can be performed by running *VBoxManage modifyhdd –compact* and doing so takes several minutes to run through (on my SSD).

Let us take one final look at my host drive and in particular see the size of the wonko virtual disk…

![After shrink](/images/2012/after-shrink.png)
*Wow!*

I think the results are quite evident and staggering. The wonko disk is now 18GB or so lighter. Time to load up my SSD with lots more VMs!!!