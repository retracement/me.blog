+++
date = '2020-04-14T12:34:00+01:00'
draft = false
title = 'Restore Azure Devops Repository'
+++
If you are using [Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/), you might be comforted that your [Git](https://git-scm.com/) repo is "in the Cloud" and automatically has availability and disaster guarantees. However you (or someone else) still have the ability to accidentally (or maliciously) delete repos from Azure DevOps Repos. Surprisingly, at the time of writing, there is no GUI based option to restore your repo. This might initially instill a sense of panic as you frantically search for the latest local clone to replace your remote – but there is a better way.

When you delete an Azure DevOps repository, it is initially soft-deleted to the "recycle bin". After a period of time (oddly I have failed to find an offical Microsoft reference stating exactly what this but I believe it is 28 days) it is automatically purged and hard-deleted. Although there is no GUI support to restore your soft-deleted repositories, that ability is exposed through the Azure DevOps REST API, but frustratingly the [Microsoft Azure DevOps Services REST API Reference](https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-6.0) does not provide a worked example in the [Repositories – Restore Repository From Recycle Bin](https://docs.microsoft.com/en-us/rest/api/azure/devops/git/repositories/restore%20repository%20from%20recycle%20bin?view=azure-devops-rest-5.1) API call page.

To make your life easier, I will provide the solution below!

---

# Getting started with Azure DevOps REST API and PAT token

Within my blog so far I have provided several worked examples of making a REST API call to Azure DevOps. If you are new to this, I suggest you first check out my post titled [Querying Azure DevOps REST API with PowerShell](/posts/2020/03/11/querying-azure-devops-rest-api-with-powershell).

Once you have assigned your `$header` variable from an encoded PAT token (_as documented in the aforementioned article_) you are ready to roll!

# Set your repository's Organisation and project
Each project will contain its own set of Azure repositories. Ensure you provide the correct values for your organization and project- and ensure that for any names with spaces are correctly replaced using `%20` (so that a valid url can be formed).

```powershell
$organization = "retracement"
$project = "ACME%20Corp"
```
---

# REST API call to list repositories in the recycleBin
From the [Microsoft Azure DevOps Services REST API Reference](https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-6.0) we can call the [Repositories – List](https://docs.microsoft.com/en-us/rest/api/azure/devops/git/repositories/list?view=azure-devops-rest-5.1) REST API call to return a list of all deleted repositories in our recycleBin for our organization's project.

We will first build up our `$url` using the variables set earlier.

```powershell
$url = "https://dev.azure.com/$organization/$project/_apis/git/recycleBin/repositories?api-version=5.1-preview.1"
```

Now that all variables are set we can make our REST API call and iterate over all deleted repositories.

```powershell
$deletedRepos = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
  # for each repository
Write-Host "Deleted repositories"
Write-Host "--------------------"
 
$deletedRepos.value | ForEach-Object {
    $repoId = $_.id
    $repoName = $_.name
    $deletedBy = $_.deletedBy.displayName
    $deletedDate = $_.deletedDate
    Write-Host "repoId:" $repoId $repoName "deleted on" $deletedDate "deleted by" $deletedBy
}
```

The following output is returned:

```text
Deleted repositories
--------------------
repoId: 3b1bbfe0-470d-4724-bc69-6ec29ff88cb5 SuperImportantRepo deleted on 2020-04-08T13:44:21.807Z deleted by Mark Broadbent
repoId: 4c3abef0-520a-2461-ac70-1ad30ef11ab7 NotImportantRepo deleted on 2020-04-12T10:00:01.201Z deleted by Mark Broadbent
```

We have now identified that someone (me!) has deleted a super important repository by accident. Using the repoId we can use this to restore it from the `recycleBin`.

---

# Recover soft deleted repository

First we need to set a variable $repoId to the deleted repository (SuperImportantRepo) repoId that we identified earlier. This will be used in our next REST API call.

```powershell
$repoId = "3b1bbfe0-470d-4724-bc69-6ec29ff88cb5"
```

Now we can return back to the [Repositories – Restore Repository From Recycle Bin](https://docs.microsoft.com/en-us/rest/api/azure/devops/git/repositories/restore%20repository%20from%20recycle%20bin?view=azure-devops-rest-5.1) REST API call page as use this to build out our new url. As you will see, the url contains our `$repoId` and we will also create a `$body` variable set to a JSON key value pair setting the `deleted` property to false. This JSON body is passed into our REST API call using the `Patch` Method.

```powershell
$url = "https://dev.azure.com/$organization/$project/_apis/git/recycleBin/repositories/" + $repoId +"?api-version=5.1-preview.1"
$body = ConvertTo-Json @{"deleted"= "false"}
Invoke-RestMethod -Uri $url -Method Patch -Body ($body) -ContentType "application/json" -Headers $header
```

The output of our final REST API call results in:

```text
id            : 3b1bbfe0-470d-4724-bc69-6ec29ff88cb5
name          : SuperImportantRepo
url           : https://dev.azure.com/retracement/e6fa212f-3520-4c30-8c28-d6bd88926ff2/_apis/git/repositories/3b1bbfe0-470d-4724-bc69-6ec29ff88cb5
project       : @{id=e6fa212f-3520-4c30-8c28-d6bd88926ff2; name=ACME%20Corp; description=Super Important Repository for mission critical systems; url=https://dev.azure.com/retracement/_apis/projects/e6fa212f-3520-4c30-8c28-d6bd88926ff2; state=wellFormed; revision=626; 
                visibility=private; lastUpdateTime=2019-11-20T15:49:09.773Z}
defaultBranch : refs/heads/master
size          : 731
remoteUrl     : https://retracement@dev.azure.com/retracement/ACME%20Corp/_git/SuperImportantRepo
sshUrl        : git@ssh.dev.azure.com:v3/retracement/ACME%20Corp/SuperImportantRepo
webUrl        : https://dev.azure.com/retracement/ACME%20Corp/_git/SuperImportantRepo
```

As we can see from the above output success!

---

# Summary

As I have shown, deleting a repository by accident in Azure DevOps does not have to be a disaster recovery situation since the recycleBin and Azure DevOps REST API makes it relatively simply to view and restore (when you know how!). However it is worth pointing out that for Git repositories, no similar situation exists if you delete a branch (unlike with Tfs Repos in Azure DevOps). So the moral of the story is to ensure you periodically back up all your remote repositories AND set branch policies to protect them against accidental deletion.

Hope you enjoyed the post!