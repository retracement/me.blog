+++
date = '2013-03-20T17:19:00+01:00'
draft = false
title = 'Upload files to your Azure VM instance'
categories = ['Technology']
tags = ['Azure','AzureVM']
+++

Sometimes some of the simplest tips are the best, and recently I was trying to figure out just how to upload SQL Server 2012 Enterprise/ Developer Edition installation files to my Windows Azure Virtual Machine since my pre-provisioned copy of SQL Server (Evaluation Edition) had expired.

As it turned out I realized pretty quickly that I didn’t need to bother since the VM also comes with a copy of SQL Server 2012 Evaluation Edition media pre-loaded at the root of the *C:\\* drive. All I had to do therefore was to grab an existing Developer Edition license key and run the Evaluation Edition media setup and step through the Edition Upgrade to *convert* my existing instance.

However, I also knew that there were other utilities and tools (and possibly other installation media) that I would want to bring into my Azure VM and it all felt slightly restrictive. After checking out the [Windows Azure Management Dashboard](https://manage.windowsazure.com) I could not find any options whatsoever to upload files to my VM.

![Manage Console](/images/2013/manage_console.jpg)

Even after a little digging around in the usual search engines, the most useful suggestion I managed to locate was to download and install an FTP server onto the VM itself and play around a little with port settings! Whilst feasible, that level of tinkering was surely ridiculous. After a few moments of frustration I had an idea. Since I was connecting to my Windows Azure VM over RDP (and knowing that my RDP client is able to provide remote access to certain local resources such as the clipboard I decided to try taking the easy route and simply copy and paste an executable after enabling this option. To do so, you simply have to:-

- Launch your RDP client (`mstsc.exe` in Windows) and expand the details button to reveal the remote computer access to local resources checkboxes.
- Ensure that the Clipboard option is checked and connect and log into your Azure VM.
- Copy the files, folders or ISO from on your local host.
- Paste into your Azure VM.

![Access local resources](/images/2013/remote.jpg)<br/>
*Remote Desktop can provide access to local resources!*

Whilst I have seen this fail in certain restrictive network configurations, to my delight *SUCCESS*. I know this operation is hardly rocket science, but sometimes the most simplest things are the hardest to figure out. Now my Azure VMs suddenly seem to be far more useful and I can’t wait to play further!