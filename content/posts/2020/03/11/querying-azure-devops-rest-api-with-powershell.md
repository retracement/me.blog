+++
date = '2020-03-11T13:13:00+01:00'
draft = false
title = 'Querying Azure Devops Rest Api With Powershell'
+++

![sad dog](/images/2020/saddog.png)

In previous posts we have talked about trying to use and consume Azure DevOps using PowerShell and utilizing the Azure CLI. In particular, my post titled Use [PowerShell to consume your Azure CLI DevOps result](/posts/2020/02/17/use-powershell-to-consume-your-azure-cli-devops-result-set.md) set painted a rather frustrating picture when trying to manipulate the tabular dataset from the Azure CLI. Furthermore, our functionality is restricted to only those commands implemented by the Azure CLI Azure DevOps add-in -as will become increasingly obvious, this is limited to say the least.

There is a better way to query Azure Devops – Azure DevOps REST API to the rescue.

---

Probably the first thing you will want to do is understand what kind of queries and actions you can make against the Azure DevOps REST API. These are not limited to reporting upon existing configurations, it can also be used to change configuration. For example, through the REST API we could POST a call to create a brand new release. For the purposes of simplicity we will simply query Azure DevOps in this article.

In order to understand all the potential Azure DevOps queries and actions you can make through the Azure DevOps REST API you can refer to the [Microsoft Azure DevOps Services REST API Reference](https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-6.0). We will return back to this reference when we look to make specific calls but before we get there, we will first break down the steps that you will need to take in order to successfully make your call.

# Create your PAT token

In order to securely communicate with Azure DevOps, you will first need to create a PAT token which will allow your code to make an authorized call to the REST API. This can be created by clicking the configuration icon from the toolbar of Azure DevOps.

![PAT token](/images/2020/pattoken.png)

In my specific example I am going to create a PAT token with Full access, but it is recommended that you should create a Custom defined scope to limit the security surface area. Also note that you must set an expiration date to this token and once it expires, you will need either regenerate it, or create a new one to meet your personal requirements.

[PAT Details](/images/2020/patdetails.png)

We can now use this PAT in your REST API call, but it is important to ensure this string uses [Base64](https://en.wikipedia.org/wiki/Base64) encoding.

# Assign and encode your PAT token

```powershell
$personalToken = "tiksj25oumfavuzr4316vhpxw2mywzbapxj7sw3x2xet3dml1ygy"
 
#Write-Host "Initialize authentication context" -ForegroundColor Yellow
$token = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($personalToken)"))
$header = @{authorization = "Basic $token"}
```

# Make your REST API call
From the [Microsoft Azure DevOps Services REST API Reference](https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-6.0) select the REST API call URI that you need to use.

In this first example, the URI chosen is used to query all existing Azure DevOps Projects. The following code invokes the Azure DevOps REST API call and iterates through each project.

For example, to make a call to query all projects in your Azure DevOps organisation you can call the following:

```powershell
$url = "https://retracement.visualstudio.com/_apis/projects?api-version=5.0"
 
$output = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
 
$output.value | ForEach-Object {
    Write-Host $_.name
}
```

In my case I get the following Projects returned:

```text
Parts Unlimited
ACME Corp
Main Project
My Test Project
```

---

# Other examples

In the following examples I will perform some common queries against our Azure DevOps project. I will do my best to expand and implement new REST API calls over time in follow up posts.

# Query all build definitions

In this example we will return the results in descending order. There is also a bit of further work needed to parse the definition output to improve the quality of the result set. I've left a few extra fields commented out for brevity.

```powershell
# Builds API call
$url = "https://retracement.visualstudio.com/ACME%20Corp/_apis/build/builds?api-version=5.0"
$output = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
$output.value | Sort-Object id -Descending|ForEach-Object {
    Write-Host $_.buildNumber - $_.status - $_.reason# - $_.definition - $_.url
}
```

This returns the following builds:

```text
26740 - completed - schedule
20021 - completed - schedule
17436 - completed - schedule
14701 - completed - manual
```

# Query all release pipeline definitions

In this example we will pull back a sorted list of all release pipeline definitions.

```powershell
# Release Definitions API call
$url = "https://retracement.vsrm.visualstudio.com/ACME%20Corp/_apis/release/definitions?api-version=5.0"
$output = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
$output.value | Sort-Object name|ForEach-Object {
    Write-Host $_.name
}
```

And we get all current release definitions:

```text
Big Daddy Release Pipeline
Little Tom Release Pipeline
Widgets Release Pipeline
```

# Query all repositories in a project

```powershell
# Repositories API call
$url = "https://retracement.visualstudio.com/ACME%20Corp/_apis/git/repositories?api-version=6.0-preview.1"
$output = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
$output.value | ForEach-Object {
    Write-Host $_.id,$_.name
}
```

The following repositories are returned:

```text
ba008565-118a-41e6-878c-d7a8180bf734 Widget Database
49ebc167-8b48-4202-af4e-f8fd885aede1 Widget Notebooks
682c7ebf-11d1-443f-b0b0-fbd7f2bfdd71 ACME dotNet master
112635ba-c5e7-4c91-bae7-ff014cf36be4 ACME Helper
```

# Query a repository branches

In this next example we will take a repository id and use this in our REST API call to query it's branches.

```powershell
# Repository API call
$repoId = "ba008565-118a-41e6-878c-d7a8180bf734"
$url = "https://retracement.visualstudio.com/ACME%20Corp/_apis/git/repositories/$repoId/refs?api-version=6.0-preview.1"
$output = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
$output.value | ForEach-Object {
    Write-Host $_.name
}
```

The branches returned for this repo are as follows:

```text
ba008565-118a-41e6-878c-d7a8180bf734 refs/heads/mybrillfeature
ba008565-118a-41e6-878c-d7a8180bf734 refs/heads/development
ba008565-118a-41e6-878c-d7a8180bf734 refs/heads/master
```

# Query all branches for all repositories in a project

We can put the previous two API calls together to query all branches for all repositories.

```powershell
#branches for each repo
$url = "https://retracement.visualstudio.com/ACME%20Corp/_apis/git/repositories?api-version=6.0-preview.1"
$repo = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
$repo.value | ForEach-Object {
    $repoId = $_.id
    $repoName = $_.name
    $url = "https://retracement.visualstudio.com/ACME%20Corp/_apis/git/repositories/$repoId/refs?api-version=6.0-preview.1"
    $output = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
    $output.value | ForEach-Object {
        Write-Host $repoId - $repoName - $_.name
    }
}
```

This returns:

```text
ba008565-118a-41e6-878c-d7a8180bf734 - Widget Database - refs/heads/mybrillfeature
ba008565-118a-41e6-878c-d7a8180bf734 - Widget Database - refs/heads/development
ba008565-118a-41e6-878c-d7a8180bf734 - Widget Database - refs/heads/master
49ebc167-8b48-4202-af4e-f8fd885aede1 - Widget Notebooks - refs/heads/development
49ebc167-8b48-4202-af4e-f8fd885aede1 - Widget Notebooks - refs/heads/master
682c7ebf-11d1-443f-b0b0-fbd7f2bfdd71 - ACME dotNet master - refs/heads/development
682c7ebf-11d1-443f-b0b0-fbd7f2bfdd71 - ACME dotNet master - refs/heads/master
112635ba-c5e7-4c91-bae7-ff014cf36be4 - ACME Helper - refs/heads/development
112635ba-c5e7-4c91-bae7-ff014cf36be4 - ACME Helper - refs/heads/master
```

# Tabular query of all branches for all repositories in a project
And finally, our Azure DevOps REST API result set is far more useful as a tabular object so that we can manipulate it further in PowerShell (should we so wish) and perform various filters and sorts against it. So extending the previous example we will put our result set into a table object.

```powershell
#branches for each repo in a table
$table = New-Object System.Data.DataTable #create table and columns
$table.Columns.Add("Id")
$table.Columns.Add("Repository")
$table.Columns.Add("Branch")
 
$url = "https://retracement.visualstudio.com/ACME%20Corp/_apis/git/repositories?api-version=6.0-preview.1"
$repo = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
$repo.value | ForEach-Object {
    $repoId = $_.id
    $repoName = $_.name
    $url = "https://retracement.visualstudio.com/ACME%20Corp/_apis/git/repositories/$repoId/refs?api-version=6.0-preview.1"
    $output = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
    $output.value | ForEach-Object {
        $table.Rows.Add($repoId, $repoName, $_.name)|Out-Null
    }
}
$table | select Repository, Branch | Sort-Object Repository, Branch| ft
```

Querying the table object and filtering on two columns returns:

```text
Repository          Branch                                         
----------          ------
ACME dotNet master  refs/heads/development
ACME dotNet master  refs/heads/master
ACME Helper         refs/heads/development
ACME Helper         refs/heads/master
Widget Database     refs/heads/mybrillfeature
Widget Database     refs/heads/development
Widget Database     refs/heads/master
Widget Notebooks    refs/heads/development
Widget Notebooks    refs/heads/master
```

# Conclusion
As you have seen, the Azure DevOps API is not only very easy to use and consume through PowerShell (when you know how), but provides a much more comprehensive route to interface with Azure DevOps than the other techniques I have previously talked about (such as the Azure CLI).

In future posts I will talk about implementing other queries and actions such listing all outstanding pull requests across repositories and even how to create a release.

Hope you find this post useful, please leave your comments below!