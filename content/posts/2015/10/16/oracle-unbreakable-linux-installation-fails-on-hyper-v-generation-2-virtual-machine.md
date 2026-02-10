+++
date = '2015-10-16T14:10:00+01:00'
draft = false
title = 'Oracle Unbreakable Linux installation fails on Hyper-V Generation 2 Virtual Machine'
categories = ['Technology']
tags = ['Hyper-V','Oracle','Installation','Windows']
+++

Have you been attempting (and failing) to install [Oracle Unbreakable Linux](https://www.oracle.com/us/technologies/linux/overview/index.html) as a Virtual Machine under Hyper-V and cannot figure out what is wrong?

# The problem
If you are receiving the following message:

```text
Boot Failed. EFI SCSI Device.
Boot Failed. EFI SCSI Device.
Boot Failed. EFI SCSI Device. Failed Secure Boot Verification.
PXE Network Boot using IPv4
......
PXE-E18: Server response timeout.
Boot Failed. EFI Network.
No Operating System was Loaded.
Press a key to retry the boot sequence...
```

![Secure Boot verification](/images/2015/secure-boot-verification.jpg)

Then I bet you are using a Hyper-V generation 2 VM?!

When I first saw the error message it is quite clear that something is not working properly with the boot sequence and my first thought was that either my install media was corrupt or that there was an incompatibility between the Oracle Linux boot media and Hyper-V generation 2 virtual machines (the error message gives a big clue on this).

If you would prefer to attempt the original resolution detailed in this post and skip the update approach listed below click [here](#the-resolution).

---

<mark>**Update 11th October 2016**</mark>

Before continuing with this article, you should know that there is a slightly better way to address this problem, since you are also very likely to run into the same (or similar) problem on CentOS, Ubuntu, or anything else that is not Microsoft Windows in Hyper-V. For instance, I have noticed that on an attempted CentOS installation within a version 8 generation 2 Hyper-V VM (Windows 10 Anniversary Update Edition)  there is not even a visual clue given that there is a Secure Boot failure -at least not on the initial boot (as before).

All we see is:

![centos_no_boot](/images/2015/centos_no_boot.png)

However, if you wait long enough you will eventually arrive at the following screen:

![no-uefi](/images/2015/no-uefi.png)

So rather than disable *Secure Boot* as this blog post instructs, I recommend changing Secure Boot to use the *Microsoft UEFI Certificate Authority* template rather than the *Microsoft Windows* template. Make this change through the Virtual Machine *Settings* page (*Security* node in the hardware pane) and if it is not already obvious, your VM must be stopped in order to change this setting.  Once you have made the change, your problems should be resolved and your Linux distribution should automatically boot.

![secure_boot](/images/2015/secure_boot.png)

If you do not see this option available to you, then feel free to proceed with the alternative route as described [below](#the-resolution).

<mark>**End of update**</mark>

---

# The resolution
Immediately I tried installing to a generation 1 VM and it ran through smoothly without incident (proving media was fine), so I returned back to my generation 2 VM to resolve the issue. *Returning back to the error message, the Failed Secure Boot Verification warning stands out like a sore thumb*, and Hyper-V afficionados will recognise that secure boot was actually introduced by generation 2 VMs. Thankfully it is very easy to turn this off and disable secure boot through the *System Center/ Firmware* option within *Virtual Machine Properties* pages (or via *Hyper-V Manager/ Settings/ Firmware* option). Alternatively, we can also do this through PowerShell as follows:

```powershell
Set-VMFirmware –VMName "VMname" -EnableSecureBoot Off
```

The next time the machine boots, the installer should automatically launch (and if it isn’t clear already, you must leave secure boot disabled post install).

![installing](/images/2015/installing.jpg)

For more information on this subject see [Oracle Linux virtual machines on Hyper-V](https://technet.microsoft.com/en-GB/library/dn609828.aspx) and visit [What’s New in Hyper-V for Windows Server 2012 R2](https://technet.microsoft.com/en-gb/library/dn282278.aspx) for and in-depth discussion of Hyper-V new features in Windows 2012 R2.