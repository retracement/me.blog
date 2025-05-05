+++
date = '2020-03-25T19:34:00+01:00'
draft = false
title = 'Cannot delete old build definitions in Azure DevOps'
+++
I have been experiencing a problem for quite a while now in my current environment, in that some of our old builds cannot be deleted. When you attempt to do so it results in the following error:

![build error](/images/2020/cannotdel.webp)

_One or more builds associated with the requested pipeline(s) are retained by a release. The pipeline(s) and builds will not be deleted._

Many of our pipelines have undergone a lot of change over time to the degree it is not even obvious anymore why (or indeed where) these builds are being prevented from being dropped. The only thing that is clear is that until they can be, the old build definitions will remain.

I have tried to set the _Stop retaining the build_ setting for all builds associated with a build definition to no avail. The setting just does not seem to want to take in most cases.

![stop retaining build](/images/2020/stopretaining.png)

I have also tried playing around with build retention policies and even tried tidying up the release pipelines (and releases) themselves. Unfortunately for me, those darn build pipelines do not want to delete.

Today I decided to put some of my recent Powershell and Azure DevOps REST API experiences (see previous posts in this blog) to the test and attempt to get to the bottom of the problem. As it turns out there is a build property called `retainedByRelease` that is exposed through the REST API which is the reason why a build cannot be removed -resulting in our irritating error.

Using the same technique that I wrote about in [Querying Azure DevOps REST API with PowerShell](/posts/2020/03/11/querying-azure-devops-rest-api-with-powershell) I first decided to try an report on this property. Please refer back to the post above for more explanation on utilizing the REST API, but I realized I would need to make two REST API calls. The first would be to query one or more build definitions and the second would be to query all builds for each build definition. More specifically, with this last call I would report on the `retainedByRelease` property.

---

# Querying the build definition builds

In the first piece of code we create our authorization token.

```powershell
$personalToken = "tiksj25oumfavuzr4316vhpxw2mywzbapxj7sw3x2xet3dml1ygy"
$token = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($personalToken)"))
$header = @{authorization = "Basic $token"}
```

Next we set our organization and project variables.

```powershell
$organization = "retracement"
$project = "ACME%20Corp"
```

Our first REST API call queries all build definitions within the project.

```powershell
#all build definitions
$url = "https://dev.azure.com/$organization/$project/_apis/build/definitions?api-version=6.0-preview.7"
$builddefinitions = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
$builddefinitions.value | Sort-Object id|ForEach-Object {
Write-Host $_.id $_.name $_.queueStatus
 
#all builds for a definition
$url = "https://dev.azure.com/$organization/$project/_apis/build/builds?definitions=" + $_.id + "&api-version=6.0-preview.5"
$builds = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
 
$builds.value | Sort-Object id|ForEach-Object {
#report on retain status
Write-Host " BuildId" $_.id "- retainedByRelease:" $_.retainedByRelease
}
Write-Host
}
```

For brevity I provide only a subset of the results:

```text
339 SQL Dacpac Build enabled
BuildId 43045 - retainedByRelease: False
BuildId 43051 - retainedByRelease: False
BuildId 43053 - retainedByRelease: True
BuildId 43307 - retainedByRelease: True
BuildId 43325 - retainedByRelease: True
 
366 Databricks Notebooks Build enabled
BuildId 45338 - retainedByRelease: False
BuildId 45340 - retainedByRelease: False
BuildId 45346 - retainedByRelease: True
BuildId 46032 - retainedByRelease: True
 
375 ARM Templates Build enabled
BuildId 46452 - retainedByRelease: False
BuildId 46454 - retainedByRelease: True
```

As you can see, from the three active build definitions listed, each one has at least one build that is marked for retention by release.

---

# Setting the build retainedByRelease property

Now we have a procedure in place to query the `retainedByRelease` property, it is just as easy to set it. If you are trying to remove a specific Build Definition (or builds), you can implement a filter in the builddefinitions iterator. So:

```powershell
$builddefinitions.value | Sort-Object id|ForEach-Object {
```

Would now become:

```powershell
$builddefinitions.value | where {$_.name -eq "ARM Templates Build"}|Sort-Object id|ForEach-Object {
```

In the above example we are filtering on a single build definition, but feel free to use the filter of your choosing.

The final thing we need to do is make a REST API call to update each build returned by this filtered build definition. We can so this as follows by adding the following line inside our build iterator:

```powershell
Invoke-RestMethod -Uri $url -Method Patch -Body (ConvertTo-Json @{"retainedByRelease"='false'}) -ContentType "application/json" -Headers $header
```

You will note the use of `-Method Patch` within this call rather than `-Method Get` and the JSON body. The patch method allows us to partially update resources (in this case one field) with the JSON body provided.

---

# Putting it all together

So if we wanted to update the builds of one specific Build Definition called _ARM Templates Build_ we would run the following code:

```powershell
$personalToken = "tiksj25oumfavuzr4316vhpxw2mywzbapxj7sw3x2xet3dml1ygy"
$token = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($personalToken)"))
$header = @{authorization = "Basic $token"}
 
$organization = "retracement"
$project = "ACME%20Corp"
 
#all build definitions
$url = "https://dev.azure.com/$organization/$project/_apis/build/definitions?api-version=6.0-preview.7"
$builddefinitions = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
 
$builddefinitions.value | where {$_.name -eq "ARM Templates Build"}|Sort-Object id|ForEach-Object {
Write-Host $_.id $_.name $_.queueStatus
 
#all builds for a definition
$url = "https://dev.azure.com/$organization/$project/_apis/build/builds?definitions=" + $_.id + "&api-version=6.0-preview.5"
$builds = Invoke-RestMethod -Uri $url -Method Get -ContentType "application/json" -Headers $header
 
$builds.value | Sort-Object id|ForEach-Object {
#report on retain status
Write-Host " BuildId" $_.id "- retainedByRelease:" $_.retainedByRelease
 
#api call for a build
$url = "https://dev.azure.com/$organization/$project/_apis/build/builds/" + $_.id + "?api-version=6.0-preview.5"
 
#set retainedByRelease property to false
Invoke-RestMethod -Uri $url -Method Patch -Body (ConvertTo-Json @{"retainedByRelease"='false'}) -ContentType "application/json" -Headers $header
}
Write-Host
}
```

Now that all your builds for the ARM Templates Build Build Definition are deleted, you should be able to remove this build definition without further error (you do not need to first remove its builds).

---

# Summary

There are certain issues that you might experience in Azure DevOps which seem almost impossible to resolve through the GUI, but yet again the Azure DevOps API can come to our rescue. In this specific example we have easily queried aspects of DevOps through PowerShell, and this time even changed information through it to resolve our problem.

I hope you find this post useful for this rather frustrating problem!