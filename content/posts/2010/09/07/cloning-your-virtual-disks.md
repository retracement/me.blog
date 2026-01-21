+++
date = '2010-09-07T14:23:00+01:00'
draft = false
title = 'Cloning your VirtualBox virtual disks'
+++

The cloning of disk images (especially Windows) can present a few problems from the OS perspective which I shall expand upon shortly, however depending upon the Virtualization solution that you are using there can be a few problems to overcome unique to that solution.

# Get a new GUID
VirtualBox uses a GUIDs in several places to uniquely identify various things. I'm not really sure I like this, and haven't really thought about why it is required, but it could have something to do with the multi-platform support of VirtualBox and its VMs.

Not only is the Virtual Machine itself identified by a GUID, but the Virtual Disks they use do too. Where the Virtual Machine's GUID can be altered fairly easily in an XML configuration file, there is a hard-coded GUID in the Virtual Disks. This presents a problem when you copy (or clone) the Virtual Disk, but there seems to be a couple of ways around it.

The first is to make a file copy of the Virtual Disk whilst it is obviously not in use. Then you can use the VBoxManage internalcommands command to set a new GUID on your copied disk as thus:

```bash
VBoxManage internalcommands sethduuid newimagename.vdi
```

![Set disk UUID](/images/2010/setguid1.png)

The second slightly more involved way is to use the VBoxManage clonevdi command to copy a disk image and auto set a new GUID on this copy. The slight quirk is that it appears that in order for you to copy an existing disk, your source disk needs to be released and deleted from the VirtualBox Media Manager. Obviously if you need to do this, ensure that when presented with the option to remove the physical media from the disk, you select the option no leave the physical media on disk! Once you have copied the source disk, you can then add it back into the Media Manager and attach back into its respective VM. The other quirk is that it appears as though you need to close your VirtualBox GUI interface for the clone to complete -but this could just be me being impatient and jumping to conclusions. The clonevdi syntax is as follows:

```bash
VBoxManage  clonevdi sourcedisk destinationdisk
```

![Clone disk](/images/2010/clonevdi1.png)

If a destination path is not specified then your cloned disk will end up in the VirtualBox default disk location, which on Linux systems is usually `~/.VirtualBox/HardDisks.`

Once your clone is completed you can then add back in your disks to the Media Manager.

# Ask SID
The problem with Windows disk duplication is that internally to the OS, every machine has its own Security Identifier known as the Machine SID. In the past it has been customary to change the Machine SID since there has always been a believe that this was required in order to avoid problems. I personally have come across a few, and from recollection I think I have experienced issues with duplicate Machine SIDs when using SMS 2.0 and clustering in the past. Even though I think in most cases having duplicate SIDs would not cause an issue, it is something that I would still insist on for peace of mind and in case MSS help is required.

Anyway the following [link](https://blogs.technet.com/b/markrussinovich/archive/2009/11/03/3291024.aspx) written by Mark Russinovich provides more information on how to change the Machine SID and whether it is still really required. Interestingly my old favorite tool NewSid is no more, although sysprep does the job (more on this perhaps another time).