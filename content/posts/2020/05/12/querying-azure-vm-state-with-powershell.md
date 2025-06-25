+++
date = '2020-05-12T15:16:36+01:00'
draft = false
title = 'Querying Azure Vm State With Powershell'
+++
I was recently given the task of identifying the state of an Azure VM so that an automation script using the `az vm run-command invoke` would not fail if the VM was down or under a reboot.

I initially thought the task would be really easy and a simple query of the VM state using `Get-AzVM` would provide us with a running state property of the VM, but as it happens the state is a little abstracted.

# Using Get-AzVM cmdlet
We will query our Virtual Machine running state using the Get-AzVM cmdlet using the -Status switch.

`Get-AzVM -resourcegroupname "Demo" -name "server1" -Status`

Running this command will return an object of type `PSVirtualMachineInstanceView` that we can use to test the status of the VM. The output to the above command is as follows:

```text
ResourceGroupName          : Demo
Name                       : server1
HyperVGeneration           : V1
BootDiagnostics            : 
  ConsoleScreenshotBlobUri : https://ortodiag.blob.core.windows.net/bootdiagnostics-server1-cb292d11-c4c2-4abd-8f28-c0de31e68
010/server1.cb292d11-c4c2-4abd-8f28-c0de31e68010.screenshot.bmp
  SerialConsoleLogBlobUri  : https://ortodiag.blob.core.windows.net/bootdiagnostics-server1-cb292d11-c4c2-4abd-8f28-c0de31e68
010/server1.cb292d11-c4c2-4abd-8f28-c0de31e68010.serialconsole.log
Disks[0]                   : 
  Name                     : server1_OsDisk_1_6311f5e2cf24419eb27ac5c3734e952b
  Statuses[0]              : 
    Code                   : ProvisioningState/succeeded
    Level                  : Info
    DisplayStatus          : Provisioning succeeded
    Time                   : 12/05/2020 12:37:11
Disks[1]                   : 
  Name                     : server1_log_drive
  Statuses[0]              : 
    Code                   : ProvisioningState/succeeded
    Level                  : Info
    DisplayStatus          : Provisioning succeeded
    Time                   : 12/05/2020 12:37:11
Statuses[0]                : 
  Code                     : ProvisioningState/succeeded
  Level                    : Info
  DisplayStatus            : Provisioning succeeded
  Time                     : 12/05/2020 12:37:11
Statuses[1]                : 
  Code                     : PowerState/deallocated
  Level                    : Info
  DisplayStatus            : VM deallocated
```

This is quite good in that we have lots of information about our VM and associated resource states. We can see that it is a Generation 1 VM and see the state of our disks and our VM. More specifically we see the `Statuses[1]` element and it’s Code property to get our VM status.

The possible Code states are:

- PowerState/deallocated (VM is stopped and deallocated)
- PowerState/starting (VM is starting)
- PowerState/running (VM is running – the Azure Portal may show a VM as running before this state returns)
- PowerState/deallocating (VM is stopping)

We now have enough information to write our PowerShell code.

# Querying Azure VM state
As touched upon earlier, we need to access the `PSVirtualMachineInstanceView` object and access the `Statuses[1]` element and it’s Code property. This gives us a `$provisioningState` value that we can test against our static state (in our case PowerState/running).

If the state is not running then we will keep looping with a 5 second wait. The previous state is tracked only for output sugar so that we will only write to the screen on a state change:

```powershell
$vmName = "server1"
$resourceGroup = "Demo"
$lastProvisioningState = ""
$provisioningState = (Get-AzVM -resourcegroupname $resourceGroup -name $vmName -Status).Statuses[1].Code
$condition = ($provisioningState -eq "PowerState/running")
while (!$condition){
    if ($lastProvisioningState -ne $provisioningState){
        write-host $vmName "under" $resourceGroup "is" $provisioningState "(waiting for state change)"
    }
    $lastProvisioningState = $provisioningState
 
    sleep -Seconds 5
    $provisioningState = (Get-AzVM -resourcegroupname $resourceGroup -name $vmName -Status).Statuses[1].Code
 
    $condition = ($provisioningState -eq "PowerState/running")
}
write-host $vmName "under" $resourceGroup "is" $provisioningState -fore green
```

# Results
The script is executed and we stop and (ultimately) start the server1 VM. The output from the script was as follows:
![VM State](/images/2020/vmstate-output.webp)

As can be seen, this simple backoff script correctly reported on shutdown and startup of our VM.

# Conclusion
Whilst I do not pretend for one moment that this script is particularly clever (or even optimally written), it does at least demonstrate how easy it is to query our Azure VM state (with appropriate backoffs) through PowerShell so that we can accurately and predictably perform follow up actions on them.

Obviously this can be adapted for your specific use-case.