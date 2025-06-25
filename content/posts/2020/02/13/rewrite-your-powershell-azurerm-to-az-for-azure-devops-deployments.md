+++
date = '2020-02-13T14:58:00+01:00'
draft = false
title = 'Rewrite Your Powershell Azurerm to Az for Azure Devops Deployments'
+++

I just wanted to get a quick post out on specific problem that I have just encountered from one of our Azure DevOps release pipelines in which one of the release stage tasks started failing for no reason (despite no obvious changes having been made). Hopefully this post will help others if you are also experiencing this problem and don’t know what to do.

The error message was as follows for our task `Enable Firewall on Data Lake`

```text
2020-02-13T12:30:51.4672549Z ##[section]Starting: Enable Firewall on Data Lake
2020-02-13T12:30:51.4779190Z ==============================================================================
2020-02-13T12:30:51.4779279Z Task : Azure PowerShell
2020-02-13T12:30:51.4779364Z Description : Run a PowerShell script within an Azure environment
2020-02-13T12:30:51.4779425Z Version : 3.153.2
2020-02-13T12:30:51.4779503Z Author : Microsoft Corporation
2020-02-13T12:30:51.4779571Z Help : https://docs.microsoft.com/azure/devops/pipelines/tasks/deploy/azure-powershell
2020-02-13T12:30:51.4779656Z ==============================================================================
2020-02-13T12:30:53.6479788Z ##[command]Import-Module -Name C:\Modules\azurerm_6.7.0\AzureRM\6.7.0\AzureRM.psd1 -Global
2020-02-13T12:31:00.6580029Z ##[command]Clear-AzureRmContext -Scope Process
2020-02-13T12:31:01.3041473Z ##[command]Disable-AzureRmContextAutosave -ErrorAction Stop
2020-02-13T12:31:01.9336559Z ##[command]"D:\a\_tasks\AzurePowerShell_xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\3.153.2\ps_modules\VstsAzureHelpers_\openssl\openssl.exe" pkcs12 -export -in d:\a\_temp\clientcertificate.pem -out d:\a\_temp\clientcertificate.pfx -password file:"d:\a\_temp\clientcertificatepassword.txt"
2020-02-13T12:31:02.1959058Z ##[command]Add-AzureRMAccount -ServicePrincipal -Tenant *** -CertificateThumbprint ****** -ApplicationId *** -Environment AzureCloud @processScope
2020-02-13T12:31:02.6748632Z ##[command] Select-AzureRMSubscription -SubscriptionId xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -TenantId ***
2020-02-13T12:31:03.1996315Z ##[command]&amp; 'd:\a\_temp\xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.ps1'
2020-02-13T12:31:05.7869733Z ##[command]Disconnect-AzureRmAccount -Scope Process -ErrorAction Stop
2020-02-13T12:31:06.1648969Z ##[command]Clear-AzureRmContext -Scope Process -ErrorAction Stop
2020-02-13T12:31:06.9334190Z ##[error]Exception of type 'Microsoft.Rest.Azure.CloudException' was thrown.
2020-02-13T12:31:07.0104724Z ##[section]Finishing: Enable Firewall on Data Lake
```

The task was configured as follows
![Azure PowerShell task](/images/2020/rmtask.webp)

As you can see we are simply trying to execute the following Azure RM command to configure an existing DataLakeStorage v1 resource:

```powershell
Set-AzureRmDataLakeStoreAccount -ResourceGroupName myazureresourcegroup -Name myazuredlresource-AllowAzureIpState Enabled -FirewallState Enabled
```

As several failed attempts to re-run this task with the same error and confirmation that nothing else had changed in our environment I decided to rewrite the now deprecated AzureRm code (I have discussed this elsewhere in this blog) to Azure PowerShell Az module. The command was now as follows:

```powershell
Set-AzDataLakeStoreAccount -Name myazuredlresource -FirewallState Enabled -ResourceGroupName myazureresourcegroup -AllowAzureIpState Enabled
```

Also, in order to use Azure PowerShell Az module we also need to change the task version to 4.* and above.

This time upon release we get success!

# Conclusion

While it is still not clear to me that the failures we were experiencing are as a direct result of AzureRm being slowly phased out (doubtful at this stage), it is clear this particular problem is perhaps a wake up call for us all to slowly start considering converting our existing code to Az in order to future proof it in our pipelines. I will check with Microsoft for feedback on this issue to understand if there is official advise with regards to our legacy AzureRm code and update this post accordingly.

For now, I hope this resolution fixes your problem too!