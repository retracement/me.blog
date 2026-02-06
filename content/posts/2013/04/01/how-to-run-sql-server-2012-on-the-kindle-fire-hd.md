+++
date = '2013-04-01T12:44:00+01:00'
draft = false
title = 'How to run SQL Server 2012 on the Kindle Fire HD'
categories = ['Technology']
tags = ['Funny','SQL']
+++

<mark>Warning! Please do not attempt to perform any of the following steps on production systems. This information is provided purely for reference and any attempt to replicate these actions on your own hardware will be purely at your own risk!</mark>

One of the biggest benefits to the latest release of SQL Server (and reasons to upgrade) is that Windows Server Core is now a supported option. For those of you who are not yet familiar with Server Core, it could be best described as a reduced GUI version of Microsoft Windows. In other words, if you like the command line then you are in for a very big treat! Regular visitors to these pages will most likely be very aware of my passion for Windows Server Core and more to the point running SQL Server on it.

I digress slightly, but the point is that because SQL Server can now run on within a reduced graphical interface environment, one large beneficial side effect to this is the portability of the database engine. More specifically we can “hack” SQL Server to run on more and more types of hardware and software. The following is a closely guarded secret by [Microsoft](https://www.microsoft.com/en-us/default.aspx), but with with a few simple hacks, re-compilations and know how, we can spoof SQL Server to run on several other Operating Systems.

![SQL on Kindle](/images/2013/screen1.jpg)<br/>
*Let’s get to this!!!*

Now is probably a good time to stress that whilst I will reveal some of the magic involved in fooling SQL to run elsewhere other than Windows, I am currently bound into a Non-Disclosure Agreement with Microsoft and will therefore be careful not to break it. Therefore any information that I have compiled within these pages are public domain and can be found from various sources elsewhere. It is also important to stress that as of this time of writing, only the SQL Server features supported by Windows Server Core can be “ported” across to other environments -so if you are wanting to use Master Data Services elsewhere other than “full-fat” Windows, then unfortunately you are out of luck.

For a list of all the SQL Server features currently supported on Server Core you should visit [Install SQL Server 2012 on Server Core](https://msdn.microsoft.com/en-gb/library/hh231669.aspx).

Several months ago I came into possession of a lovely new Kindle Fire HD and as you may know, this tablet runs a customized version of Google’s Android Operating System. Having had some recent (but limited) success with running SQL Server on some other Android based Hardware I decided to try my luck on this nice bit of kit, and the following is how I did it.

# Install Server Core

Probably one of the most time consuming parts of this whole exercise is the installation of the Server Core Operating System. This can be especially bewildering if you have never done so before, but a very good source of information is the MSDN article [Install and Deploy Windows Server 2012](https://technet.microsoft.com/en-gb/library/hh831620.aspx). One reason I chose Windows 2012 as opposed to Windows 2008R2 was simply to eliminate any possible backwards compatibility issues with Android when we come to port SQL Server. For instance there is a well known bug in [Android 1.6 Donut](https://developer.android.com/about/versions/android-1.6.html) where a port of SQL Server from Windows 2008 will cause erratic buffer pool memory corruption.

# Install SQL Server onto Server Core

Installation of SQL Server onto Server Core is relatively straight forward (see the previous link provided) as long as you are comfortable with performing a command line installation -either through using a configuration file or a full  set of command line options. Unfortunately (and perhaps understandibly) GUI based installation will not work, however I did identify a “bug” which allows a workaround which I detailed in the following post [Using the SQL Server Installation Wizard on Server Core!](https://tenbulls.co.uk/2012/09/19/using-the-sql-server-installation-wizard-on-server-core/).

# Dump the SQL Server Process to disk

There are lots of ways to dump a process working set (the memory a process is consuming) to disk. On Server Core, it is no more difficult to do so since the Task Manager is (partially) supported and works as you would expect. You simply need to locate the SQL Server Service process, right click it and select the Create Dump File option.

![Dump Process](/images/2013/dump_process.jpg)

Once this is successful you will see a message telling you where the dump file has been created. One very important consideration for you is to make sure that you have set a fairly low [maximum memory](https://msdn.microsoft.com/en-gb/library/ms178067.aspx) cap for SQL Server’s buffer pool otherwise it the dump file will be much larger (and take longer to output) in addition it will make your ported SQL Server application to run faster.

![Dumping process](/images/2013/dumped-process.jpg)

# Rename the dump file to .src

A fairly simple but necessary step to perform is the renaming of the dump file that we have just created. The reason for doing so is so that we can recompile the assembly in the next step and target our assembly for Operating System of choice. Simply right click the dump file and rename its extension to `.src`.

# Recompile the source file using the .NET Native Image Generator

The Native Image Generator tool ([ngen.exe](https://msdn.microsoft.com/en-GB/library/6t9t5wcf(v=vs.110).aspx)) is an often overlooked and ignored utility for the .NET Framework but is very useful for improving the performance of our .NET applications and avoiding JIT compilation. There are several undocumented options for ngen and we shall use them to port our application. Since SQL Server is not a full .NET application and only uses parts of the .NET Framework (for optimization reasons) we can use the undocumented `/partial` switch which will recompile all necessary IL code sections of the dump file and will translate all other machine code sections into one complete assembly. Since we are also moving to a different Operating System we must also use the undocumented `/architecture:ux64` switch to target a 64 bit Android environment.

![ngen.exe](/images/2013/ngen.jpg)

Once we have created our native Android image of SQL Server there are only a few final steps remaining!

# Install FSExplorer on your Kindle

In order to copy our SQL Server Assembly across to our Kindle Fire HD, navigate to the root and create a new folder we need a decent file explorer application. Probably the best I have come across on the Android Operating System is a nice little free app called FSExplorer. Open the Kindle App Store, search for and install FSExplorer.

Move the SQL Server image to your Kindle
Using FSExplorer, navigate to the file system root and create a folder called *SQLSrvr* where you will stored the Android SQL Server assembly. Copy the recompiled SQL Server image to your Kindle under this folder.

Set SQL Server to auto-load
In order to auto-load SQL Server as a daemon process you should first create an auto loader file under the root calling it startp (if one does not already exist). Make sure the file has the execute permission assigned and within it enter one simple line start `/SQLSrvr/sqlsrvr.d`.

![Create folder](/images/2013/create_file_and_folder.jpg)<br/>
*Don’t forget to edit the startp file and copy the SQL assembly!*

After you have made this last change, reboot your Kindle Fire HD and SQL Server should launch successfully.

Enjoy!