---
layout: post
title: "IaC Deployment of AVD Observations"
subtitle: "- Host Pool Registration via ARM"
date: 2026-03-04
author: Matthew

series: "avd-iac-observations"
series_part: 3
series_title: "IaC FSLogix Registry Configurations"

tags:
  - azure
  - avd
  - iac
---

# Azure Virtual Desktop + FSLogix
## Configuring Registry Keys Reliably Inside an ARM Template

*A practical guide for IT Pros packaging AVD as an Azure Marketplace managed application*  
*Frametype Solutions LLC • March 2026*

---

## Introduction

The [previous post in this series](/) covered what Entra Kerberos is and why it is the right authentication model for FSLogix profile storage in cloud-only AVD deployments. That post ended with a list of registry keys that must be present on every session host for Entra Kerberos and FSLogix to work:

- `HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\Kerberos\Parameters\CloudKerberosTicketRetrievalEnabled` = 1
- `HKLM:\Software\Policies\Microsoft\AzureADAccount\LoadCredKeyFromProfile` = 1
- `HKLM:\SOFTWARE\FSLogix\Profiles\Enabled` = 1
- `HKLM:\SOFTWARE\FSLogix\Profiles\VHDLocations` = `\\<storageaccount>.file.core.windows.net\profiles`
- `HKLM:\SOFTWARE\FSLogix\Profiles\VolumeType` = VHDX

What that post did not cover is how to write those keys automatically as part of an ARM template deployment — without any manual intervention after the VMs are provisioned. That is what this post is about.

Getting this right turned out to be harder than expected. The obvious approach — using a Custom Script Extension — has a class of failure modes that are not well documented and that produce no useful error output. This post documents what those failure modes are, why they occur, and the pattern that actually works reliably.

---

## The Obvious Approach: Custom Script Extension

The Custom Script Extension (CSE) is the standard ARM mechanism for running scripts on a Windows VM after provisioning. It accepts a script file and a command to execute, downloads the script to the VM, and runs it. For simple post-provisioning tasks it works well.

For this use case the intent was to:

1. Encode `Configure-FSLogix.ps1` as a base64 string and embed it directly in the template as a variable — eliminating any dependency on an external script URL, which is a hard blocker for Azure Marketplace certification
2. Pass the base64 string to the CSE
3. Have the CSE decode it, write it to disk, and execute it with the FSLogix share UNC path as a parameter

That approach fails in two distinct ways, and the failure modes are easy to confuse with each other.

---

## Failure Mode 1: protectedSettings.script Is a Linux-Only Feature

The Custom Script Extension has two variants: one for Windows (`Microsoft.Compute.CustomScriptExtension`) and one for Linux (`Microsoft.Azure.Extensions.CustomScript`). They share a similar interface but differ in what `protectedSettings` supports.

The Linux CSE supports a `protectedSettings.script` field that accepts a base64-encoded script and executes it directly — no download, no file URL required. This is exactly what we needed.

The Windows CSE does not support `protectedSettings.script`. It accepts the field without error — the ARM deployment succeeds, the extension reports as provisioned — but the field is silently ignored. No script is written to disk. No script is executed. The only indication that anything went wrong is that the effects of the script are absent.

This behavior is not documented clearly in a single place. The Microsoft Learn documentation for the [Windows CSE](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows) and the [Linux CSE](https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-linux) are separate pages and the `protectedSettings.script` field does not appear in the Windows documentation at all. If you are working from Linux CSE examples or from community posts that do not specify the OS variant, it is easy to assume the field works on both.

**The diagnostic:** Run the following on the VM after deployment and check whether the script file exists:

```powershell
Test-Path "C:/configure-fslogix.ps1"
```

If it returns `False`, the script was never written. The CSE did not execute.

---

## Failure Mode 2: The Sequence Number Guard

Once `protectedSettings.script` was removed and the full decode-write-execute logic was moved into `commandToExecute`, a second problem appeared. The CSE handler on the VM was not executing the script at all on fresh VMs in a fresh resource group.

The CSE handler tracks sequence numbers internally to implement idempotency — it will not re-execute a configuration it has already processed. Each time the ARM deployment engine invokes the CSE handler during a deployment, the handler increments its internal sequence counter. The `enable` command that actually runs the script checks whether the current sequence number is greater than the last processed sequence number. If it is not, the handler exits without executing.

The handler log entry that indicates this is:

```
WARN: Current sequence number, 0, is not greater than the sequence number
      of the most recently executed configuration. Exiting...
```

In testing this appeared even on VMs that had just been created in a brand new resource group — which rules out state carrying over from a previous deployment. The cause is that during a deployment with multiple concurrent extensions (AADLoginForWindows, AzureMonitorWindowsAgent, DSC, and CSE all running on the same VM), the ARM deployment engine invokes the CSE handler multiple times while waiting for other extensions to complete. Each invocation increments the sequence counter. By the time all dependencies are resolved and the final `enable` call arrives, the handler considers the sequence already processed and skips execution.

The CSE handler log for this scenario shows multiple invocation entries within a short window — in one test deployment, nine `CommandExecution_*.log` files were created within thirteen minutes on a single VM before the final sequence guard fired.

`forceUpdateTag` on the CSE resource is supposed to address this by generating a new sequence number on each deployment, but it does not prevent the within-deployment sequence collision caused by repeated handler invocations during dependency resolution.

**The diagnostic:** Check the CSE handler log on the VM:

```powershell
Get-Content "C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension\1.10.20\CustomScriptHandler.log" | Select-Object -Last 50
```

If you see the sequence number warning, the script was never executed regardless of what `commandToExecute` contains.

---

## The Solution: deploymentScript + Invoke-AzVMRunCommand

The reliable pattern is to move the FSLogix registry configuration out of the CSE entirely and into a `Microsoft.Resources/deploymentScripts` resource that uses `Invoke-AzVMRunCommand` to push the script into each VM from outside.

This approach has several advantages over CSE:

- The deployment script runs in an Azure Container Instance, not on the VM. It has no sequence number state and no handler lifecycle to contend with.
- It runs after an explicit `dependsOn` chain that ensures the VM and all its extensions are fully provisioned before the script is pushed.
- If the script fails, the deployment script resource reports a failed state back to ARM — the error surfaces in the deployment output rather than being silently swallowed.
- The base64-encoded script is passed as an environment variable to the container, decoded at runtime, written to a temp file, and sent to the VM via `Invoke-AzVMRunCommand`. The VM never needs to download anything from an external URL.

### What the deployment script does

The script running inside the container:

1. Decodes the base64 FSLogix configuration script from an environment variable
2. Writes it to a temporary file in the container
3. Calls `Invoke-AzVMRunCommand` to execute it on the target VM, passing the FSLogix share UNC path as a parameter
4. Checks the result and throws if the command failed

```json
{
  "type": "Microsoft.Resources/deploymentScripts",
  "apiVersion": "2023-08-01",
  "name": "[concat('script-fslogix-', padLeft(string(copyIndex(1)), 2, '0'))]",
  "location": "[parameters('location')]",
  "kind": "AzurePowerShell",
  "identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('scriptIdentityName'))]": {}
    }
  },
  "copy": {
    "name": "fslogixScriptLoop",
    "count": "[parameters('vmCount')]"
  },
  "dependsOn": [
    "[resourceId('Microsoft.Compute/virtualMachines/extensions', concat(variables('vmNamePrefix'), '-', padLeft(string(copyIndex(1)), 2, '0')), 'Microsoft.PowerShell.DSC')]",
    "[resourceId('Microsoft.Compute/virtualMachines/extensions', concat(variables('vmNamePrefix'), '-', padLeft(string(copyIndex(1)), 2, '0')), 'AADLoginForWindows')]",
    "[resourceId('Microsoft.Authorization/roleAssignments', guid(resourceGroup().id, variables('scriptIdentityName'), variables('roleVmContributor')))]"
  ],
  "properties": {
    "azPowerShellVersion": "9.0",
    "retentionInterval": "PT1H",
    "timeout": "PT15M",
    "forceUpdateTag": "[parameters('forceUpdateTag')]",
    "environmentVariables": [
      { "name": "ResourceGroupName", "value": "[resourceGroup().name]" },
      { "name": "VmName",            "value": "[concat(variables('vmNamePrefix'), '-', padLeft(string(copyIndex(1)), 2, '0'))]" },
      { "name": "ShareUNC",          "value": "[variables('fslogixProfileUnc')]" },
      { "name": "ScriptB64",         "value": "[variables('fslogixScriptB64')]" }
    ],
    "scriptContent": "$b = [Convert]::FromBase64String($env:ScriptB64); $tmp = [IO.Path]::GetTempFileName() + '.ps1'; [IO.File]::WriteAllBytes($tmp, $b); $result = Invoke-AzVMRunCommand -ResourceGroupName $env:ResourceGroupName -VMName $env:VmName -CommandId RunPowerShellScript -ScriptPath $tmp -Parameter @{ShareUNC=$env:ShareUNC} -ErrorAction Stop; Remove-Item $tmp -Force; $out = $result.Value | ForEach-Object { $_.Message }; Write-Output $out; if ($result.Status -eq 'Failed') { throw 'FSLogix configuration failed on ' + $env:VmName }"
  }
}
```

### The FSLogix configuration script

The script executed on the VM writes all required registry keys for FSLogix and Entra Kerberos in a single pass. It creates any missing registry paths, uses `-Force` on all `New-ItemProperty` calls so it is safe to run multiple times, and writes a transcript log to `C:\WindowsAzure\Logs\FSLogix\` for post-deployment verification.

```powershell
param(
  [Parameter(Mandatory=$true)][string]$ShareUNC
)

$ErrorActionPreference = "Stop"

$logDir  = "C:\WindowsAzure\Logs\FSLogix"
New-Item -ItemType Directory -Force -Path $logDir | Out-Null
$logFile = Join-Path $logDir ("Configure-FSLogix_{0:yyyyMMdd_HHmmss}.log" -f (Get-Date))
Start-Transcript -Path $logFile -Append

try {
  # Ensure required services are running for Entra Kerberos
  foreach ($svc in @("WinHttpAutoProxySvc", "iphlpsvc")) {
    $s = Get-Service -Name $svc -ErrorAction SilentlyContinue
    if ($s -and $s.Status -ne "Running") {
      Set-Service -Name $svc -StartupType Automatic -ErrorAction SilentlyContinue
      Start-Service -Name $svc -ErrorAction SilentlyContinue
    }
  }

  # Enable cloud Kerberos ticket retrieval
  $kerbPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\Kerberos\Parameters"
  if (-not (Test-Path $kerbPath)) { New-Item -Path $kerbPath -Force | Out-Null }
  New-ItemProperty -Path $kerbPath -Name "CloudKerberosTicketRetrievalEnabled" `
    -PropertyType DWord -Value 1 -Force | Out-Null

  # Required for FSLogix + Entra Kerberos credential key loading
  $aadPath = "HKLM:\Software\Policies\Microsoft\AzureADAccount"
  if (-not (Test-Path $aadPath)) { New-Item -Path $aadPath -Force | Out-Null }
  New-ItemProperty -Path $aadPath -Name "LoadCredKeyFromProfile" `
    -PropertyType DWord -Value 1 -Force | Out-Null

  # FSLogix profile container configuration
  $fsxProfiles = "HKLM:\SOFTWARE\FSLogix\Profiles"
  if (-not (Test-Path $fsxProfiles)) { New-Item -Path $fsxProfiles -Force | Out-Null }

  New-ItemProperty -Path $fsxProfiles -Name "Enabled"       -PropertyType DWord       -Value 1        -Force | Out-Null
  New-ItemProperty -Path $fsxProfiles -Name "VHDLocations"  -PropertyType MultiString  -Value $ShareUNC -Force | Out-Null
  New-ItemProperty -Path $fsxProfiles -Name "VolumeType"    -PropertyType String       -Value "VHDX"   -Force | Out-Null
  New-ItemProperty -Path $fsxProfiles -Name "DeleteLocalProfileWhenVHDShouldApply" `
    -PropertyType DWord -Value 1 -Force | Out-Null
  New-ItemProperty -Path $fsxProfiles -Name "FlipFlopProfileDirectoryName" `
    -PropertyType DWord -Value 1 -Force | Out-Null

} catch {
  Write-Error $_
  throw
} finally {
  Stop-Transcript | Out-Null
}
```

### RBAC requirement

The deployment script uses `Invoke-AzVMRunCommand`, which requires the **Virtual Machine Contributor** role on the resource group. The same user-assigned managed identity used for the registration token script handles this — add the role assignment alongside the existing Desktop Virtualization Contributor assignment:

```json
{
  "type": "Microsoft.Authorization/roleAssignments",
  "apiVersion": "2022-04-01",
  "name": "[guid(resourceGroup().id, variables('scriptIdentityName'), variables('roleVmContributor'))]",
  "dependsOn": [
    "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('scriptIdentityName'))]"
  ],
  "properties": {
    "roleDefinitionId": "[variables('roleVmContributor')]",
    "principalId": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('scriptIdentityName')), '2023-01-31').principalId]",
    "principalType": "ServicePrincipal"
  }
}
```

Where `roleVmContributor` is `[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', '9980e02c-c2be-4d73-94e8-173b1dc7cf3c')]`.

---

## Verifying the Result

After deployment, verify the registry keys are present with a single Run Command query in the Azure Portal:

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\FSLogix\Profiles" -ErrorAction SilentlyContinue |
  Select-Object Enabled, VHDLocations, VolumeType,
    DeleteLocalProfileWhenVHDShouldApply, FlipFlopProfileDirectoryName
```

Expected output:

```
Enabled                              : 1
VHDLocations                         : {\\<storageaccount>.file.core.windows.net\profiles}
VolumeType                           : VHDX
DeleteLocalProfileWhenVHDShouldApply : 1
FlipFlopProfileDirectoryName         : 1
```

The FSLogix transcript log at `C:\WindowsAzure\Logs\FSLogix\Configure-FSLogix_*.log` on the VM will also confirm the script executed and completed without errors.

The end-to-end validation is a user login via the [AVD web client](https://client.wvd.microsoft.com). After a successful login, check the Azure Files share in the portal — a `<username>` directory containing a `.vhdx` file should appear within a few seconds of the desktop loading.

---

## What to Watch Out For

### Do not use protectedSettings.script on Windows CSE

If you find examples online that use `protectedSettings.script` to pass an inline base64 script to a CSE resource, check whether the example is for Linux or Windows. The field is a Linux CSE feature. On Windows it is silently ignored, the ARM deployment succeeds, and you will spend time debugging why the script effects are absent.

### The CSE sequence number guard fires on fresh VMs

In ARM templates that deploy multiple VM extensions concurrently, the ARM engine may invoke the CSE handler multiple times during dependency resolution. Each invocation increments the internal sequence counter. By the time the actual `enable` command arrives, the handler may consider the sequence already processed and exit without running the script. This happens on fresh VMs with no prior deployment history and is not prevented by `forceUpdateTag`.

The only reliable mitigation is to avoid using CSE for post-provisioning configuration that must run exactly once after all other extensions complete. Use `deploymentScript` + `Invoke-AzVMRunCommand` instead — it has no sequence number state and a deterministic execution model.

### Invoke-AzVMRunCommand output is not streamed

`Invoke-AzVMRunCommand` is a synchronous call that blocks until the script completes on the VM. The output is returned in the result object, not streamed to the deployment script container in real time. For scripts that take longer than a few minutes, set the deploymentScript `timeout` appropriately — `PT15M` is sufficient for the FSLogix configuration script which completes in under ten seconds.

### The deployment script container needs outbound internet access

The deployment script runs in an Azure Container Instance that needs to reach the Azure management APIs to call `Invoke-AzVMRunCommand`. If your VNet has outbound internet access restricted (for example via an NSG or Azure Firewall without the required service tags), the container will fail to authenticate. The AVD session host VNet should have outbound access via a NAT Gateway for normal operation — the deployment script container uses a separate network context and is not affected by the session host VNet configuration.

---

## A Related Issue: Az PowerShell Module Broker Failures in Post-Deployment Scripts

The deploymentScript pattern solves FSLogix configuration inside the ARM template. But there is a class of AVD tasks that genuinely cannot run inside the template — the scaling plan host pool association being the most common example. These tasks require post-deployment scripts that run in the operator's local PowerShell session.

Those scripts have their own reliability problem worth documenting here.

### The symptom

The post-deployment script for scaling plan association uses `Get-AzADServicePrincipal` to look up the AVD service principal object ID before assigning RBAC roles. In some environments this cmdlet fails with a misleading error:

```
Your Azure credentials have not been set up or have expired, please run Connect-AzAccount
Method not found: 'Void Azure.Identity.Broker.SharedTokenCacheCredentialBrokerOptions..ctor(...)'
```

The credentials are valid — the same session successfully ran other Az cmdlets moments earlier. The real error is buried in the second line: a `Method not found` exception in `Azure.Identity.Broker`. This is a DLL conflict between the Az module version and the Azure Identity broker library. The specific cmdlets affected depend on which code path they hit internally — `Get-AzADServicePrincipal` and `Get-AzWvdScalingPlan` both hit the broken path in this environment, while other Az cmdlets do not.

### Why it is hard to diagnose

The error message says "credentials have not been set up" which sends you down an authentication troubleshooting path that leads nowhere. The actual problem is a version conflict in the Az module dependency chain, not authentication. Running `Connect-AzAccount` again does not help. The `-UseDeviceAuthentication` flag does not help. The credentials are fine.

### The solution: use the Azure CLI throughout

The Azure CLI (`az`) uses a completely separate authentication context and dependency stack. It is not affected by the Az PowerShell module broker issue. For post-deployment scripts that interact with AVD, Graph, and ARM resources, using `az` CLI calls throughout — rather than Az PowerShell cmdlets — produces scripts that work consistently regardless of the local Az module version.

The rewritten scaling plan association script uses `az rest` for all ARM API calls:

```powershell
# Resolve AVD service principal — works where Get-AzADServicePrincipal fails
$avdSp = az ad sp list `
    --filter "appId eq '9cdead84-a844-4324-93f2-b2e6bb768d07'" `
    --query "[0].{id:id,displayName:displayName}" -o json | ConvertFrom-Json

# Get scaling plan via REST — works where Get-AzWvdScalingPlan fails
$scalingPlan = az rest `
    --method GET `
    --uri "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.DesktopVirtualization/scalingPlans/$name?api-version=2023-09-05" | ConvertFrom-Json

# Associate via PATCH
$body = @{ properties = @{ hostPoolReferences = @(@{ hostPoolArmPath = $hostPoolId; scalingPlanEnabled = $true }) } } | ConvertTo-Json -Depth 5
$body | Set-Content -Path $tmp -Encoding UTF8
az rest --method PATCH --uri $scalingPlanUri --body "@$tmp" --headers "Content-Type=application/json"
```

The `az rest` approach also avoids the Az.DesktopVirtualization extension version problem — the CLI extension version 1.0.0 does not include scaling plan subcommands, but `az rest` bypasses the extension entirely and speaks directly to the ARM API.

### The broader pattern

If you are writing post-deployment scripts for AVD Marketplace offers and need them to work reliably across diverse customer environments, prefer `az` CLI calls over Az PowerShell cmdlets for any operations that touch:

- Entra ID / Graph (`az ad sp`, `az ad app`)
- AVD resources (`az rest` against the DesktopVirtualization API)
- Role assignments (`az role assignment`)

Reserve Az PowerShell for operations where it has a clear advantage and no known broker dependency issues, such as `Connect-MgGraph` for Microsoft Graph SDK operations where the CLI equivalent is more verbose.

---

## Summary

Configuring FSLogix registry keys from an ARM template is straightforward once you know which delivery mechanism to use. The Custom Script Extension is the intuitive choice but has two failure modes that are difficult to diagnose and produce no useful error output: silent `protectedSettings.script` support on Windows, and the sequence number guard that fires when the ARM engine invokes the CSE handler multiple times during dependency resolution.

The reliable pattern for in-template configuration is:

- Encode the FSLogix configuration script as a base64 string in a template variable
- Use a `deploymentScript` resource with a copy loop to run one container per session host
- Pass the base64 script and the FSLogix share UNC path as environment variables to the container
- Use `Invoke-AzVMRunCommand` inside the container to push and execute the script on the VM
- Grant the deployment script identity the **Virtual Machine Contributor** role on the resource group

For post-deployment scripts that run in the operator's local session, use Azure CLI calls throughout rather than Az PowerShell cmdlets for any operations that touch Entra ID, AVD resources, or role assignments. The Az module broker dependency issue is silent, environment-specific, and produces misleading authentication error messages that send you in the wrong direction.

Both patterns together give you a fully automated, reliably reproducible AVD deployment from a single Marketplace offer with minimal post-deployment manual intervention.

---

*Frametype Solutions LLC • Azure Virtual Desktop Practice • March 2026*
