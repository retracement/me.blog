+++
date = '2019-02-01T13:18:00+01:00'
draft = false
title = 'AzureRM, Azure CLI and the PowerShell Az Module'
categories = ['Technology']
tags = ['Azure','PowerShell','AzureRM','Azure CLI','Linux']
+++

![Azure CLI](/images/2019/azurecli.png)

There is now a variety of Microsoft provided command line tools available to connect to (and manage) Azure Resources and the situation is quite confusing to new-comers or those individuals who have not kept up to date with new developments. This post is designed to rectify this situation.

It is probably also worth me mentioning that I am focusing on the usage and deployment of these tools with Linux in mind, however, the following explanation is also applicable to Windows and macOS environments.

---

# PowerShell Core (on Linux)

If you are running a Linux machine, to use AzureRM or the new PowerShell Az module you will first need to install PowerShell Core for Linux. Installation is fairly easy to perform and you can follow [this post](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6) using the relevant section for your particular distribution and release. In my specific case, I am running Linux Mint 18.1 Serena, however, to get my Ubuntu version I first run the following:

```bash
more /etc/os-release
```

This returns:

```bash
NAME=\"Linux Mint"
VERSION="18.1 (Serena)"
ID=linuxmint
ID_LIKE=ubuntu
PRETTY_NAME="Linux Mint 18.1"
VERSION_ID="18.1"
HOME_URL="http://www.linuxmint.com/"
SUPPORT_URL="http://forums.linuxmint.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/linuxmint/"
VERSION_CODENAME=serena
UBUNTU_CODENAME=xenial
```

As you can see, the *UBUNTU_CODENAME* is *xenial*. So if I visit the [Ubuntu list of releases](https://wiki.ubuntu.com/Releases) page, I can see that xenial equates to version 16.04.x LTS. This means that (for me at least) I can follow the PowerShell on Linux installation section titled [Ubuntu 16.04](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#ubuntu-1604) (instructions provided below simply as an example):

```bash
# Download the Microsoft repository GPG keys
wget -q https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb
# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb
# Update the list of products
sudo apt-get update
# Install PowerShell
sudo apt-get install -y powershell
```

To enter a PowerShell session in Linux, you simply type the following within a bash prompt (in Windows you instead use the powershell executable):

```bash
pwsh
```
![PowerShell on Linux](/images/2019/pwsh-1.png)

# AzureRM module (aka Azure PowerShell)

*Before you read further, you should understand that AzureRM is deprecated, and if you are currently using it to manage to Azure resources, then you should seriously consider migrating to one of the other options described in the next sections (and removing AzureRM from your system).*

I first came across the AzureRM PowerShell Module many years ago when I wanted to manage Azure Resources from my Windows laptop through PowerShell. At the time, this was the only way of doing so from the command line, and the functionality of AzureRM was provided by (in Windows at least) a downloadable installer to make the module available to PowerShell. You can check out the latest version and instructions for AzureRM by visiting [here](https://docs.microsoft.com/en-us/powershell/azure/azurerm/install-azurerm-ps?view=azurermps-6.13.0), but as mentioned earlier, you should avoid attempting this and instead use the Azure CLI or PowerShell Az module as described in the next sections. At the last time of trying, attempts to install AzureRM via PowerShell Core in Linux resulted in failure with warning messages pointing to the PowerShell Az module, so you are forced to go to the newer options anyway.

Upon import into your PowerShell environment, the AzureRM module provided up to 134 commandlets (in version 2) so that you could manage all your Azure subscription resources via PowerShell.

---

# Azure CLI

The Azure CLI was intended as the defacto cross-platform command-line tool for managing Azure resources. Version 1 was originally conceived and written using node.js, and offered the ability to manage Azure resources from Linux and macOS, as well as from Windows (prior to PowerShell Core being available on macOS and Linux). Version 2 of the Azure CLI was re-written in python for better platform compatibility and as such, there is now not always direct one-one compatibility between commands across those versions. Obviously, you should use version 2 where possible.

Head over to Microsoft docs article titled [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) for instructions on installing the CLI for your operating system of choice, or you might also be interested in how to do so on my personal Linux distribution of choice (Linux Mint) in my post titled [Installing Azure CLI on Linux Mint](https://tenbulls.co.uk/2018/12/01/installing-azure-cli-on-linux-mint/). Installing the Azure CLI will enable the ability to use the az command syntax from within your Windows command prompt, or via your PowerShell prompt, or through bash (if you are specifically using Linux).

Once installed, to use the Azure CLI you will prefix any instruction with the `az` command. Thankfully there is strong contextual help so you can simply run `az` for a full list of available subcommands, or provide a specific subcommand for help on that.

To execute an az command, you will first need to login to your Microsoft account (that you use to access your Azure subscription/s) as follows:

```bash
az login
```

This will return the following message:

```text
Note, we have launched a browser for you to login.
For old experience with device code,
use "az login --use-device-code"
```

A browser window will be automatically launched for Azure and require you to log in with your credentials. Alternatively (as you can see in the message), you can use the old authentication method (which is especially useful if your machine does not have a browser). In this case, you would run the following command:

```bash
az login --use-device-code
```

Then log into your account from a browser entering the device code provided.

Either way, after you have done this, we can issue our az commands against our Azure resources, for example:

```bash
az vm list
```

Would list out all current Azure IaaS VMs in your subscription.
Another useful tip with az is to use the *find* subcommand to search through command documentation for a specific phrase. For example if I want to search for Cosmos DB related commands I would use the following:

```bash
az find --search-query cosmos
```

Returns the following list of commands:

```text
`az cosmosdb database show`
    Shows an Azure Cosmos DB database
 
`az cosmosdb database create`
    Creates an Azure Cosmos DB database
 
`az cosmosdb database delete`
    Deletes an Azure Cosmos DB database
 
`az cosmosdb collection create`
    Creates an Azure Cosmos DB collection
 
`az cosmosdb collection delete`
    Deletes an Azure Cosmos DB collection
 
`az cosmosdb collection update`
    Updates an Azure Cosmos DB collection
 
`az cosmosdb database`
    Manage Azure Cosmos DB databases.
 
`az cosmosdb collection`
    Manage Azure Cosmos DB collections.
 
`az cosmosdb delete`
    Deletes an Azure Cosmos DB database account.
 
`az cosmosdb update`
    Update an Azure Cosmos DB database account.
```

# Az PowerShell module

You might have already questioned that if PowerShell already had a way to manage Azure resources (through the AzureRM module) and we now have the Azure CLI (providing the cross-platform Az functionality), how could there be any synergy between those two environments?

The answer is that there isn't. This is one reason why AzureRM is deprecated.

With the arrival of PowerShell Core on Linux and macOS, it became possible to import the AzureRM module also into those environments, and yet as we have already found out, the Azure CLI was the newer mechanism to manage Azure resources. The problem is that both used different commands to do so (however subtle). In December 2018, Microsoft addressed this situation by introducing the new PowerShell Az module to replace AzureRM which gave a level of synergy between managing Azure resources through the Azure CLI and managing resources through PowerShell. This really means that if you understand one command line environment, your scripts will be relatively easily transferable into the other.

If you are looking to migrate from AzureRM to the new Azure Az module then you should check out this excellent [post by Martin Brandl](https://about-azure.com/2018/12/21/how-to-migrate-azure-powershell-from-azurerm-to-the-new-az-module/). It is also worth you checking out this announcement from Microsoft titled [Introducing the new Azure PowerShell Az module](https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az).

To install the PowerShell Az module you can follow the Microsoft article titled [Install the Azure PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps). As you will read, you must install this module in an elevated prompt, otherwise, the installation will fail. On PowerShell Core for Linux this means that from bash you would first elevate your PowerShell session as follows:

```bash
sudo pwsh
```

Then you can run the following PowerShell Install-Module command:

```bash
Install-Module -Name Az -AllowClobber
```

Once installed and imported, you are now able to utilize this new PowerShell Az module but must first login to your Azure account using the following PowerShell command:

```bash
Connect-AzAccount
```

This will return a message similar to the one below:

```text
WARNING: To sign in, use a web browser to open
the page https://microsoft.com/devicelogin and
enter the code D8ZYJCM2V to authenticate.
```

Once you have performed this action, your Az module in PowerShell will be ready to use. For instance, we can now list IaaS VMs in PowerShell as follows:

```powershell
Get-AzVM
```

As you may note, there is a correlation between this command and the Azure CLI command (`az vm list`) that we ran earlier. However, it is important to realize that the functionality and behavior of the PowerShell Az module and Azure CLI are not identical. For instance, in this specific case, the Azure CLI will return a verbose result set in YAML format by default whereas the PowerShell Az module returns the results in a shortened tabular format.

---

# Summary

Hopefully, it is clear by now that the AzureRM module is redundant across all environments and if you need the ability to manage resources in Azure through PowerShell then the Az module is what you should be using. However, given the ease of multi-platform deployment and use of the Azure CLI, it is probably something you should not ignore and arguably might prefer over PowerShell Az (or at very least run both alongside each other). For example, at the time of writing, exporting an Azure resource template to PowerShell results in code that uses AzureRM rather than PowerShell Az whereas exporting to CLI uses (of course) the Azure CLI itself.

There is also an argument that the Azure CLI is far better suited to Azure automated deployments over ARM templates due to its brevity, and this is discussed in detail by Pascal Naber in his excellent post titled [Stop using ARM templates! Use the Azure CLI instead](https://pascalnaber.wordpress.com/2018/11/11/stop-using-arm-templates-use-the-azure-cli-instead/).

Whether you eventually decide to favor Azure CLI over PowerShell Az (or use both), I sincerely hope this post has helped clear up any confusion you may have between all the available command line options to manage Azure resources -I know it had confused me!