+++
date = '2010-09-06T16:02:00+01:00'
draft = false
title = 'VirtualBox virtual disk corruption when using ext4 host partitions'
+++

Sometimes it is not always a great idea to be running the latest and greatest, and it seems that there is a small problem when VirtualBox virtual machines have their virtual disks housed on ext4 filesystems. I am sure that this will be corrected at some point, but in the meantime luckily there is a simple workaround.

>When the virtual machine is first started up, and if the error condition is satisfied, you are prompt with the warning message.
>
>The virtual machine execution may run into an error condition as described below. We suggest that you take an appropriate action to avert the error.
>
>The host I/O cache for at least one controller is disabled and the medium '/home/xxxx/server1_disk1.vdi' for this VM is located on an ext4 partition. There is a known Linux kernel bug which can lead to the corruption of the virtual disk image under these conditions.
>
>Either enable the host I/O cache permanently in the VM settings or put the disk image and the snapshot folder onto a different file system.
>
>The host I/O cache will now be enabled for this medium.

If you receive the warning message the best thing to do is to immediately shutdown your VM (your IO cache will have been temporarily set) and locate the host IO cache checkbox.

![Disk cache warning](/images/2010/disk_cache_warning.png)

 To get there, first click the Storage option in the sidebar, then click each and every disk controller that services any disks housed on ext4 partitions. From there it is possible to set the host IO cache permanently.

![Use host I/O cache](/images/2010/diskcache.png)

Perhaps it's time to revert back to ext2 or 3? First I shall look into the benefits of running on ext4,  and if I am not gaining anything from it I proabably will.