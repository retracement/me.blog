+++
date = '2020-01-06T20:41:00+01:00'
draft = false
title = 'Using Azure Cli to Query Azure Devops'
+++

In previous posts, I have touched upon the use of Azure Cloud Shell for generic querying of Azure resources and I thought it would be useful to quickly document its use for something a little more specific such as querying or manipulating [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) through the command line.

For my example, I will focus on something as mundane and straight-forward as querying the Azure DevOps repository meta-data (so that I can look at and compare branch settings against each other) but I hope you get the idea that this is just scratching the tip of the iceberg and the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) is a powerful tool to add to your arsenal of scripting languages.

The whole end-end process required to query Azure DevOps is itself is a relatively straight-forward affair -especially when you know exactly what you are doing (isn’t everything!) but before we get there, you will first need to have access to the Azure CLI. You have two ways of using it, the first being to install it locally -and instructions to do this can be found via an earlier post titled "[AzureRM, Azure CLI and the PowerShell Az Module]()". Alternatively, you may also use the Azure CLI through Azure Cloud Shell (i.e. directly from Azure) as detailed in another of my posts titled "[Introduction to Azure Cloud Shell]()".

# Configure az devops pre-requisites

Once you are up and running with the Azure CLI and have access to its az command, there are a few pre-requisites needed before you can query Azure DevOps directly. These are detailed as follows:

1. You must ensure that you are running Azure CLI version 2.0.49 or higher. You can check this by simply running the following command:

   ```powershell
   az --version
   ```

1. Your Azure CLI must have the azure-devops extension added to it. To check if this is already available run the following command to list your extensions:
   ```powershell
   az extension list
   ```

   If the extension is not listed you can add it as follows:
   ```powershell
   az extension add --name azure-devops
   ```

   For further information on this extension, you can view the Microsoft documentation titled "[Use extensions with Azure CLI"](https://docs.microsoft.com/en-us/cli/azure/azure-cli-extensions-overview?view=azure-cli-latest).

1. Your az session must be signed in to your Azure tenant, and to do this use the az login command and provide the relevant credentials:
   ```powershell
   az login
   ```
1. Finally, to avoid having to provide a project context every time you run an az devops command you should set a default project context as follows (obviously use your own organization and project):
   ```powershell
   az devops configure --defaults organization=https://retracement.visualstudio.com/ project="ACME Corp"
   ```

You are now ready to go!

---

# Querying DevOps through Azure CLI

In order to find out all the commands now made available to you with your new extension, you can execute the following command:

```powershell
az devops -h
```

By doing so, you will note that the extension provides devops subgroup commands such as teams -for example to list your current devops teams:

```powershell
az devops team list
```

As the help context shows, the extension also provides “related groups” (such as repos) to manage other facets of Azure DevOps. In our specific example, we want to query all available repos for our Azure DevOps project. We can do this as follows:

```powershell
az repos list
```

Notice that your results come back in [JSON](https://www.json.org/json-en.html) format by default. We can override this and return results in tabular format by using the output parameter:

```powershell
az repos list --output tabletable output
```

The Azure CLI also provides a query option so that you can provide a [JMESPath](https://jmespath.org) query string to filter your results. For instance, in the most basic scenario we can return the first element from our results (using zero-based index notation):

```powershell
az repos list --query [0]
```

That is clearly not so useful, so instead, I want to return specific properties from all repos. In this case, I want to return its name, Azure repo url path, and the default branch that is set:

```powershell
az repos list --query [].[name,webUrl,defaultBranch]
```

In our final example we will return the results in a tabular format and alias our property names (for our column headings):

```powershell
az repos list --query "[].{Name:name, Url:webUrl, DefaultBranch:defaultBranch}" --output tablewith aliases
```

# Summary

Being able to programmatically query Azure DevOps through the Azure CLI is incredibly useful and powerful and could help you keep your environment standardized (for example ensure branch policies across repos are identical) or even provides a method that you can easily track change. Obviously we are not just restricted to the Azure DevOps repos, we can look at all facets of the environment. For example, to list all current builds in a project we can issue the following command:

```powershell
az pipelines build list -o table
```

As a final point of note, I confess to finding JMESPath to query and filter my results far less intuitive or simple than with other languages (especially given the semi-structured nature of the data you are filtering), but with a little bit of trial and error, you can eventually get there!

I hope you find my post useful and please feel free to provide feedback in the comments.

# Further References
https://docs.microsoft.com/en-us/cli/azure/query-azure-cli?view=azure-cli-latest
https://jmespath.org/examples.html