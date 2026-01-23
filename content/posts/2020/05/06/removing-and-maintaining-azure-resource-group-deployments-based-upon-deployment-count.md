+++
date = '2020-05-06T11:37:47+01:00'
draft = false
title = 'Removing and Maintaining Azure Resource Group Deployments Based Upon Deployment Count'
+++
Whenever you create or update an Azure resource, a new *deployment* is created under the resources' configured resource group. This deployment history is retained ad-infinitum until you eventually hit the hard limit of *800 deployments (per resource group)*. You may think this figure is more than enough to accommodate all the possible resource changes that could ever be made in a resource group, but if you are running CICD pipelines to push out your Infrastructure as Code (IaC) (or create lots of resources per resource group) then it is very likely you will exhaust this figure very quickly.

Every time a release pipeline runs, regardless of whether you are changing resources or not, all configured and enabled deployments in the pipeline will result in a new deployment record. You can view all historic deployments in the Azure Portal for each resource group by selecting its Deployments item under the Settings pane (see below).

![Cosmos Deployments](/images/2020/cosmos-deployments.webp)

In the example above you will note that we only have 4 deployments that have been created in this resource group. When the hard limit is eventually hit, all subsequent deployments to that specific resource group will fail.

# Microsoft's solution
Microsoft provide a solution to this in the MS doc titled [Resolve error when deployment count exceeds 800](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-quota-exceeded) which allows you to programmatically remove deployments (through Azure CLI or PowerShell Az) based upon a deployment date and this is made possible because of the Timestamp property. I have also seen many blog posts that simply seem to regurgitate this Microsoft code giving really just one solution – to maintain deployments based on date.

This is all well and good if your deployments span many weeks or months and that the counts are predictable, based on date-time, but what happens if you have highly active, highly unpredictable, or high number of resources per resource group?

# Deployment count solution
Perhaps a far better solution would be to maintain a set deployment count that will allow each release to succeed each time. If you are only deploying a single resource, then clearly you would only have to ensure a spare deployment slot is available. If you are deploying resources through a CICD pipeline then you simply need to ensure that you have at least that number of resources in your pipeline available.

# Running from Azure CLI or PowerShell Az command-line
If you are manually running the maintenance code either from a remote command-line session or directly within the Portal command-line itself, you will have to set your subscription context that you wish to maintain. We can do this easily in PowerShell by running the following code (ensuring that you change the subscription text to the one you wish to target):

```powershell
$subscriptionName =  "ACMEPRODSUB"
$subscription = Get-AzSubscription -SubscriptionName $subscriptionName
Set-AzContext -Subscription $subscription | Out-Null
```
Once you have set your subscription you can then use the subsequent code (detailed in the Running from within an Azure DevOps Pipeline section).

# Running from within an Azure DevOps Pipeline
I have generally found that running a maintenance step at the start of any infrastructure Release Pipeline is a good point of execution. It will reduce the time to cycle through and delete any excess deployments to a minimum, and will also ensure there are enough deployment slots to prevent release failure. For our pipelines, maintaining a deployment count of 700 is a good compromise -it leaves 100 spare slots for each run and plenty of past deployment history.

In the release pipeline, we can create an Azure PowerShell task within a release stage.

![Azure PowerShell Task](/images/2020/azure-powershell-task.webp)

For convenience we can use our code as an inline script -though you may ultimately decide to parameterize the $retainCount variable and publish the script from a repo instead.

![Inline Script](/images/2020/script-type.png)

Use your common sense when setting the number of deployments you wish to retain.

```powershell
$retainCount = 700
```

Since our Azure PowerShell task has explicit settings for the subscription that we wish to execute the script against, we do not need to worry about changing subscription context. All we are concerned about is the functionality of the code itself.

In the code below we are first looping through all resource groups in the current subscription context. For each resource group we return all its deployments, and for any deployment that is above the iterator threshold set by $retainCount (assuming there are any) -it will be deleted.

```powershell
foreach ($resourceGroup in Get-AzResourceGroup){
    $resourceGroupName = $resourceGroup.ResourceGroupName
    $deployments = Get-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName
 
    write-host "Resource group" $resourceGroupName "has" $deployments.Count "deployments..."
 
    $iterationCount = 1
    foreach ($deployment in $deployments) { #deployments are returned sorted by age desc
        if ($iterationcount -gt $retainCount){
 
            Remove-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -Name $deployment.DeploymentName | Out-Null
            write-host "   Deleted deployment on" $deployment.Timestamp -fore magenta
        }
 
        $iterationCount = $iterationCount + 1
    }
    Write-Host "   Resource group deployment maintenance complete." -fore green
}
```
This results in the following output:

![Deployment Results](/images/2020/deployment-results.webp)

If you are using an Azure DevOps release task to execute this code you will not see coloured text in the task output.

# Conclusion
If you are manually maintaining your resource group deployments or wish to automate it through Azure DevOps, the timestamped solution provided by Microsoft may not fit your requirements given the frequency of your deployments or other considerations. Given that the deployments are returned in a time sorted descending order, we can easily delete deployments based upon the deployment count -always leaving enough space for future deployments and not removing those based upon date alone. Ensuring that this maintenance task is run prior to any automated infrastructure releases can improve the success rate of your release pipelines in highly active environments.