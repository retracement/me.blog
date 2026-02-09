+++
date = '2015-03-30T07:52:00+01:00'
draft = false
title = 'How to install SharePoint 2013 on Windows 2012R2'
categories = ['Technology']
tags = ['SharePoint','Installation','Windows']
+++

*If you want to jump straight to the condensed solution, then please feel free to go to the [Condensed Installation instructions](#condensed-installation) below. If you find this post useful or have any feedback then please leave me a comment so that I know my efforts were not wasted!*

Installing Microsoft Backoffice products on Microsoft Operating Systems has over the years become a much easier proposition and is largely the reason why SQL Server got the reputation of being “easy”. Except it isn’t. Ask any idiot to install a SQL Server instance and create a database and there is a good chance that they will succeed, but ask them to install SQL Server correctly and configured to best practice, suddenly the knowledge gap becomes a little more obvious.

I recently had a requirement to deploy SharePoint 2013 and at very least expected a relatively easy installation. Working along the lines of current is probably best, I decided that Windows 2012 R2 was probably the right choice of OS going forward. Famous last words.

# The Problem with SharePoint 2013 on Windows 2012 R2

Prior to SharePoint 2013 SP1, Windows 2012 R2 is not supported and I suppose it shouldn’t surprise anyone since SharePoint 2013 came onto the market before Windows 2012 R2. Anyhow, I gathered that I could install SharePoint 2013 onto Windows 2012 R2 and then patch up to SharePoint 2013 SP1. No big deal right?

Wrong.

*Before I start delving into the problem and solution, it is important for me to highlight that it is possible to download a Microsoft provided slip-streamed iso of SharePoint 2013 with SP1 assuming you have access to a lovely MSDN subscription. Using this media would avoid the problems documented within this post.*

However there will be many of you who do not have access to a MSDN subscription and may be using Microsoft Evaluation Editions for your test lab. Quite (un)helpfully Microsoft do not provide a slip-streamed Evaluation Edition of SharePoint 2013 -and as far as I know never provide slip-streamed Evaluation Edition images apart from exceptional circumstances. At this point you may ask yourself *“Why not create my own slip-streamed install?”* but unfortunately this would put you in an unsupported state due to package changes as mentioned in the blog post [SharePoint 2013, SP1 and Slipstreaming](https://blogs.msdn.com/b/ronalg/archive/2014/02/27/sharepoint-2013-sp1-and-slipstreaming.aspx).

*So let’s get back to business…*

The SharePoint media provides the ever so useful pre-requisite installer to help automate the configuration and installation of (you guessed it) pre-requisites prior to installing SharePoint. The problem with this approach is that on Window 2012 R2 the pre-requisite installer fails resulting in the error *“There was an error during installation. The tool was unable to install Application Server Role, Web Server (IIS) Role”* (the MSDN iso of SharePoint 2013 SP1 succeeds).

![pre-requisite installer error](/images/2015/prereq_error.gif)

After a little bit of playing around, it became obvious that things were not going to be plain sailing and so I resorted to Google-Fu expecting to find an authoritative and straight-forward set of instructions from either Microsoft themselves or a SharePoint MVP/ community SharePoint expert. Sadly what I found was a whole melting pot of solutions -some of which appeared to be slightly more complicated than I suspected they needed to be. For instance many suggested enabling a whole long list of Windows Features through PowerShell, even though we can see quite clearly in the pre-requisite check a fairly simple list. Furthermore there was not one single set of instructions I found that took into account the latest releases of SQL Server or Microsoft AppFabric Cumulative Updates.

I decided to have a go myself and see if I could simplify this process and arrive at what I hope will be the definitive solution, so that you (hopefully) won’t have to waste your time like I did!

# Defining our list of pre-requisites

Probably your first port of call is to take a look at the [Microsoft Hardware and software requirements for SharePoint 2013](https://technet.microsoft.com/en-GB/library/cc262485.aspx) which will give you a useful starting point -at least for minimum software requirements, but we can see from the pre-requisite installer feedback that in Windows 2012 R2, Microsoft .NET Framework 4.5 and Windows Management Framework 3.0 is already installed by default. The problem with this pre-requisite checker is that it is intended as an automated way to install the pre-requisite components rather than providing feedback for a manual install. After further executions after enabling a couple of Windows Features I decided that it didn’t really suit my purposes and instead decided to run the SharePoint setup executable hoping that it would provide a simpler way to get feedback.

Thankfully setup immediately reports back on progress and provides those missing requirements.

![setup missing pre-reqs](/images/2015/setup_missing_prereqs.webp)

# Installing the pre-requisites

The first thing I do is to enable those easy Windows-Features through PowerShell. I notice the Windows Identity Foundation requirement (you may find various posts alluding to the fact that this is a download but it is in fact a Windows Feature in 2012 R2).

```powershell
Get-WindowsFeature Windows-Identity-Foundation|
Add-WindowsFeature -source:D:\sources\sxs
```

I will deal with the Identity Foundation extensions in a moment but for now the second feature of interest is Application Server and this is simple enough to add.

```powershell
Get-WindowsFeature Application-Server|
Add-WindowsFeature -source:D:\sources\sxs
```

Next we can add the IIS Web Server (better known as the Web-Server feature); I include all management tools since I find the ability to troubleshoot and configure locally particularly valuable when problems are hit.

```powershell
Get-WindowsFeatureWeb-Server|
Add-WindowsFeature -source:D:\sources\sxs `
-IncludeManagementTools
```

There is also a requirement for the IIS 6 Management Compatibility component and this can be found through the `Web-Mgmt-Compat` feature:

```powershell
Get-WindowsFeature Web-Mgmt-Compat|
Add-WindowsFeature -source:D:\sources\sxs
```

I did initially think that the WCF Data Services 5.0 requirement equated to the `Net-WCF-Services` feature, but I was wrong. It is instead an executable that we need to download and we shall cover these downloads next.

I originally tried the most recent WCF Data Services download (version 5.6.0) but *found that the SharePoint installer surprisingly did not detect it*. After running the installer again I opted for *Uninstall* and instead downloaded version 5.0.512 which can be found at http://www.microsoft.com/en-gb/download/details.aspx?id=29306 and installed it without incident. This time, setup no longer reported this component missing.

You will recall we enabled the Windows Identity Foundation feature, but were still missing the Identity Extensions (1.0). After fumbling around for a little while expecting to find it as a sub-component Windows Feature to enable, I realized that it was a download as reported by several articles. The extensions can be downloaded from http://go.microsoft.com/fwlink/?LinkID=252368 -irritatingly I could only find a direct link! Installing this component was painless and now the requirement was removed from the list of missing components.

The next (and probably the most frustrating and problematic) component I installed was Windows Server Appfabric which can be downloaded via http://www.microsoft.com/en-gb/download/details.aspx?id=27115 and attempting GUI based execution caused me a few issues.

![AppFabric MS Update](/images/2015/appfab.gif)

I first checked that I didn’t want to use Microsoft Update since my environment was disconnected from the Internet. Next I selected all the AppFabric features (which meant checking *Caching Services and Cache Administration*) and completed the installation.

![AppFabric Features](/images/2015/appfabfeatures.gif)

Probably the must annoying aspect of the AppFabric installer is that it can leave your installation in a bit of a pickle. First off, if you cancel the installation and try and rerun SharePoint setup you will almost certainly receive a warning about Microsoft Update being required to run this tool (a crazy situation if you consider the dialog screen earlier), and the second point to raise is that even if you do perform a GUI based installation of AppFabric you will realize upon rerunning the SharePoint setup that it errors with the message *“Windows Server Appfabric is not correctly configured. You should uninstall Windows Server Appfabric and reinstall it using the SharePoint Products Preparation Tool.”* -obviously in our situation we cannot do this!

To get around this conundrum you will first have to uninstall this failed installation of AppFabric. I confess I had a torrid time performing this having first used the command line */r* switch but now could not get the SharePoint setup error to disappear. Eventually I noticed that the AppFabric installation was still registered after looking through *Control Panel/ Programs/ Programs and Features* but the GUI based removal kept erroring with message about the .NET Framework 3.5! To resolve this latest snag, I re-installed AppFabric before heading back to the Programs and Features app to remove it again! A fudge but it worked.

Once the SharePoint setup only complains that AppFabric is missing (and not that it is incorrectly configured as described above), you can perform the AppFabric installation for SharePoint correctly through the command line:

```shell
WindowsServerAppFabricSetup_x64.exe /i CacheClient,CachingService,CacheAdmin /gac
```

I should highlight that I have seen various references indicating that you must use quotes around the parameter options but this has not been my experience so far. Once installed (you will not receive a notification of completion!), reboot your server once more.

AppFabric must be patched to CU1, but I decided to try applying 1.1 CU5. I should have known better since the installer really must have CU1 installed and you can obtain this from http://support.microsoft.com/en-us/kb/2671763 and installation can be performed through the GUI.

The next component to address is the Sync Framework Runtime v1.0 SP1 and you can download this from http://www.microsoft.com/en-gb/download/details.aspx?id=17616 and install with no problems. Obviously in my case I chose *SyncSetup_en_x64.zip* for my installation. Extract and execute the *synchronization.msi* installer from the Microsoft Sync Framework directory.

For some reason I could only find the Microsoft Information Protection and Control Client as a download link, simply download from http://go.microsoft.com/fwlink/p/?LinkID=219568 and install. Like the Appfabric GUI I chose not to enable Microsoft Update.

![Update](/images/2015/mpicc.gif)

A restart is required after installing this component.

The final component to add is the SQL Server Native Client. The pre-requisite states Microsoft SQL Server 2008 R2 SP1 Native Client, but SQL Server 2012 Native Client appears to work too. Note that SQL Server 2014 does not have its own Native Client and instead installs 2012 Native Client. As mentioned earlier, my personal preference is to always deploy client management tools for troubleshooting so I would tend to always deploy the Native Client from full SQL Server 2014 installation media so that I can include those tools (such as SQL Server Management Studio). If you don’t have this requirement then you can download and install the SQL Server 2012 Native Client either as part of the [SQL Server 2012 Feature pack](https://www.microsoft.com/en-gb/download/details.aspx?id=29065) or directly from  http://go.microsoft.com/fwlink/?LinkID=239648 and installation is of course fairly trivial.

By this stage you should have met all pre-requisites and while a reboot is not necessary prior to installation, I advise doing so for one last time before running SharePoint 2013 setup -which you will be happy to see now prompts you for a license key! 🙂

Once SharePoint 2013 installation has completed do not forget to install SharePoint 2013 Service Pack 1.

---

# Condensed installation

*This section is intended as a quick and concise way for you to run through all the pre-requisites before attempting to install SharePoint 2013. If you run into any problems or want a more detailed explanation you should read the entire post described above.*

1. Enable the Windows-Identity-Foundation feature:

   ```powershell
   Get-WindowsFeature Windows-Identity-Foundation|
   Add-WindowsFeature -source:D:\sources\sxs
   ```

2. Enable the Application-Server role:

   ```powershell
   Get-WindowsFeature Application-Server|
   Add-WindowsFeature -source:D:\sources\sxs
   ```

3. Enable the Web-Server role and management tools:

   ```powershell
   Get-WindowsFeatureWeb-Server|
   Add-WindowsFeature -source:D:\sources\sxs `
   -IncludeManagementTools
   ```

4. Enable the IIS 6 Management Compatibility component:

   ```powershell
   Get-WindowsFeature Web-Mgmt-Compat|
   Add-WindowsFeature -source:D:\sources\sxs
   ```
5. Download and install WCF Data Services (5.0.512) from http://www.microsoft.com/en-gb/download/details.aspx?id=29306

6. Download and install Windows Identity Extensions (1.0) from http://go.microsoft.com/fwlink/?LinkID=252368

7. Download and install Windows Server AppFabric (1.1) from http://www.microsoft.com/en-gb/download/details.aspx?id=27115 through the command line:

   ```shell
   WindowsServerAppFabricSetup_x64.exe /i CacheClient,CachingService,CacheAdmin /gac
   ```

8. Reboot Server.

9. Download and install AppFabric 1.1 CU1 from http://support.microsoft.com/en-us/kb/2671763

10. Download and install Sync Framework Runtime v1.0 SP1 from http://www.microsoft.com/en-gb/download/details.aspx?id=17616 (extract and execute the synchronization.msi installer from the Microsoft Sync Framework directory).

11. Download and install Microsoft Information Protection and Control Client from http://go.microsoft.com/fwlink/p/?LinkID=219568

12. Reboot Server.

13. Download and install the SQL Server 2012 Native Client from http://go.microsoft.com/fwlink/?LinkID=239648 (minimum requirement is SQL Server 2008 R2 SP1).

14. Reboot Server (for good measure) and SharePoint 2013 installation should work.

15. Post SharePoint 2013 installation do not forget to patch to SharePoint 2013 Service Pack 1.

I hope this post was helpful to you, and if you would like to see more like this or have any useful feedback then I look forward to receiving your comments.