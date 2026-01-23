+++
date = '2023-06-18T20:25:00Z'
draft = false
title = 'Automating Pause of Synapse Dedicated Pool Part I'
+++
Synapse Analytics is an enterprise analytics engine that offers multiple (compute) Pool types that you can use for varying data processing and analytics requirements. Each Pool type has a slightly different cost model based upon its architecture, capabilities, and consumption patterns, but if we over simplify things, we can break these cost considerations down to compute costs and (in some cases) storage costs.

One of the biggest benefits with modern data lakehouse architectures is that with the separation of compute and storage comes the ability to reduce costs when compute is not needed, and most importantly, not running.

Fortunately, of the Synapse Pool types, Spark Pools and Data Explorer Pools both come with built-in mechanisms for auto-pausing their compute clusters when they are not in use (after a timeout period).

![Spark pause settings](/images/spark-pause-settings.png)
*Figure 1 - Spark pool pause settings*

This capability is neither earth shatteringly new nor unexpected, and something that Databricks has provided for some time. Of the two Data Exploration & Data Warehousing Pool types, Synapse Serverless Pool (otherwise know as the built-in Pool) by its very definition does not incur compute charges when it is not running.

Therefore this leaves us with only dedicated SQL Pool to worry about and this is where our problems begin.

![Dedicated Pool no pause capability](/images/dedicatednopause.png)
*Figure 2 - Dedicated Pool no pause capability*

If you implement dedicated SQL Pool in your organisation, it probably won't take you too long to realise that you do not have any automatic pause settings as your CEO starts to complain about Azure costs and you seek to optimise your environment. Bewilderingly perhaps, Microsoft have neglected to offer us this capability despite all the other Pool types providing cost effective compute options when not in use. Fortunately Microsoft provide a manual pause option allowing us to write our own pause and resume functionality, and we can turn to PowerShell Az or the Azure CLI to write some of this code.

The problem should be broken down into two specific concerns and the first (and arguably most important) is to write the suspend and resume scripts. For each we will parameterise so that the script will be reusable for multiple instances of synapse across multiple subscriptions.

```powershell
[CmdletBinding()]
param (
    [Parameter(Mandatory=$false)]
    [String]
    $synapseWorkspaceName,
    [Parameter(Mandatory=$false)]
    [String]
    $subscriptionName
)
```

Depending upon how you orchestrate your scripts, it is possible that subscriptionName will not be needed. For example, with an Azure DevOps deployment, the subscription is already specified by the executing task and your runtime will have this subscription selected. However if you are orchestrating from somewhere else and you have multiple subscriptions in your Azure tenant you will need to explicitly find and select that subscription context to successfully find your resources (as shown next).

```powershell
# Get subscription and select if needed
$subscription = Get-AzSubscription|where {$_.Name -eq $subscriptionName}

# Subscription selection not needed for ADO since subscription is already in context
$subscription|Select-AzSubscription
```
The next thing you need to do is to search for the Synapse dedicated Pool. If one is not found then you can gracefully exit the routine. It is probably worth pointing out that the assumption here in code is that there will be just one dedicated pool created per workspace (clearly not always the case) and if your environment has several then you will need to add a pool name parameter and apply this to the where clause to restrict this search.

```powershell
# Search for existing synapse pool
$sqlPool = Get-AzSynapseSqlPool -WorkspaceName $synapseWorkspaceName
 
# Exit routine if does not exist
if(!$sqlPool){
    Write-Host 'SQL Pool not found!'
    exit
}
 
#Output current state
$sqlPool
 
$sqlPoolName = $sqlPool.Name
$sqlPoolStatus = $sqlPool.Status
```
All of the code so far will be the same in both the suspend and resume scripts.

Now that the sqlPool object is defined, let's now write the core part of the suspend script. If the sqlPool is already paused or pausing we should not do anything else. The following script checks that the pool is not in either of these two states and assuming that is the case will suspend the pool.

```powershell
if($sqlPoolStatus -eq '"Paused"' -Or $sqlPoolStatus -eq '"Pausing"'){
    Write-Host 'SQL Pool is paused or pausing state already.'    
}
else{
    Write-Host 'Pausing SQL Pool'
    Suspend-AzSynapseSqlPool -WorkspaceName $synapseWorkspaceName -Name $sqlPoolName
}
```
Coding the resume is almost identical, but instead checks that the pool is not in online or resuming state and attempts to resume the pool.

```powershell
if($sqlPoolStatus -eq '"Online"' -Or $sqlPoolStatus -eq '"Resuming"'){
    Write-Host 'SQL Pool is started or resuming already.'
}
else{
    Write-Host 'Resuming SQL Pool'
    Resume-AzSynapseSqlPool -WorkspaceName $synapseWorkspaceName -Name $sqlPoolName
}
```
If we put all this together we now have our Synapse.Suspend.ps1 and Synapse.Resume.ps1 scripts, and you can automate these using a range of different options. Personally I like to do this kind of thing in Azure DevOps if I quickly want to get something working and scheduled quickly, and in another post we will cover how to do this.