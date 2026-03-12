---
layout: post
title: "IaC Deployment of AVD Observations"
subtitle: "Entra Kerberos - No Domain Services"
date: 2026-02-11
author: Matthew

series: "avd-iac-observations"
series_part: 1
series_title: "IaC Deployment of AVD Observations"

tags:
  - azure
  - avd
  - iac
---

# Azure Virtual Desktop + FSLogix
## Identity-Based File Access with Entra Kerberos

*A practical guide for IT Pros deploying cloud-native AVD*  
*Frametype Solutions LLC • March 2026*

---

## Introduction

If you are deploying Azure Virtual Desktop (AVD) in a cloud-only environment — meaning your users live entirely in Microsoft Entra ID with no on-premises Active Directory — one of the first questions you will run into is: how do I store user profiles?

The answer for most organizations is FSLogix, Microsoft's profile container solution that ships with every AVD license. FSLogix stores each user's profile as a VHD or VHDX file on a network share. In a traditional domain environment that share would be on a Windows file server joined to your AD domain. In a cloud-only AVD deployment, the recommended choice is Azure Files — Microsoft's fully managed SMB file share service.

Getting Azure Files and FSLogix to work together in a cloud-only environment requires a feature called Entra Kerberos. This post explains what Entra Kerberos is, why it matters, what the prerequisites are, and what to watch out for when you enable it.

---

## What is FSLogix and Why Does it Need a Network Share?

FSLogix solves a fundamental problem with Windows user profiles in a pooled virtual desktop environment. In a traditional desktop, your profile — your desktop, documents, browser favorites, application settings — lives on the local disk. That works fine when you always log into the same machine.

In AVD with pooled host pools, you might land on a different session host every time you connect. If your profile stays on the local disk of whichever VM you happened to get, your settings and files will be different every session. That is not acceptable for end users.

FSLogix solves this by storing your entire profile in a single VHDX file on a network share. When you log in, FSLogix attaches that VHDX as a virtual disk so Windows sees it as a local drive. When you log out, it detaches cleanly. From the operating system's perspective, your profile is always local — but it is actually following you across every session host in the pool.

For this to work, every session host needs to be able to reach the network share, and every user needs to have permission to read and write their own profile container on that share. That is where Azure Files and identity-based authentication come in.

---

## Why Azure Files?

In a cloud-only AVD deployment you do not have on-premises file servers. Azure Files gives you a fully managed SMB share in Azure that your session hosts can reach over the virtual network. There is no infrastructure to manage, no VM to patch, and it scales automatically.

Azure Files supports two authentication models for SMB access:

- **Storage account key** — a shared secret that gives full access to everything in the storage account. Simple to set up but not suitable for production because every user and every machine would share the same key, and rotating it requires updating every consumer.
- **Identity-based authentication** — users and computers authenticate using their Entra ID identity, and access is controlled by Azure RBAC role assignments on the share. This is the right model for FSLogix because each user gets a Kerberos ticket scoped to their own identity, and you can grant fine-grained permissions to specific users or groups.

Identity-based authentication is what Entra Kerberos enables for cloud-only environments.

---

## What is Entra Kerberos?

Kerberos is the authentication protocol that Windows has used for network resource access since Windows 2000. When you access a network share on a domain-joined machine, Windows uses Kerberos to obtain a service ticket from a domain controller that proves your identity to the file server. The file server accepts that ticket without you ever entering a password.

In a traditional on-premises environment the domain controller issues Kerberos tickets. In a cloud-only Entra ID environment there are no domain controllers — so who issues the tickets?

That is exactly what Entra Kerberos solves. When you enable Entra Kerberos on an Azure Files storage account, Microsoft registers a special enterprise application in your Entra ID tenant that acts as a Kerberos service principal for that storage account. Entra ID itself can then issue Kerberos service tickets for the storage account on behalf of your cloud-only users.

The result is that when a user logs into an Entra-joined AVD session host, Windows automatically requests a Kerberos ticket for the Azure Files share from Entra ID via a cloud KDC proxy. That ticket is used to authenticate the SMB connection to the share, and Azure Files validates the ticket and checks the user's RBAC permissions to decide what access to grant.

> **In practical terms:** Entra Kerberos lets your cloud-only Entra ID users authenticate to Azure Files the same way domain-joined users authenticate to on-premises file servers — seamlessly, without passwords, and without any on-premises infrastructure.

---

## How Entra Kerberos Works for Azure Files

Understanding the flow helps when things go wrong. Here is what happens at user login:

1. The user signs into their Entra-joined AVD session host using Windows Hello for Business or a password.
2. Windows detects that the FSLogix VHD location points to an Azure Files share (`\\<storageaccount>.file.core.windows.net\profiles`).
3. Windows requests a Kerberos Ticket Granting Ticket (TGT) from Entra ID via the cloud KDC proxy at `login.microsoftonline.com`.
4. Using the TGT, Windows requests a service ticket for the Azure Files share (`cifs/<storageaccount>.file.core.windows.net`).
5. Entra ID issues the service ticket because the storage account's enterprise application is registered as a valid Kerberos service principal in the tenant.
6. FSLogix uses the service ticket to authenticate the SMB connection to the share.
7. Azure Files validates the ticket and checks the user's RBAC permissions on the share.
8. If the user has the **Storage File Data SMB Share Contributor** role (or equivalent), the connection succeeds and FSLogix mounts the profile VHDX.

You can verify this is working by running `klist` in a PowerShell session on the session host after login. You should see two tickets: the TGT for `KERBEROS.MICROSOFTONLINE.COM` and a service ticket for `cifs/<storageaccount>.file.core.windows.net`.

---

## Requirements

Before enabling Entra Kerberos for Azure Files in your AVD environment, the following must be in place.

### Tenant and Identity Requirements

| Requirement | Details |
|---|---|
| **Entra ID tenant** | Users must be cloud-only Entra ID members. Hybrid (synced from AD) users require a different configuration. |
| **Entra-joined session hosts** | Session hosts must be Entra ID joined (not hybrid joined, not AD domain joined). The AADLoginForWindows extension must be installed. |
| **Enterprise app creation** | The account enabling Kerberos must have permission to create enterprise applications in Entra ID. A Global Administrator or Application Administrator role is required. |
| **Admin consent** | Admin consent must be granted for the storage account enterprise app to access Microsoft Graph (openid, profile, User.Read scopes). |
| **CA policy exclusion** | If your tenant has Conditional Access policies that require MFA, the storage account enterprise app must be excluded. Kerberos authentication does not support MFA challenges. |

### Azure Infrastructure Requirements

| Requirement | Details |
|---|---|
| **Storage account type** | General-purpose v2 (StorageV2). Standard or Premium SKU both work. Standard LRS is sufficient for PoC. |
| **Azure Files share** | An SMB-enabled file share on the storage account. Quota should be sized appropriately for your user count and profile sizes. |
| **Network connectivity** | Session hosts must be able to reach the storage account over port 445 (SMB). Outbound access via the virtual network is required. A NAT Gateway is recommended for cloud-only deployments without VPN or ExpressRoute. |
| **TLS 1.2** | The storage account must enforce a minimum TLS version of 1.2. This is the default for new accounts. |
| **HTTPS only** | The storage account must require HTTPS traffic. This is the default for new accounts. |

### Session Host Requirements

| Requirement | Details |
|---|---|
| **Windows 11 multi-session** | Entra Kerberos for Azure Files requires Windows 11 or Windows 10 21H2 or later. Windows 11 multi-session with Microsoft 365 Apps is the recommended image for AVD. |
| **FSLogix installed** | FSLogix Apps must be installed on the session host. FSLogix is included in all Microsoft 365 and Windows E3/E5 licenses and is pre-installed in the Microsoft-provided AVD marketplace images. |
| **Cloud Kerberos ticket retrieval** | The registry key `HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Kerberos\Parameters\CloudKerberosTicketRetrievalEnabled` must be set to `1`. This enables Windows to request Kerberos tickets from Entra ID. |
| **LoadCredKeyFromProfile** | The registry key `HKLM\Software\Policies\Microsoft\AzureADAccount\LoadCredKeyFromProfile` must be set to `1`. This is required for FSLogix to load the Kerberos credential from the user profile context. |
| **FSLogix registry config** | FSLogix must be configured with the UNC path to the Azure Files share via `HKLM\SOFTWARE\FSLogix\Profiles\VHDLocations`. |

### RBAC Requirements

| Role | Purpose |
|---|---|
| **Storage File Data SMB Share Contributor** | Assigned to users or groups who need to read and write their own profile containers. This is the minimum required role for FSLogix profile access. |
| **Storage File Data SMB Share Elevated Contributor** | Optional. Assign to administrators who need to manage all profile containers in the share. |
| **VM User Login or VM Administrator Login** | Required on the resource group containing the session hosts to allow Entra ID users to log into the VMs. |
| **Desktop Virtualization User** | Required on the AVD application group to allow users to see and launch the desktop. |

---

## What to Watch Out For

Entra Kerberos for Azure Files is a relatively new feature and has some rough edges worth knowing about before you start.

### The Enterprise App May Not Be Created Automatically

When you enable Entra Kerberos on a storage account, Azure is supposed to automatically create an enterprise application in your tenant named `[Storage Account] <storageaccountname>.file.core.windows.net`. This app is what Entra ID uses to issue Kerberos service tickets for the storage account.

In practice this registration is intermittent. Sometimes the app does not appear, or it appears with the wrong display name (missing the `[Storage Account]` prefix) and without the correct Kerberos service principal names (SPNs). A manually created app registration will not work because it lacks the required `cifs/` and `host/` SPNs that Windows needs to resolve when requesting a service ticket.

The reliable fix is to toggle Kerberos off and back on via the Azure CLI or portal until the correctly configured Microsoft-managed app appears:

```bash
az storage account update -g <rg> -n <storage> --enable-files-aadkerb false
az storage account update -g <rg> -n <storage> --enable-files-aadkerb true
```

Wait 60–120 seconds after re-enabling and check for the enterprise app using the Microsoft Graph PowerShell SDK. Look specifically for the `[Storage Account]` prefix in the display name — that is the indicator of a properly configured Microsoft-managed app.

### Toggling Kerberos Clears the Default Share Permission

If you set a default share permission on the storage account (described below) and then toggle Kerberos off and on, the default share permission is cleared. Always set the default share permission after completing the Kerberos toggle cycle, not before.

### Admin Consent Must Be Granted

After the enterprise app is created, an administrator must grant tenant-wide admin consent for the app to access Microsoft Graph with the `openid`, `profile`, and `User.Read` scopes. Without this, the Kerberos ticket issuance flow will fail silently.

You can grant admin consent using the Microsoft Graph PowerShell SDK or through the Azure portal under **Entra ID > Enterprise applications > [Storage Account] app > Permissions**.

### Conditional Access Policies Will Block Kerberos

Kerberos authentication to Azure Files does not go through an interactive sign-in flow, so it cannot satisfy an MFA challenge. If your tenant has Conditional Access policies that require MFA for all apps or all cloud apps, those policies will block the Kerberos ticket request and FSLogix will fail with an access denied error.

The fix is to explicitly exclude the storage account enterprise application from any CA policies that enforce MFA. This is a per-storage-account exclusion — if you have multiple storage accounts with Entra Kerberos enabled, each one needs to be excluded individually.

### Group Claims May Not Appear in the Token

In some cloud-only Entra ID tenants, the Kerberos service ticket issued for Azure Files may not include the user's group memberships as claims. Azure Files uses these claims to evaluate RBAC role assignments — if the groups are absent, even correctly assigned roles will not be evaluated and the user will receive an access denied error.

A practical workaround for PoC and development environments is to set a default share permission on the storage account:

```bash
az storage account update -g <rg> -n <storage> \
  --default-share-permission StorageFileDataSmbShareContributor
```

This grants the equivalent of Storage File Data SMB Share Contributor to all authenticated users, bypassing the group claim evaluation entirely. It is acceptable for a PoC but should be replaced with explicit RBAC assignments for production. Investigating why group claims are absent in your tenant (token configuration, group assignment settings) is the longer-term fix.

### The Kerberos Service Ticket Has a Short Lifetime

Unlike the TGT which is renewable for 7 days, the `cifs/` service ticket for Azure Files has a maximum lifetime of 1 hour and is not renewable. For most users this is not a problem because FSLogix mounts the profile at login and holds the connection open for the session. However if a session runs longer than an hour you may see connectivity issues if the ticket expires and is not automatically renewed. In practice Windows will re-request the service ticket automatically in most cases, but it is worth being aware of in long-running session environments.

---

## Summary

Entra Kerberos for Azure Files is the right solution for FSLogix profile storage in cloud-only AVD deployments. It eliminates the need for storage account keys, integrates cleanly with Entra ID identity, and provides a familiar Kerberos-based authentication experience that FSLogix is designed to work with.

The key things to get right are:

- Ensure the Microsoft-managed enterprise app is created correctly after enabling Kerberos — look for the `[Storage Account]` prefix.
- Grant admin consent for the enterprise app before testing.
- Exclude the enterprise app from any CA policies that require MFA.
- Set the default share permission after the Kerberos toggle cycle completes.
- Configure `CloudKerberosTicketRetrievalEnabled` and `LoadCredKeyFromProfile` on all session hosts.

With these pieces in place, FSLogix profiles will mount transparently at login using the user's Entra ID identity, with no passwords, no storage keys, and no on-premises infrastructure required.

---

*Frametype Solutions LLC • Azure Virtual Desktop Practice • March 2026*
