+++
date = '2018-12-01T21:35:00+01:00'
draft = false
title = 'Installing Azure CLI on Linux Mint'
categories = ['Technology']
tags = ['Azure CLI','PowerShell','Linux','Azure']
+++

The Azure CLI (or Azure Command Line Interface) allows provides an easy way to create and manage your Azure resources on macOS, Linux, and Windows. If you (like me) are using Linux and wish to use and control [Microsoft Azure](https://portal.azure.com) easily through the command line, then it is probably something you should have.

I wanted to write a very quick post in order to explain the very simple steps you must follow to get the Azure CLI working for you if are using an [Ubuntu derivative distribution](https://en.wikipedia.org/wiki/Category:Ubuntu_derivatives) such as [Linux Mint](https://linuxmint.com/). Microsoft’s basic installation guide [Install Azure CLI with apt](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest) has one specific problem for those distros, so let’s take a look at the Ubuntu section of that guide:

![Install Azure CLI](/images/2018/installazurecli.png)

If you run steps 1 to 3 you will not observe a problem at the time of execution but will hit an error on running the first line of step 4. We see the following error:

```text
Ign:13 https://packages.microsoft.com/repos/azure-cli serena/main Translation-en
Reading package lists... Done
W: The repository 'https://packages.microsoft.com/repos/azure-cli serena Release' does not have a Release file.
N: Data from such a repository can't be authenticated and is therefore potentially dangerous to use.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: Failed to fetch https://packages.microsoft.com/repos/azure-cli/dists/serena/main/binary-amd64/Packages  404  Not Found
E: Some index files failed to download. They have been ignored, or old ones used instead.
```

In the highlighted line we can quite clearly see the *404 Not Found* error, and if we take a look inside the `/etc/apt/sources.list.d/azure-cli.list` file (created in step 2), you will see that it contains that repository path generated, which is, of course, the problem.

Linux Mint uses its own release codenames and so the default script (provided by Microsoft) picks this up rather than the (required) Ubuntu release name for the Microsoft software repository. See the `$(lsb_release -cs)` piece of code in their script. So before we first start with the Microsoft code, you will need to find the Mint release name and replace this with the correct Ubuntu package base.

Find out your Linux Mint short codename by running the following:

```bash
lsb_release -cs
```

In my case, I am running Linux Mint Serena. Next, so I now need to find out the short codename of the Ubuntu base build that my edition of Mint is derived from. To do this, visit the [Linux Mint Releases page](https://linuxmint.com/download_all.php).

From this page, I can see that *Serena* uses the *Xenial* package base (as below):

![Linux Mint Releases](/images/2018/mintrelease.png)

All we need to do is add the right repository path for the right package base. There are two ways to do this. The first you can simply edit `/etc/apt/sources.list.d/azure-cli.list` and replace (in my case) `serena` with `xenial` as we have done below.

![add to package base](/images/2018/path.png)

Alternatively, you can edit (and hard code) the variable substitution within script 2 and rerun (this programmatically does the same thing we performed manually above):

```bash
AZ_REPO=xenial
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
    sudo tee /etc/apt/sources.list.d/azure-cli.list
```

And that’s it, once the right package base has been corrected, you can rerun the step 4 script which should no longer error. Azure CLI is now ready for you to manage and deploy your Azure resources from your lovely Linux Ubuntu derivative distribution! Check out [Common Azure CLI commands for managing Azure resources](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/cli-manage) for some guidance on how to use it.

I’ll give it a try and attempt to list all my Azure resource groups in tabular format:

```bash
az group list --output table
```

Which gives me:

```text
Name                                      Location     Status
----------------------------------------  -----------  ---------
cloud-shell-storage-westeurope            westeurope   Succeeded
future_decoded_demo                       eastus2      Succeeded
Gothenburg                                northeurope  Succeeded
mysql                                     northeurope  Succeeded
sql2014sp1                                northeurope  Succeeded
sqlonlinux                                uksouth      Succeeded
stretchgroup-gothenburg-northeurope       northeurope  Succeeded
stretchgroup-hhserverf-sql01-northeurope  northeurope  Succeeded
stretchgroup-techdaysvm-northeurope       northeurope  Succeeded
techdays                                  northeurope  Succeeded
Testing                                   eastus       Succeeded
```

As you can see, Azure CLI is very cool and you should start using it now, so don’t let minor configuration difficulties stop you!