+++
date = '2020-02-17T13:43:00+01:00'
draft = false
title = 'Use Powershell to Consume Your Azure Cli Devops Result Set'
+++

Use PowerShell to consume your Azure CLI DevOps result set
Before we get going it is probably first worth me pointing out (in case you are wondering) that the whole premise of this post stems from a lack of native Azure DevOps PowerShell Module. Yes there are a few solutions out there, but at the time of writing there is no official Microsoft PowerShell DevOps module, so we are stuck with using the CLI if you want to avoid using these other solutions.

In my post [Using Azure CLI to query Azure DevOps]() I explained how you can use the Azure CLI to query Azure DevOps so you can obtain useful information on builds, releases, and other useful information. The solution required a certain level of skill with [JMESPath](http://jmespath.org/) to manipulate your result sets -which as explained can be a little confusing.

However once you have a bare bones result set, it is likely that you will want to consume these results in a more user-friendly environment such as PowerShell so that you can build upon these data sets. I thought this would be an easy thing to do, but as you will see below it was anything but.

# A simple example
Lets write a simple Azure CLI query with JMESPath and assign the results to a PowerShell variable:
$releases = az pipelines release list --query "[].{name:name,pipeline:releaseDefinition.name}" --output table

If we take a quick look at the contents of the $releases variable we get the following result set:

```powershell
PS C:\> $releases
```

```text
Name          Pipeline
------------  ----------------------------------------------
Release-12    Databricks Pipeline
Release-11    Databricks Pipeline
Release-10    Databricks Pipeline
```

The problem is that this result set is coming across as an array of values rather than a tabular data-set and we can see this if we try to select a column:

```powershell
PS C:\> $releases|Select-Object Name
```

```text
Name
----
```

# ConvertFrom-String
After lots of trial and error and frustrating attempts to arrive at a solution which I would have expected to be easy to get around, I was given a great tip from Jonathan Allen ([t](https://twitter.com/fatherjack)) to try and use the `ConvertFrom-String` cmdlet. Using this on its own gives us the following:

```powershell
PS C:\> $releases | ConvertFrom-String -PropertyNames Name, Pipeline
```

```text
Name          Pipeline
------------  ----------------------------------------------
Release-12    Databricks
Release-11    Databricks
Release-10    Databricks
Release-12    Pipeline
Release-11    Pipeline
Release-10    Pipeline
```

Clearly this looks like we are getting closer so I tried removing the header and adding an explicit delimiter into this cmdlet:

First run the following to assign object into `$releases` variable.
```powershell
PS C:\> $releases = $releases | where-object {($_.tostring() -ne "------------  ----------------------------------------------" -And $_.tostring() -ne "Name          Pipeline")}
```
Then pipe into the `ConvertFrom-String` cmdlet.
```powershell
PS C:\> $releases | ConvertFrom-String -PropertyNames Name, Pipeline -Delimiter "  "
```

```text
Name          Pipeline
----          --------
Release-12    Databricks Pipeline
Release-1     Databricks Pipeline
```

Unfortunately due to the fact we do not have a consistent delimiter due to the change in the first column width when our result number changes in digit size we get inconsistent formatting results as exampled above.

# Back to basics
I have always been told that you should try and use PowerShell pipeline functionality to keep code brief and optimal, but try as I might I could not arrive at an acceptable solution. In the end I decided to try and return to programming basics and write code to format the result sets as I would in C# or other procedural languages.

So in a nutshell, I decided to do the following:

1. Create a table, with correct number of columns
1. Create DevOps result set array and strip the header
1. Iterate through the array and add a new row to our table containing data for each column
1. Manipulate our table in PowerShell as we want

The full solution was as follows:

```powershell
$releases = az pipelines release list --query "[].{name:name,pipeline:releaseDefinition.name}" --output table
$table = New-Object System.Data.DataTable
$table.Columns.Add("Name")
$table.Columns.Add("Pipeline")
 
# strip headers and column string
$releases = $releases | where-object {($_.tostring() -ne "------------  ----------------------------------------------" -And $_.tostring() -ne "Name          Pipeline")}
$break = 14 # where 14 is the breakpoint between columns
foreach ($row in $releases)
{
    #$row
    $column1 = $row.ToString().Substring(0,$break)
    $column2 = $row.ToString().Substring($break,$row.ToString().Length - $break)
    $table.Rows.Add($column1, $column2) | Out-Null # Out-Null required to avoid echo to screen
}
$table | select Name | where {($_.Pipeline -eq "Databricks Pipeline")} | ft
```

And the last line of this code simply selects the Name of the last three releases where the pipeline was called Databricks Pipeline (proving that our result set is tabular).

```text
Name          
------------
Release-12
Release-11
Release-10
```

# Conclusion
As you can see, in those situations where our result sets are not tabular at the point of assignment to a PowerShell variable, it is still possible to convert them as you see fit. Whilst this is a little painful to initially set up (depending upon the number of columns you might have), it is a fairly repeatable pattern. It is disappointing that I could not find a pure pipeline driven solution (and I did try many different techniques including formatting strings), but if you are aware of a better solution, please let me know! If I come across a better solution myself I will update this post accordingly!