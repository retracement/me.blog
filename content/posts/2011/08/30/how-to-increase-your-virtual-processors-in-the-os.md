+++
date = '2011-08-30T22:34:00+01:00'
draft = false
title = 'How to increase your OS Virtual Processors'
categories = ['Technology']
tags = ['Windows','VirtualBox']
+++

There usually comes a time in your Virtual Machine life-cycle that you think... *I wish I’d given that baby more processors for me to play with!* This is especially true if the Virtual Machine in question is your little plaything that you use to test out funky stuff using parallelism and allows you to play with various SQL Server settings and functionality such as *Processor Affinity* and *MAXDOP*.

So there was a time in the dark and distant past when wanting to increase your processor count for your OS would mean a reinstall. Thankfully, this is no longer necessary and once you have performed this operation once you will never forget. So here goes...

The first thing we shall do is look at exactly how many processors are activily in use by the Operating System, and as you will see in this screenshot there are two (see the *CPU Usage History* section).

![Task Manager](/images/2011/taskman1.jpg)

The next thing I will do is run `msconfig.exe` (Start/ Run/ msconfig.exe) and click the Boot tab. Now click the *Advanced options...* button and from the *BOOT Advanced Options* dialog, click the dropdown list just underneath the tickbox for *Number of processors*. As you can see it is currently set to *2* and there isn’t any scope for changing it right now, so come out of this screen and close msconfig. Next shutdown your Virtual Machine.

![msconfig](/images/2011/msconfig1.jpg)

From your Virtual Machine console for your Virtualization platform, bump up the number of virtual processors you wish to allocate and when you are ready, start your machine back up. As you can see, I’ve given my bad boy another 2 extra processors 🙂

![Increase virtual processors](/images/2011/increase-processors.jpg)

Once your Virtual Machine has loaded backup up, the next step is to run msconfig.exe again and go back to the *BOOT Advanced Options* dialog again. If you click on the same dropdown as before you will be surprised to find that the number of processors available is still *2*! Now dont panic, all you need to do is to untick the *Number of Processors* tickbox and to check the *Detect HAL* checkbox and click the *OK* button for each prompt (there should be two). You will now get a message asking if you would like to restart to apply the changes or to exit without restarting. Click the *Restart* button and wait for the Virtual Machine to reboot.

Now take a look at the task manager and you will see those extra beauties all staring back at you. Pretty simple huh?

![Task Manager](/images/2011/taskman2.jpg)