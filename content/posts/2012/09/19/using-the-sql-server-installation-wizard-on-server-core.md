+++
date = '2012-09-19T18:47:00+01:00'
draft = false
title = 'Using the SQL Server Installation Wizard on Server Core!'
categories = ['Technology']
tags = ['SQL','Windows','Automation','Installation']
+++

Ever heard the phrase *"just because you can, doesn’t mean you should?"*. Yep me too -many times, and what I am about to say comes with that caveat in mind although as I shall explain, I see absolutely no reason why you shouldn’t -certainly not in a development environment OR using this technique to produce a configuration installation file. But I’m getting ahead of myself, first let me describe our scenario.

With SQL server 2012, Windows Server Core became a supported option (on versions 2008 R2 and above) and whilst not every single component of SQL Server is supported on Core, there is now a very strong argument to move to that flavour of the Windows Operating System should your scenario warrant it. In fact for the last 12 months or so and in particular when I am presenting my *Enter the Dragon: SQL 2012 on Server Core* presentation, I always like to try and make a point to the audience that if they do not yet know what Server Core is, then they very much will do in 2 years time. Love it or loathe it, in my opinion, they **will** be using it by then. I predict a massive shift to what I like to call *"Skinny"* (or *"Low Fat"*) Windows as opposed to *"Full Fat"* Windows.

For those of you reading this article who are not entirely familiar with exactly **what** Windows Server Core is, I shall describe it simply as *"a reduced graphical interface version of Windows"*. Essentially what this means to you as an IT Professional or DBA is, that certain applications may or may not work on Core -especially if they use a Windows based GUI. This does not mean that all GUI based apps will not work, just the majority. Simple GUI driven applications such as `Notepad.exe` or `Taskman.exe` have full, or in the case of Taskman -partial support. There are a couple of technical reasons why many GUI driven applications many not work on Server Core, but this is outside the scope of this post and perhaps I shall write about that another time.

We move nicely onto the subject of SQL Server and its related technologies and functionalities that it provides -which are supported on core. By and large there is a number to choose from so running SQL Server on Core is a very real proposition. Essentially, if you are thinking of moving to Skinny Windows, then I suggest you take a look at the MSDN page [Install SQL Server 2012 from the Command Prompt](https://msdn.microsoft.com/en-us/library/hh231669.aspx) that lists the supported SQL Server features on Server Core.

By the time you are ready to install SQL Server on your Server Core Operating system, you will probably realize very early on that you have a little bit of a problem. Attempting to launch `setup.exe` will result in a fairly self explanatory error message.

![Server Core Error](/images/2012/servercoreerror.png)<br/>
*Oh flip!*

In fact I am sure you probably won’t have ever got to that error message because being the type of person I know you probably are, you will have read all the necessary literature and have noticed in the MSDN article [Install SQL Server 2012 on Server Core](https://msdn.microsoft.com/en-us/library/hh231669.aspx) it explicitly states:
> SQL Server 2012 does not support setup by using the installation wizard on the Server Core operating system. When installing on Server Core, SQL Server Setup supports full quiet mode by using the /Q parameter, or Quiet Simple mode by using the /QS parameter“.

There is a rather interesting parameter that you may see mentioned in the error message in the previous screen shot called **UIMODE**. Essentially if we pass in the switch and parameter of `/UIMODE=EnableUIOnServerCore` to `setup.exe`, this time we DO get the Installation Wizard. But (and this is a big BUT) none of the installation options will work.

![Doesn't work](/images/2012/dont-work.png)<br/>
*Who needs working installation options anyway?*

By now you are probably starting to curse, and wondering what is the point of that option then? Looking back at the error you will also see the hint about using the `/QS` (or `/QUIETSIMPLE`) switch which shows progress but does not allow input. So if you try to use `/UIMODE=EnableUIOnServerCore` to bypass the Server Core restriction and specify `/QUIETSIMPLE` to show progress you will surprisingly receive an error message stating that these options are incompatible with each other!

Ironically we can simply specify the `QUIETSIMPLE` parameter without `UIMODE` set and now see the installation progress but only for unattended installs using a configuration file. For example:

```bash
setup.exe /IACCEPTSQLSERVERLICENSETERMS /CONFIGURATIONFILE=”ConfigurationFile.ini” /QUIETSIMPLE
```

![Install with quietsimple](/images/2012/installing-core-with-quietsimple.png)

We will return to the `UIMODE` shortly but it is worth highlighting that we are still no closer to using the SQL Server Installation Wizard on Server Core! We have proved that the GUI does seem to work (albeit in unattended mode) and you will be wondering why *if* the GUI appears to work without error on Server Core in unattended mode, doesn’t it work in Wizard based attended mode. The truth of the matter is that the SQL installer is blocking itself on the Server Core OS and I assume Microsoft are gently trying to nudge you away from this type of deployment and that way of thinking on Server Core.

Now what if I told you that you can?

I first stumbled upon this bug (for want of a better word) whilst heavily involved in a SQL Server 2012 writing project earlier in the year -it was loosely related to another problem and bug caused through the very same issue. For more details on the other issue you can read my [Connect 713497](https://connect.microsoft.com/SQLServer/feedback/details/713497/qs-command-line-parameter-does-not-override-configuration-file-quietsimple). If you are performing a command line based installation of SQL Server (and Core will force you through the command line), there are essentially two ways in which you can configure the SQL Server installation. The first is through parameters passed via `setup.exe` (as we have seen above) and the second is through the use of a configuration file -the location of which must be passed through the `/CONFIGURATIONFILE` parameter from the command line.

Now consider the situation where we have one parameter passed through the command line and one configuration file setting for the same parameter, both with different inputs. The expected behaviour is for the command line parameter to override any configuration file setting, and this is exactly how it works and is very useful for what I refer to as *"templated deployments"*. Specifically, you template standard configuration settings in configuration files and override them where your deployment necessitates so that your builds only deviate depending upon their requirements.

![Ladybird](/images/2012/lady.png)<br/>
*some bugs can be useful*

In the aforementioned connect item, you will notice that I discovered the bug in which specifying the short-cut equivalent of *QUIETSIMPLE* (QS) on the command line and providing an alternative configuration file setting using the long syntax, the short-cut does not override the long syntax in the configuration file. In contrast, the short-cut equivalent of *QUIET* (Q) **DOES** (correctly) override the long syntax in the configuration file.

One behaviour that I did not mention when I raised that connect item was that by using the `/QS` (command line) and `QUIETSIMPLE` (configuration file) setting is that we can confuse `setup.exe` to believe we have specified a unattended installation through the `/QS` switch. Because it fails to override the QUIETSIMPLE setting (we have set it to *False* in the configuration file) -we manage to fool the SQL installation, skip the block for Core and enable the Wizard deployment through that configuration file setting.

Cool eh?!

![Wizard Installation on core](/images/2012/wizard-based-installation-on-core.png)

Now I have to confess that through my connect item you will note that it has been marked as fixed in the next Service Pack... I might have single handily caused the demise of the most useful SQL Server 2012 bug (for Server Core deployments) by raising that connect 🙂...Just call me badassss!

But have no fear. Whilst writing this post and performing a little testing I came across another bypass. Whilst I demonstrated earlier the relative uselessness of the *UIMODE* switch, and it is worth pointing out that according to [Connect 706799](https://connect.microsoft.com/SQLServer/feedback/details/706799/rc0-core-install-with-enableuionservercore-not-functional) it seems the switch is not even a supported feature, we can actually use it to perform a similar piece of magic as before. If you remember earlier, when we used the switch and parameter of `/UIMODE=EnableUIOnServerCore`, we did not need to specify an unattended installation switch such as `/QUIETSIMPLE` or `/QUIET`. Unfortunately, when we performed this operation we essentially entered the rather useless setup screen. **HOWEVER** all we need to do is provide a switch to a configuration file ensuring that within it, we have set `QUIET=”False”` and `QUIETSIMPLE=”False”`. By doing so you will also skip the installation block and viola enter a fully functional SQL Server Installation Wizard on Server Core!

Enjoy.

p.s. Remember that Server Core does not support every single SQL Server feature, so it is important that you only select the supported features as documented by MSDN, otherwise the installation is likely to fail.