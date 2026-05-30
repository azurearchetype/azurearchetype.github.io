---
layout: post
title: "AVM Adoption as a CAF, WAF, and Zero Trust Alignment Signal"
subtitle: "What Choosing Azure Verified Modules Says About Your Azure Practice"
date: 2026-05-29
author: Matthew

series:
series_part:
series_title:

tags:
  - azure
  - iac
  - bicep
  - avm

excerpt: "Azure Verified Modules isn't just a convenience library. It's Microsoft's formal encoding of what CAF, WAF, and Zero Trust-aligned Bicep looks like, and adopting it is a signal about how you build."
---

<svg width="100%" viewBox="0 0 680 200" xmlns="http://www.w3.org/2000/svg" role="img">
<title>AVM as CAF, WAF, and Zero Trust alignment signal</title>
<desc>Three framework layers stacked above Azure Verified Modules as their shared foundation, with annotated Bicep code on the right</desc>
<rect width="680" height="200" fill="#0A1F35" rx="12"/>
<g font-family="monospace" font-size="8" fill="#4A8ABF">
  <text x="400" y="24">module 'kv' 'br/public:avm/res/key-vault/vault:0.3.0' = {</text>
  <text x="400" y="38">  params: {</text>
  <text x="400" y="52">    enablePurgeProtection: true</text>
  <text x="400" y="66">    publicNetworkAccess: 'Disabled'</text>
  <text x="400" y="80">    diagnosticSettings: diagnosticSettings</text>
  <text x="400" y="94">    roleAssignments: rbacAssignments</text>
  <text x="400" y="108">    skuName: 'standard'</text>
  <text x="400" y="122">  }</text>
  <text x="400" y="136">}</text>
</g>
<rect x="50" y="22" width="48" height="24" rx="4" fill="#163D6A" stroke="#3A9EE8" stroke-width="0.5"/>
<text x="74" y="38" font-family="monospace" font-size="11" fill="#5BB8F5" text-anchor="middle" font-weight="600">CAF</text>
<text x="108" y="38" font-family="monospace" font-size="9" fill="#5BB8F5" opacity="0.7">naming · landing zones · governance</text>
<line x1="74" y1="46" x2="74" y2="58" stroke="#2A7BC8" stroke-width="0.5" opacity="0.6"/>
<rect x="50" y="58" width="48" height="24" rx="4" fill="#0d4a38" stroke="#4CAF7D" stroke-width="0.5"/>
<text x="74" y="74" font-family="monospace" font-size="11" fill="#5ECAA0" text-anchor="middle" font-weight="600">WAF</text>
<text x="108" y="74" font-family="monospace" font-size="9" fill="#5BB8F5" opacity="0.7">reliability · security · cost · ops · perf</text>
<line x1="74" y1="82" x2="74" y2="94" stroke="#2A7BC8" stroke-width="0.5" opacity="0.6"/>
<rect x="50" y="94" width="84" height="24" rx="4" fill="#342160" stroke="#9B8BE8" stroke-width="0.5"/>
<text x="92" y="110" font-family="monospace" font-size="11" fill="#B0A0F0" text-anchor="middle" font-weight="600">Zero Trust</text>
<text x="144" y="110" font-family="monospace" font-size="9" fill="#5BB8F5" opacity="0.7">verify · least privilege · assume breach</text>
<line x1="74" y1="118" x2="74" y2="132" stroke="#3A9EE8" stroke-width="0.75"/>
<line x1="92" y1="118" x2="92" y2="132" stroke="#3A9EE8" stroke-width="0.75"/>
<rect x="40" y="132" width="300" height="44" rx="4" fill="#1A4575" stroke="#3A9EE8" stroke-width="0.75"/>
<text x="190" y="150" font-family="monospace" font-size="13" fill="#5BB8F5" text-anchor="middle" font-weight="700">Azure Verified Modules</text>
<text x="190" y="167" font-family="monospace" font-size="9" fill="#3A9EE8" text-anchor="middle" opacity="0.8">the specification beneath the signal</text>
<text x="40" y="193" font-family="monospace" font-size="8" fill="#3A9EE8" opacity="0.3">azurearchetype.com</text>
</svg>

*Infrastructure as Code · Azure Verified Modules · CAF · WAF · Zero Trust · Bicep*

**Matthew Collins** · MCT · Azure Infrastructure · IaC · AI Foundry · 6 min read

---

> When I'm reviewing an Azure IaC codebase and I see Azure Verified
> Modules as the foundation, I'm not just looking at a module choice. I'm looking
> at a signal about how deeply that organization understands what Microsoft is actually trying
> to say about how Azure should be deployed.

---

There are certain terms that get repeated constantly in Azure architecture conversations: *CAF-aligned*, *WAF-aligned*, and *Zero Trust*. They show up in partner advanced specializations, Marketplace certified software designations, and client architectural review meetings. They are used loosely enough that they sometimes seem meaningless, a marketing overlay applied to infrastructure that may or may not actually follow the principles these frameworks describe.

Azure Verified Modules changes that. AVM is Microsoft's method to encode what CAF, WAF, and Zero Trust alignment actually looks like at the resource level, in code rather than in documentation. If you understand what AVM is doing and why, you understand why adopting it is a genuine alignment signal and not just an easy button for deploying infrastructure via code.

---

## What AVM Actually Is

Azure Verified Modules is Microsoft's official library of Bicep and Terraform modules, published and maintained by the Azure product engineering teams. It replaces older community patterns like CARML (Common Azure Resource Modules Library) and establishes a specification that module authors must follow for a module to carry the Verified badge.

That specification is the important part. AVM modules are not just convenience wrappers around `az` CLI commands. They are opinionated implementations that:

- Default to secure configurations (managed identity, no public network access where applicable, private endpoints wired in)
- Include diagnostic settings and monitoring outputs by default
- Follow CAF-standard resource naming abbreviations
- Are designed to compose: they expose consistent outputs so that one module can feed cleanly into another
- Are tested against real Azure deployments, not just template validation

In short, an AVM module for a Key Vault doesn't just create a Key Vault. It creates a Key Vault the way Microsoft's own architects would create one if they were following their own published guidance.

---

## Where CAF Alignment Lives in AVM

The Cloud Adoption Framework's IaC guidance focuses on three things that are often treated as separate concerns: resource naming and tagging, landing zone structure, and governance at scale through Azure Policy. AVM touches all three.

**Naming conventions** are encoded directly. AVM's naming helper modules follow CAF abbreviations: `kv-` for Key Vault, `rg-` for resource group, `st` for storage, so a codebase built on AVM produces resource names that are CAF-compliant by default rather than by discipline.

**Composability for landing zones** is baked into the module design. The CAF landing zone pattern is fundamentally about composing building blocks: connectivity (hub/spoke or Virtual WAN), identity (Entra ID, Privileged Identity Management), management (Log Analytics, Defender for Cloud), and workload landing zones. AVM modules are designed to produce outputs that wire directly into that composition pattern. You're not fighting the framework to assemble a landing zone; you're assembling the pieces it was designed to produce.

**Policy alignment** comes through the WAF side of the specification, but the CAF governance layer, particularly around tagging strategy and resource hierarchy, is reflected in how AVM modules accept and pass through tag parameters across resource hierarchies.

---

## Where WAF Alignment Lives in AVM

The Well-Architected Framework's five pillars (Reliability, Security, Cost Optimization, Operational Excellence, and Performance Efficiency) each have a footprint in AVM module specifications.

**Reliability** shows up in zone-redundancy defaults and health probe configurations. AVM modules for compute and networking resources surface availability zone parameters prominently and provide sane defaults rather than leaving zone configuration as an afterthought.

**Security** is the most visible pillar in AVM. The secure-by-default principle means public network access is disabled where it can be, managed identity is the authentication default, and RBAC assignments are explicit rather than relying on legacy access key patterns. A Key Vault module that enables purge protection and soft delete by default is making a WAF Security decision on your behalf.

**Operational Excellence** is reflected in the diagnostic settings pattern. AVM modules accept a `diagnosticSettings` parameter block that routes logs and metrics to your Log Analytics workspace. In a codebase without AVM, enabling diagnostics on every resource is a discipline problem: it happens when someone remembers to add it. In an AVM-based codebase, it's a parameter you pass, consistently, across every resource type.

**Cost Optimization** and **Performance Efficiency** are less prescriptive at the module level; these pillars require context that a generic module can't have. But AVM surfaces SKU choices and right-sizing parameters in a way that forces an explicit decision rather than defaulting to the largest or cheapest option silently.

---

## Where Zero Trust Alignment Lives in AVM

Zero Trust is not a product or a feature set. It is a security model built on three principles: verify explicitly, use least privilege access, and assume breach. In an Azure IaC context, those principles translate directly into resource configuration decisions that AVM modules make by default.

**Verify explicitly** is reflected in AVM's identity defaults. Every module that can use managed identity does. Shared access keys and connection strings are not the default authentication path; they are explicitly surfaced as override options that require a conscious decision to enable. You do not accidentally deploy a storage account that authenticates via key because the module's secure default points you toward identity-based access.

**Use least privilege** shows up in how AVM modules handle RBAC. Rather than granting broad Contributor access at a resource group level, AVM modules surface granular data-plane role assignments: Storage Blob Data Reader, Key Vault Secrets User, Service Bus Data Sender. The module design pushes you toward the minimum required permission surface rather than the most convenient one.

**Assume breach** is encoded in the observability and resilience defaults. Diagnostic settings enabled by default means audit logs are flowing from day one, not after an incident triggers a scramble to enable logging retroactively. Private endpoints, network access restrictions, and soft delete with purge protection are all assume-breach configurations: they limit blast radius, preserve forensic data, and make recovery possible.

The distinction between WAF Security and Zero Trust in AVM is worth naming. WAF Security covers the broad security posture of a workload: encryption, key management, access control, threat detection. Zero Trust is specifically the trust model governing how identity and network position interact. An AVM-based codebase can satisfy WAF Security requirements through correct configuration choices. Zero Trust alignment requires the identity-first, least-privilege, assume-breach posture to be the default, not the exception. AVM's module specification makes it the default.

---

## The Architect's Signal in the Marketplace

AVM adoption as a market signal is worth examining, particularly for Azure SDCs building Marketplace offers.

Before AVM was formalized, early IaC adopters were arriving at similar patterns independently: modular Bicep, consistent outputs for composition, diagnostics by default, managed identity everywhere, CAF naming. The discipline to build this way came from deep engagement with CAF, WAF, and the Azure architecture review process. You had to already understand the frameworks to build in alignment with them.

<svg width="100%" viewBox="0 0 680 260" xmlns="http://www.w3.org/2000/svg" role="img">
<title>AVM as a dependency versus AVM as a specification</title>
<desc>Two-column contrast: left shows AVM as a simple import, right shows understanding of what each module parameter encodes</desc>
<rect width="680" height="260" fill="#0A1F35" rx="12"/>
<rect x="20" y="16" width="300" height="228" rx="6" fill="#0D2340" stroke="#1A4060" stroke-width="0.5"/>
<rect x="360" y="16" width="300" height="228" rx="6" fill="#0D2340" stroke="#2A5C8A" stroke-width="0.5"/>
<text x="170" y="44" font-family="monospace" font-size="11" fill="#3A7AA0" text-anchor="middle">AVM as a dependency</text>
<line x1="30" y1="52" x2="310" y2="52" stroke="#1A4060" stroke-width="0.5"/>
<g font-family="monospace" font-size="9" fill="#4A8ABF">
  <text x="36" y="74">// bicepconfig.json</text>
  <text x="36" y="90">{</text>
  <text x="36" y="106">  "moduleAliases": {</text>
  <text x="36" y="122">    "br/public": {</text>
  <text x="36" y="138">      "registry":</text>
  <text x="36" y="154">       "mcr.microsoft.com"</text>
  <text x="36" y="170">    }</text>
  <text x="36" y="186">  }</text>
  <text x="36" y="202">}</text>
</g>
<text x="170" y="228" font-family="monospace" font-size="9" fill="#3A7AA0" text-anchor="middle">The import. That is all.</text>
<line x1="335" y1="30" x2="335" y2="230" stroke="#1A3A5C" stroke-width="0.5"/>
<rect x="322" y="118" width="26" height="18" rx="3" fill="#0A1F35"/>
<text x="335" y="131" font-family="monospace" font-size="9" fill="#3A9EE8" text-anchor="middle">vs</text>
<text x="510" y="44" font-family="monospace" font-size="11" fill="#5BB8F5" text-anchor="middle">AVM as a specification</text>
<line x1="370" y1="52" x2="650" y2="52" stroke="#2A5C8A" stroke-width="0.5"/>
<text x="376" y="74" font-family="monospace" font-size="9" fill="#3A6A8A">// what each default encodes:</text>
<g font-family="monospace" font-size="9">
  <text x="376" y="96" fill="#5BB8F5">enablePurgeProtection</text>
  <text x="530" y="96" fill="#4A7099">→</text>
  <text x="546" y="96" fill="#5ECAA0">WAF · Security</text>
  <text x="376" y="118" fill="#5BB8F5">publicNetworkAccess</text>
  <text x="530" y="118" fill="#4A7099">→</text>
  <text x="546" y="118" fill="#B0A0F0">Zero Trust · verify</text>
  <text x="376" y="140" fill="#5BB8F5">diagnosticSettings</text>
  <text x="530" y="140" fill="#4A7099">→</text>
  <text x="546" y="140" fill="#5ECAA0">WAF · Ops Excellence</text>
  <text x="376" y="162" fill="#5BB8F5">roleAssignments</text>
  <text x="530" y="162" fill="#4A7099">→</text>
  <text x="546" y="162" fill="#B0A0F0">Zero Trust · least privilege</text>
  <text x="376" y="184" fill="#5BB8F5">skuName: 'standard'</text>
  <text x="530" y="184" fill="#4A7099">→</text>
  <text x="546" y="184" fill="#5ECAA0">WAF · Cost Optimization</text>
</g>
<text x="510" y="228" font-family="monospace" font-size="9" fill="#3A9EE8" text-anchor="middle">You know why each default exists.</text>
</svg>

AVM formalizes that understanding into a library, which means two things are now true simultaneously:

First, an architect who adopts AVM without understanding why is getting alignment as a side effect, which is genuinely good for the Azure ecosystem. Defaults matter at scale.

Second, an architect who understands *why* AVM is designed the way it is, and can explain what WAF pillar a particular module default is serving and what trade-off it's making, is demonstrating something that a library dependency alone cannot. That architect is aligned with Microsoft's direction, not just dependent on the library.

For SDCs building on AVM, that alignment becomes a commercial differentiator. An offer built on AVM modules carries verifiable CAF, WAF, and Zero Trust alignment by construction, not as a line item on a co-sell submission checklist, but as an architectural property of the codebase itself. In a Marketplace where many offers position around framework alignment, demonstrating it through the code is what separates a proof from a claim.

---

## What This Means in Practice

If you're building or reviewing Azure IaC today, a few practical implications follow from this framing:

**Use AVM as a specification, not just a dependency.** Read the module specifications at `azure.github.io/Azure-Verified-Modules`. Understand what each parameter is doing and which WAF pillar it serves. The library is the implementation; the spec is the knowledge.

**Treat AVM non-compliance as a gap to explain, not ignore.** There will be cases where your environment requires deviation from AVM defaults: specific network topologies, compliance requirements, legacy constraints. Document those deviations and their reasoning. An architecture that deviates from AVM with documented rationale is stronger than one that follows AVM without understanding it.

**Let AVM alignment drive your marketing narrative.** If your Marketplace offer or Azure managed service is built on AVM, that is a concrete, verifiable CAF, WAF, and Zero Trust alignment claim, not a positioning statement. Make it explicit in your offer listing and marketing materials.

---

## The Longer View

Microsoft's direction for Azure IaC is unambiguous: Bicep is the authoring format, AVM is the module standard, and CAF, WAF, and Zero Trust are the frameworks that ground both. Those who are building in alignment with that direction now are building toward where the Microsoft Partner programs, the Marketplace certification requirements, and the architecture review processes are heading.

AVM adoption is a signal. Understanding why it's designed the way it is turns that signal into something more valuable: a demonstrable, articulable alignment with how Microsoft thinks Azure should be built.

*Stay in the flow, avoid the noise.*

---

#AzureVerifiedModules #Bicep #InfrastructureAsCode #CloudAdoptionFramework #WellArchitected #ZeroTrust #AzureArchitecture #