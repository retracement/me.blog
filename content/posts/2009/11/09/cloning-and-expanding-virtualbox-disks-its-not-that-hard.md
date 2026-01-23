+++
date = '2009-11-09T13:46:00+01:00'
draft = false
title = 'Cloning and Expanding VirtualBox disks in Linux'
categories = ['Technology']
tags = ['Linux']
+++

You always know when someone is very knowledgeable about a subject by the way in which they answer a question. A simple question is always answered with a simple solution, and this is what happened to me recently when I wanted to expand or migrate a virtualbox virtual disk attached to a real physical disk partition so that I could double my capacity.

The problem was that I had built a virtual machine with a 16GB disk and realised pretty quickly that it was not going to be enough, so I posed a question to the Linux gurus "how do I clone a disk in Linux?". Sure enough the reply came "`dd if=source of=destination bb=512`". 

I have to say that I was a little bit taken aback by that response since I thought "AHHHH THATS NICE AND EASY!"

I describe the steps I took below.

First you need to shutdown your machine with the source drive. Then you need to create a destination drive to your required size and attach this to your virtual machine. Next you should obtain a Live Linux CD, I tried Knoppix and booted up.

After opening a command prompt I queried the available hard drives and mounted the hda1 partition to identify that this was indeed the source drive and it was. I then umounted this partition. I then proceeded to copy the source disk to the destination disk by running the aforementioned command:

```bash
dd if=/dev/hda of=/dev/hdb bb=512
```

![Copy drive](/images/2009/copy_drive.jpg)

After a long wait and a completed copy, I then rebooted the machine so that the new disk partition was detected. I then intended on loading up GParted to expand the partition to fill out the entire drive but unfortunately Knoppix is lacking this utility. For simplicitys sake I decided to download Ubuntu Live CD and use that instead. Upon loading up GParted I checked that the disks were exactly what I thought they were.

![Gparted Source](/images/2009/gpartedsource.jpg)

Next I selected my new disk and expanded the new partition to the drive maximum.

![Gparted Destination](/images/2009/gparteddestination.jpg)

![Gparted Expand](/images/2009/gpartedexpand.jpg)

Once finished expanding I then applied all disk operations.

![Gparted Apply](/images/2009/gpartedapply.jpg)

Finally I then shutdown the machine, promoted the newly created disk to primary and removed the old disk (keeping a temporary copy just in case) and finally booted back up. The perfect larger cloned drive boots up fine, and all without resorting to third party software such as Ghost and Partition Magic!

I think the beauty of all this is the fact that this cloning technique obviously doesn't just apply to Linux partitions/ installations or even VirtualBox disks, but could in exactly the same way be applied to physical disks. It also means that you don't need to fork out on expensive cloning software, worry about creating bootable disks and dont even need to be a Linux guru to perform the operation. Simply download an image, burn it to CD, and you are off and running.