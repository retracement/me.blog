+++
date = '2016-10-11T11:47:00+01:00'
draft = false
title = 'Configuring Red Hat Enterprise Linux 7.2 for YUM without a Redhat Subscription'
categories = ['Technology']
tags = ['Installation','Linux']
+++

It has been a very long time since I have installed a [Redhat Enterprise Linux](https://www.redhat.com/en/resources/whats-new-red-hat-enterprise-linux-7) distribution having tended to prefer Ubuntu based distributions (such as Linux Mint) or CentOS if I really wanted a Redhat derivative (although for Oracle Database installations on Linux, I would always tend to use Oracle Enterprise Linux for simplicity). With the development and impending arrival of SQL Server on Linux, I thought it was about time that I returned back to the playground with a vanilla copy of Redhat EL so that I could test it with [SQL Server](https://www.microsoft.com/en-gb/cloud-platform/sql-server-on-linux), [Docker Linux Containers](https://www.docker.com/), [Spark](https://spark.apache.org/) and lots of other sexy things that have been avoiding my immediate attention for a little too long.

After a basic installation, I decided that it was time to start using YUM to add some of my favorite packages to this build when I hit across this quite annoying error:

```bash
sudo yum install nano
```
This resulted in the following error:

> Loaded plugins: product-id, search-disabled-repos, 
subscription-manager
This system is not registered to Red Hat Subscription Management.
You can use subscription-manager to register.
There are no enabled repos.
Run "yum repolist all" to see the repos you have.
You can enable repos with yum-config-manager --enable

Ok so this is obviously not going to fly and I am certainly not going to pay for a Redhat Subscription so I decided to break out Bingoogle and came across this rather useful post from Aziz Saiful called [HowTo Install redhat package with YUM command without RHN](https://saifulaziz.com/2014/02/26/howto-install-redhat-package-with-yum-command-without-rhn/) -and I recommend you also give it a read (although some of its details are ever-so-slightly out of date with this release of RHEL 7.2.). The post discusses how to set up an alternative source to the installation DVD, and for Windows people, this is an equivalent of the *-source* parameter that we would use in PowerShell with the `Add-WindowsFeature` cmdlet to add new features from local media.

To cut a long story short, I decided to work my way through this article and provide an updated post (and if nothing else, I will not need to Bingoogle this again!).

Our first step is to ensure that we have a mounted Redhat Enterprise Linux 7.2 DVD (i.e. the one we installed Linux from).
The next step is to mount the DVD to a mount point. For simplicities sake, I chose cdrom off the root.

```bash
sudo mkdir /cdrom
sudo mount /dev/cdrom /cdrom
```

Ok so now we have a mounted cdrom, we can create the YUM repo configuration file within the path `/etc/yum.repos.d` to point to this location. Unfortunately, you will need to use `vi` to do this (*I hate vi!*), but if you need any tips on vi, please use this [vi Cheat Sheet](https://www.lagmonster.org/docs/vi.html). Once in vi, create the file dvd.repo (or called anything else you want – but ensure you keep the .repo extension otherwise the file will not be recognized by YUM).

```text
[dvd-source]
name=RHEL 7.2 dvd repo
baseurl=file:/cdrom/
enabled=1
gpgcheck=0
```

Once you have created this file, if you have performed every step correctly, you can take a look at YUM's repolists.

```bash
sudo yum repolist
```

And while you still receive the same error regarding the System not being registered to Red Hat Subscription Management, you should also see your new repo listed underneath.

To check it all works, let's install nano!

```bash
sudo yum install nano
```

![yum](/images/2016/yum.webp)

Perfect! Like everything in Linux, it is easy when you know how. On a closing note, it is unclear to me at this moment in time, whether this will entirely resolve my installation problems since I will still obviously need access to an online repo or sources in order to install third-party packages not included with the installation media,  but once I have figured that out, I will update this post.