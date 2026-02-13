+++
date = '2015-02-24T14:20:00+01:00'
draft = false
title = 'Changing the boot order of generation 2 Virtual Machines in Systems Center VMM and Hyper-V'
categories = ['Technology']
tags = ['SCVMM','Hyper-V']
+++

On Windows 2012 R2 hosts, Hyper-V introduced the concept of generation 2 Virtual Machines, which provide various [benefits and a set of restrictions](https://technet.microsoft.com/library/dn282285.aspx) to those VMs configured in this way. In most cases the performance improvements and functionality they provide make it desirable to create new Virtual Machines as generation 2 -assuming that you will run them on generation 2 capable hosts which at this time of writing is Windows 2012 R2 Servers! Furthermore you will also be restricted by your guest operating system because Windows Server 2008 R2 and Windows 7 are not supported by generation 2. There are also further considerations to take into account with regards to Linux support by Hyper-V and we shall look at that topic another time.

Benefits of generation 2 Virtual Machines include improvements to boot time support but with this come a few changes. Assuming you are deploying your Virtual Machines through Systems Center 2012 R2 -and realistically this is a given for any Enterprise worth their salt, there are a few changes to be aware of. For situations requiring it, a change to boot time ordering can be achieved easily for generation 1 VMs via the Systems Center Virtual Machine Manager Console. Click a Virtual Machines Properties and navigate to the Hardware Configuration pane. Scroll down to the Advanced grouping (found at the very bottom of the Hardware Configuration) and select the Firmware item.

![Hardware Firmware Boot Order](/images/2015/hardware-firmware-boot-order.jpg)

From the Firmware screen it is quite obvious how to change boot time ordering and helpfully, the CD (/DVD) is the default first option in the list. In other-words it is unlikely that you will ever need to change the default ordering for generation 1 Virtual Machines.

![Generation 2 Boot Ordering](/images/2015/generation-2-boot-ordering.jpg)

With generation 2 Virtual Machines there are a few changes to the Firmware screen. Helpfully the Systems Center Virtual Machine Manager Console provides a breadcrumb [link](https://technet.microsoft.com/library/dn440675.aspx) to documentation describing how to set the boot order for generation 2 virtual machines. HOWEVER reading through this documentation we are presented with the following statement:

>To customize the startup order for a generation 2 virtual machine in System Center 2012 R2, you must use a Windows PowerShell command that specifies the first boot device, rather than an ordered list of boot devices

This is not great news but at least we are provided with the necessary information so that we are able to configure boot ordering through PowerShell. Clearly the use of [`Set-SCVirtualMachine`](https://go.microsoft.com/fwlink/?LinkID=260344), [`Set-SCVMTemplate`](https://technet.microsoft.com/library/jj654252.aspx), or [`Set-SCHardwareProfile`](https://technet.microsoft.com/library/jj647740.aspx) PowerShell commands should be high on our list and perhaps the preferred option for configuration especially in those instances where we are trying to change a Systems Center template or hardware profile – but is PowerShell our only option for Virtual Machine boot ordering?

![boot-hyper-v](/images/2015/boot-hyper-v.jpg)

Well one very quick and easy way to get around this situation is to instead fire up Hyper-V Manager (you can also use this also to connect to remote Hyper-V hosts). After selecting the Settings of your Virtual Machine, scroll to the Hardware group (top left grouping) and select the Firmware item. Now you will be presented with a screen very similar to the generation 1 Firmware pane through Systems Center Virtual Machine Manager Console. Simply select the DVD Drive and move its order up to the top of the list and click OK to save.

Now whenever your Virtual Machine boots, regardless of whether you have opened an interactive connection to the VM through Hyper-V Manager or Systems Center Virtual Machine Manager Console, the boot sequence will boot the DVD first, allowing you a quick and easy way of temporarily changing the order. If you don't catch the DVD boot on the first startup, restart the VM while still connected and wait for the Press a key... sequence. Easy.

One slightly surprising observation I made while looking into this, was that after changing the boot order in this way, I did not see any change to the FirstBootDevice property of the Virtual Machine Object when queried through PowerShell. I need to investigate this fully, but my advice for now is to always remember to revert the boot order back to default setting for the Virtual Machine after re-installation of its Operating System. If you intend upon making the change permanent or make the change to a Systems Center Template or Hardware Profile then instead use the PowerShell cmdlets mentioned earlier.