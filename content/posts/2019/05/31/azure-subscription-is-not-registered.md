+++
date = '2019-05-31T20:18:00+01:00'
draft = false
title = 'Azure subscription is not registered to use Cosmos DB namespace'
+++

It is usually the simplest things that often leave me feeling like I am a complete dummy. One such issue I ran into fairly recently when trying to deploy Cosmos DB into an Azure subscription.

From Azure DevOps I received the following error:


> 2019-05-31T16:28:55.4288261Z ##[error]MissingSubscriptionRegistration : The subscription is not registered to use namespace 'Microsoft.DocumentDB'. See https://aka.ms/rps-not-found for how to register subscriptions.
> 2019-05-31T16:28:55.5005526Z ##[section]Finishing: Deploy Cosmos Account and Database with Shared Capacity


I initially assumed that the error was Azure DevOps related so I attempted to deploy using PowerShell and ran into an almost identical error.

I had deployed this Cosmos DB template successfully many times in our other subscriptions and could not understand why a simple deployment to an alternative subscription would fail. Looking back at the error message I followed the link provided which took me to a Microsoft doc titled [Troubleshoot common Azure deployment errors with Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/common-deployment-errors) and linked within I ended up on [Resolve errors for resource provider registration](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/error-register-resource-provider).

It turns out that the Azure resource providers, which you can almost think of as Class libraries, can (like their programmatic counterparts) be in either *Registered* or *NotRegistered* state. When they are in a *NotRegistered* state, this means that we are unable to call that provider to create a specific resource (such as Cosmos DB in my case).

We can use PowerShell Az or the Azure CLI (both talked about elsewhere in this blog) to report what resources are available. In this specific example, I am going to return all providers that match the wildcard pattern of `Microsoft.D*`. The code searches for and sets the relevant subscription using [Azure Cloud Shell](https://azure.microsoft.com/en-gb/features/cloud-shell/) (for simplicities sake), but you can do this through a remote Azure CLI or PowerShell Az session if you would prefer (and connectivity allows).

```powershell
$subscription = Get-AzSubscription | Where-Object Name -Match "MySubscription1*"
Select-AzSubscription $subscription 
$providers = Get-AzResourceProvider -listavailable |Select-Object ProviderNamespace, RegistrationState
$providers  | Where-Object ProviderNamespace -Match "Microsoft.D*"
```

We get the following results:

```
ProviderNamespace               RegistrationState
-----------------               -----------------
Microsoft.DBforPostgreSQL       Registered
Microsoft.DevTestLab            Registered
Microsoft.Databricks            Registered
Microsoft.DataLakeStore         Registered
Microsoft.DataLakeAnalytics     Registered
Microsoft.DBforMySQL            Registered
Microsoft.DevSpaces             Registered
Microsoft.Devices               Registered
Microsoft.DataFactory           Registered
Microsoft.DataBox               NotRegistered
Microsoft.DataBoxEdge           NotRegistered
Microsoft.DataCatalog           NotRegistered
Microsoft.DataMigration         NotRegistered
Microsoft.DataShare             NotRegistered
Microsoft.DBforMariaDB          NotRegistered
Microsoft.DeploymentManager     NotRegistered
Microsoft.DesktopVirtualization NotRegistered
Microsoft.DevOps                NotRegistered
Microsoft.DigitalTwins          NotRegistered
Microsoft.DocumentDB            NotRegistered
Microsoft.DomainRegistration    NotRegistered
```

Notice that the Microsoft.DocumentDB namespace is disabled. If you are wondering, DocumentDB was the precursor name of the Cosmos DB SQL API (before Cosmos DB supported multiple APIs). Like many other Microsoft products, early names tend to stick with the products 🙂.

To register this namespace we can simply run the following line of code against the subscription using the [Register-AzResourceProvider](https://docs.microsoft.com/en-us/powershell/module/az.resources/register-azresourceprovider?view=azps-3.6.1) cmdlet.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.DocumentDB
```

The following output is returned:

```
ProviderNamespace : Microsoft.DocumentDB
RegistrationState : Registering
ResourceTypes     : {databaseAccounts, databaseAccountNames, operations, operationResults...}
Locations         : {Australia Central, Australia East, Australia Southeast, Canada Central...}
```

If it is not obvious you would unregister a provider namespace (if you wanted to make it unavailable) using the [Unregister-AzResourceProvider](https://docs.microsoft.com/en-us/powershell/module/az.resources/unregister-azresourceprovider?view=azps-3.6.1) cmdlet as follows:

```powershell
UnRegister-AzResourceProvider -ProviderNamespace Microsoft.DocumentDB
```

Once I had registered the Microsoft.DocumentDB namespace, I was able to deploy my Cosmos DB template into my subscription without error!

---

# Summary

Depending upon your subscription and region, your enabled provider namespaces may vary, however in my case someone had explicitly un-registered `Microsoft.DocumentDB` from it. You might ask why someone might do that? Well, it is a good way to prevent deployments of certain resource types if they go against your company policy.

As you can see, if you run into a similar problem or want to start using resource types that are by default `NotRegistered` you can register and start using them incredibly easily.

