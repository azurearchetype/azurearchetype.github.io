---
layout: post
title: "IaC Deployment of AVD Observations"
subtitle: "- Host Pool Registration via ARM"
date: 2026-02-25
author: Matthew

series: "avd-iac-observations"
series_part: 1
series_title: "IaC Deployment of AVD Observations"

tags:
  - azure
  - avd
  - iac
---

# Azure Virtual Desktop + ARM Templates
## Deploying Host Pools and Registering Session Hosts Without a Script

*A practical guide for IT Pros packaging AVD as an Azure Marketplace managed application*  
*Frametype Solutions LLC • March 2026*

---

## Introduction

If you have deployed Azure Virtual Desktop using Bicep or PowerShell scripts, you are probably familiar with the two-phase deployment pattern: first create the host pool, then generate a registration token and pass it into the session host deployment. That pattern works well when a script is orchestrating the steps. It falls apart when you need to package everything into a single ARM template — for example, when publishing an AVD solution as an Azure Marketplace managed application.

This post covers the specific challenges that arise when deploying AVD host pools and registering session hosts inside a single `mainTemplate.json`, and the patterns that reliably solve them. If you have been hitting `BadRequest` errors on host pool or session host resources, or your session hosts are deploying but not registering, this is the post for you.

---

## The Two-Phase Problem

In a typical script-driven AVD deployment the workflow looks something like this:

1. Deploy the host pool with no registration token
2. Call `az desktopvirtualization hostpool update` to generate a registration token
3. Pass the token as a parameter into the session host deployment
4. Session hosts register to the host pool using the token during the DSC extension run

Each step waits for the previous one to complete. The script is the orchestrator — it holds state between phases and passes outputs forward as inputs.

In a single ARM template there is no orchestrator. All resources are declared in one file and ARM resolves the dependency graph. The challenge is that some of the things a script does between phases — generating the registration token, enabling Entra Kerberos, waiting for service principals to propagate — have no direct ARM equivalent. You have to know which patterns to use to replace them.

---

## Challenge 1: The Registration Token

### Why it is hard

A host pool registration token is not a property of the host pool resource — it is a time-limited secret generated on demand. In the original Bicep approach, `deploy.ps1` called `New-AzWvdRegistrationInfo` after the host pool was created and passed the resulting token into the session host deployment as a parameter.

In ARM there are two obvious candidates for retrieving the token inline:

- `listSecrets()` — the ARM function for retrieving secrets from resources
- `reference().registrationInfo.token` — reading the token from the host pool resource properties

Neither works reliably. `listSecrets()` on AVD host pools does not expose the registration token regardless of API version. `reference().registrationInfo.token` returns null because newer AVD APIs intentionally do not return the active token on a standard GET response — it must be retrieved via a dedicated retrieval operation. This behavior is confirmed in the Microsoft Learn documentation: [Add session hosts to a host pool](https://learn.microsoft.com/en-us/azure/virtual-desktop/add-session-hosts-host-pool).

### The solution: deploymentScripts

The reliable pattern is a `Microsoft.Resources/deploymentScripts` resource. This runs a PowerShell or Bash script inside an Azure Container Instance during the deployment, with access to the Az PowerShell module, and can output values back to the ARM template.

The script generates the registration token using `New-AzWvdRegistrationInfo` and writes it to `$DeploymentScriptOutputs`. The DSC extension on each session host then reads the token from the script output using `reference('script-name').outputs.registrationToken`.

```json
{
  "type": "Microsoft.Resources/deploymentScripts",
  "apiVersion": "2023-08-01",
  "name": "script-avd-regtoken",
  "location": "[parameters('location')]",
  "kind": "AzurePowerShell",
  "identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('scriptIdentityName'))]": {}
    }
  },
  "dependsOn": [
    "[resourceId('Microsoft.DesktopVirtualization/hostPools', variables('hostPoolName'))]",
    "[resourceId('Microsoft.Authorization/roleAssignments', guid(resourceGroup().id, variables('scriptIdentityName'), variables('roleDesktopVirtContributor')))]"
  ],
  "properties": {
    "azPowerShellVersion": "9.0",
    "retentionInterval": "PT1H",
    "timeout": "PT10M",
    "environmentVariables": [
      { "name": "ResourceGroupName", "value": "[resourceGroup().name]" },
      { "name": "HostPoolName",      "value": "[variables('hostPoolName')]" }
    ],
    "scriptContent": "
      $token = (New-AzWvdRegistrationInfo -ResourceGroupName $env:ResourceGroupName -HostPoolName $env:HostPoolName -ExpirationTime (Get-Date).AddDays(27)).Token
      $DeploymentScriptOutputs = @{}
      $DeploymentScriptOutputs['registrationToken'] = $token
    "
  }
}
```

A user-assigned managed identity is required to run the script. That identity needs the **Desktop Virtualization Contributor** role on the resource group so it can call `New-AzWvdRegistrationInfo`. The role assignment must be in the `dependsOn` of the deploymentScripts resource. It is worth knowing that the deploymentScripts service has built-in retry logic for sign-in: if the managed identity role assignment is granted in the same template, the service will retry authentication for up to 10 minutes at 10-second intervals to allow for RBAC propagation. This means a brief propagation delay will not fail the deployment outright, but having the role assignment in `dependsOn` is still correct practice. See [Use deployment scripts in ARM templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-script-template) for details.

The DSC extension then references the output:

```json
"registrationInfoToken": "[reference('script-avd-regtoken').outputs.registrationToken]"
```

### Important: keep the host pool clean

Do not include a `registrationInfo` block in the host pool resource itself. The host pool should be created with no registration configuration — just the host pool properties. The deploymentScripts resource handles token generation after the host pool exists. Putting `registrationInfo` in the host pool resource with an ARM-evaluated expiry time causes `BadRequest` errors because `utcNow()` is not valid in resource properties, only in parameter defaults.

---

## Challenge 2: Stale API Versions on AVD Resources

### Why it matters

AVD resources went through a long preview period and accumulated a number of preview API versions. If you are translating a Bicep template to ARM JSON, it is easy to carry across a preview API version that was valid when the Bicep was written but has since been deprecated or that causes silent `BadRequest` failures in ARM.

The three AVD resources most commonly affected are:

| Resource type | Symptom | Fix |
|---|---|---|
| `Microsoft.DesktopVirtualization/hostPools` | `BadRequest` with no useful message | Use `2023-09-05` or later |
| `Microsoft.DesktopVirtualization/applicationGroups` | `BadRequest` on create | Use `2023-09-05` or later |
| `Microsoft.DesktopVirtualization/workspaces` | `BadRequest` on create | Use `2023-09-05` or later |

Current supported API versions for all three resource types are listed in the ARM template reference on Microsoft Learn: [Microsoft.DesktopVirtualization/hostPools](https://learn.microsoft.com/en-us/azure/templates/microsoft.desktopvirtualization/hostpools). The API version mismatch is not obvious from the error message. If you are seeing `BadRequest` on AVD resources with no other useful detail, updating the API version to a current GA release is the first thing to try.

---

## Challenge 3: Resource Names Containing Dots

### The Bicep vs ARM JSON difference

In Bicep, variables can contain any string and the Bicep compiler handles the distinction between using a variable as a resource name versus using it in a string expression. In ARM JSON that distinction falls on you.

AVD resource names cannot contain dots. Host pool names, application group names, and workspace names must use only letters, numbers, and hyphens. If you have a variable in your ARM template that is used for both a network suffix (which may legitimately contain dots, for example `10.120`) and an AVD resource name, you will get a validation error or a silent failure.

The fix is to use separate variables for network naming and AVD resource naming. Never reuse a variable that contains dots in an AVD resource name context. This is a common translation error when converting Bicep to ARM JSON because Bicep's string interpolation handles it transparently.

```json
"variables": {
  "vnetSuffix":    "10.120",
  "hostPoolName":  "[concat('hp-avd-', parameters('environment'), '-', parameters('location'), '-01')]"
}
```

Note that `hostPoolName` does not reference `vnetSuffix` — it builds the name independently from safe components.

---

## Challenge 4: The Scaling Plan RBAC Race Condition

### What happens

The AVD scaling plan needs to be associated with the host pool, and it requires two roles to be assigned to the AVD service principal (`9cdead84-a844-4324-93f2-b2e6bb768d07`) at **subscription scope**:

- **Desktop Virtualization Power On Off Contributor** — allows AVD to manage the power state of session hosts
- **Desktop Virtualization Virtual Machine Contributor** — allows AVD to create, delete, update, start, and stop session hosts

Both roles must be assigned at subscription scope, not resource group scope. Microsoft's documentation is explicit on this: assigning either role at any level lower than the subscription — such as the resource group, host pool, or virtual machine — prevents autoscale from working properly. See [Create and assign an autoscale scaling plan](https://learn.microsoft.com/en-us/azure/virtual-desktop/autoscale-create-assign-scaling-plan) and [Assign Azure RBAC roles to the Azure Virtual Desktop service principals](https://learn.microsoft.com/en-us/azure/virtual-desktop/service-principal-assign-roles) for details.

These role assignments are also created in the ARM template, and this is where the timing problem arises. In a script-driven deployment you can wait for role propagation before associating the scaling plan. In ARM there is no equivalent wait mechanism for RBAC propagation. Even with a `dependsOn` pointing to the role assignment resources, the roles may not be active in the authorization layer by the time the scaling plan resource tries to use them. The result is a scaling plan that deploys successfully but fails silently when it tries to associate with the host pool, or a `BadRequest` on the scaling plan resource itself.

### The solution: defer the association

Remove `hostPoolReferences` from the scaling plan resource in the ARM template. Deploy the scaling plan without any host pool association. Add connecting the scaling plan to the host pool as a documented post-deployment step.

```json
"properties": {
  "friendlyName": "AVD Scaling Plan",
  "timeZone": "Pacific Standard Time",
  "hostPoolType": "Pooled",
  "schedules": [ ... ]
}
```

The scaling plan schedules are fully configured at deploy time. Only the host pool association is deferred. This is a one-time manual step that takes seconds in the portal and eliminates an entire class of intermittent deployment failures.

---

## Challenge 5: Storage and Monitoring Dependency Chains

ARM has a well-known requirement that child resources must be declared explicitly if you need to control their provisioning order. Two areas catch people out in AVD deployments.

### Azure Files file share

You cannot create a file share directly after a storage account in ARM without an intermediate `fileServices` resource. The dependency chain must be:

```
Microsoft.Storage/storageAccounts
  → Microsoft.Storage/storageAccounts/fileServices (default)
    → Microsoft.Storage/storageAccounts/fileServices/shares
```

Bicep handles this implicitly. ARM JSON requires the `fileServices/default` resource to be declared explicitly. Without it, the file share creation fails with a `ResourceNotFound` error even though the storage account exists.

### Log Analytics and Data Collection Rules

Data Collection Rules that reference a Log Analytics Workspace as a destination can fail if the workspace tables have not finished provisioning. The workspace itself may report as succeeded while the underlying table schema is still being created.

If you are seeing DCR creation errors related to table availability, insert a `deploymentScripts` delay resource between the workspace and the DCR:

```json
{
  "type": "Microsoft.Resources/deploymentScripts",
  "name": "law-table-wait",
  "dependsOn": [ "[resourceId('Microsoft.OperationalInsights/workspaces', variables('lawName'))]" ],
  "properties": {
    "azPowerShellVersion": "9.0",
    "retentionInterval": "PT1H",
    "scriptContent": "Start-Sleep -Seconds 120"
  }
}
```

The DCR then depends on this script rather than directly on the workspace. One hundred and twenty seconds is sufficient for table provisioning in all regions tested.

---

## Putting It All Together

The dependency chain for a complete AVD ARM deployment that avoids all of these issues looks like this:

```
Networking (VNet, NAT Gateway)
  → Storage Account
    → fileServices/default
      → File Share (profiles)
  → Log Analytics Workspace
    → law-table-wait (deploymentScript, 120s)
      → Data Collection Rule
        → DCR Association (per VM)
  → Managed Identity (for deploymentScripts)
  → Host Pool
    → Role Assignment (Desktop Virtualization Contributor → Managed Identity)
      → script-avd-regtoken (deploymentScript)
        → Session Host VMs
          → AADLoginForWindows extension
            → DSC extension (uses registration token from script output)
              → script-fslogix-{n} (deploymentScript, FSLogix registry config)
  → Application Group (depends on Host Pool)
  → Workspace (depends on Application Group)
  → Scaling Plan (no hostPoolReferences — associated post-deployment)
  → RBAC assignments (AVD users/admins, VM login roles, Storage SMB roles)
```

Each layer waits for the previous one. The deploymentScripts resources act as synchronization points where ARM's parallel execution model needs to be serialized.

> **Note on FSLogix configuration:** Earlier versions of this architecture used a Custom Script Extension (CSE) to write the FSLogix registry keys to each session host. CSE was replaced with a `deploymentScript` resource that uses `Invoke-AzVMRunCommand` to push the configuration into each VM from outside. The reasons for this change — and the specific CSE failure modes that led to it — are covered in detail in [the third post in this series](#).

---

## What to Watch Out For

### deploymentScripts requires a managed identity and contributor role

The deploymentScripts resource runs in an Azure Container Instance and needs an identity to authenticate to Azure. A user-assigned managed identity with Desktop Virtualization Contributor on the resource group is the minimum required. Do not use a system-assigned identity on the deploymentScripts resource itself — system-assigned identities on deploymentScripts resources are not supported. This is confirmed in the Microsoft Learn documentation: [Use deployment scripts in ARM templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-script-template).

### ARM expressions cannot be inline in scriptContent

If your deploymentScripts `scriptContent` needs values from ARM expressions (resource names, resource group names, etc.) you cannot embed those expressions directly in the script string. ARM evaluates expressions in the template but the script runs later in a container — by that point the expression has already been evaluated to a string and is no longer accessible.

The correct pattern is to pass ARM-evaluated values as `environmentVariables`:

```json
"environmentVariables": [
  { "name": "HostPoolName", "value": "[variables('hostPoolName')]" }
]
```

Then reference them in the script as `$env:HostPoolName`. Never mix ARM expressions and PowerShell syntax in the same `scriptContent` string.

### retentionInterval controls how long the script container is kept

Set `retentionInterval` to `PT1H` for production deployments. The container and its logs are retained for that period after the script completes, which gives you time to retrieve logs if something goes wrong. After the retention period the container is automatically deleted. The script output (including the registration token) is retained in ARM state regardless of the container lifecycle.

---

## Summary

Packaging an AVD deployment as a single ARM template requires replacing several script-driven orchestration steps with ARM-native patterns. The key substitutions are:

- Replace script-generated registration tokens with a `deploymentScripts` resource that calls `New-AzWvdRegistrationInfo` and outputs the token back to the template
- Keep the host pool resource clean — no `registrationInfo` block
- Use current GA API versions (`2023-09-05` or later) on all AVD resources
- Use separate variables for network naming and AVD resource naming — never use a dot-containing variable in an AVD resource name
- Declare `fileServices/default` explicitly between the storage account and the file share
- Add a deploymentScripts delay between Log Analytics Workspace and Data Collection Rule creation
- Remove `hostPoolReferences` from the scaling plan and associate it with the host pool post-deployment
- Use `deploymentScript` + `Invoke-AzVMRunCommand` rather than Custom Script Extension for post-provisioning VM configuration

With these patterns in place, a complete cloud-native AVD deployment — host pool, session hosts, FSLogix storage, monitoring, scaling plan, and RBAC — deploys cleanly from a single `mainTemplate.json` with no manual intervention required during the deployment itself.

---

*Frametype Solutions LLC • Azure Virtual Desktop Practice • March 2026*
