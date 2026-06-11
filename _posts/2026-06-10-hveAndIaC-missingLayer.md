---
layout: post
title: "The Missing Layer in Microsoft's Hypervelocity Engineering Framework"
subtitle: "HVE Core governs your code. Azure IaC governs the platform your agents run on."
date: 2026-06-10
author: Matthew

series:
series_part:
series_title:

tags:
  - azure
  - iac
  - bicep
  - avm
  - github-copilot
  - agentic-ai
  - hve

excerpt: "HVE Core's RPI methodology solves context contamination at the code layer. But for Azure practitioners building AI workloads, there is a layer below the application code that the framework doesn't fully reach. Azure IaC is HVE applied one layer down."
---

<img src="/assets/img/azureArchetype-header-1200x628.png" alt="The Missing Layer — HVE Core and Azure IaC stack diagram" style="width:100%; border-radius:8px; margin-bottom:1.5rem;" />

*Azure Infrastructure · IaC · Bicep · AVM · GitHub Copilot · Agentic AI · HVE*

**Matthew Collins** · MCT · Azure Infrastructure · IaC · AI Foundry · 7 min read

---

> Yesterday I attended a Microsoft Partner Skilling Hub workshop on the Frontier Transformation
> Engineer. The HVE Core toolbox for GitHub Copilot is compelling. But if you build
> AI workloads on Azure, there is a layer the toolbox conversation doesn't fully reach.

---

## Table of Contents

- [The Problem HVE Core Is Actually Solving](#the-problem-hve-core-is-actually-solving)
- [What the Frontier Transformation Engineer Actually Is](#what-the-frontier-transformation-engineer-actually-is)
- [Stage 1 Has a Different Meaning for Azure Practitioners](#stage-1-has-a-different-meaning-for-azure-practitioners)
- [Azure IaC Is HVE Applied One Layer Down](#azure-iac-is-hve-applied-one-layer-down)
- [What a Complete FTE Stack Looks Like on Azure](#what-a-complete-fte-stack-looks-like-on-azure)
- [The Insight I Took Away from the Workshop](#the-insight-i-took-away-from-the-workshop)

---

Yesterday I attended Microsoft's Frontier Transformation Engineer workshop from the Partner Skilling Hub. The session introduced HVE Core, an open-source Microsoft framework ([github.com/microsoft/hve-core](https://github.com/microsoft/hve-core)) that brings convention-driven AI workflows to GitHub Copilot. The toolbox is substantial: 49 specialized Copilot agents, 102 auto-applied coding instructions, 63 reusable prompts, and 11 skills, all packaged and available as either a VS Code extension or a GitHub Copilot CLI plugin.

The core methodology is called RPI: Research → Plan → Implement → Review. It sounds like a mnemonic, but the reasoning behind it is sharp.

---

## The Problem HVE Core Is Actually Solving

The HVE Core documentation articulates the failure mode they are addressing with clarity:

> *"Ask for a Terraform module for Azure IoT and you'll get 2000 lines of code — missing dependencies, wrong variable names, outdated patterns, and breaks existing infrastructure."*

That is not a hypothetical. Anyone who has worked with AI coding assistants on real infrastructure code has encountered this. The root cause is what the HVE team calls "context contamination: AI writes first and thinks never." When you ask it to build a feature, it generates plausible output immediately. Plausible and correct are not the same thing.

RPI fixes this by separating AI work into four distinct phases, each handled by a specialized Copilot agent with a constrained role:

- **Task Researcher**: investigates your codebase and external sources, produces verified findings with citations
- **Task Planner**: transforms research into a sequenced implementation plan with defined success criteria
- **Task Implementor**: executes the plan methodically, following patterns discovered during research
- **Task Reviewer**: validates implementation against specifications and identifies gaps

The mechanism that makes this work is phase isolation. When you clear context between phases, the implementation session does not carry forward the assumptions from research. It only has the documented artifacts: verified findings, explicit decisions, cited evidence. The constraint changes what AI optimizes for. Instead of generating plausible code, it generates traceable code.

This is the shift from prompt engineering to context engineering. Prompt engineering asks: *what do I say to get better output?* Context engineering asks: *what does the AI know, and what is it prevented from knowing, at each stage of the workflow?* HVE Core's phase isolation is context engineering in practice. The AI is not smarter. Its context is more deliberately designed.

This is a genuine engineering contribution from Microsoft. It addresses a real problem with a principled solution.

---

## What the Frontier Transformation Engineer Actually Is

The workshop positioned the Frontier Transformation Engineer not as a narrow specialist but as a practitioner who operates across the full AI delivery stack. Map that against the HVE Core project lifecycle (which runs nine stages from Setup through Operations) and the FTE role touches almost all of them:

- **Discovery and Product Definition** (Stages 2-3): architecture, requirements, security planning
- **Implementation and Review** (Stages 6-7): writing code and validating it
- **Delivery and Operations** (Stages 8-9): shipping reliably and keeping it running

That breadth is intentional. The AI transformation era does not reward practitioners who can write AI-augmented code but cannot reason about the system it runs on.

Which brings me to the layer I was thinking about when the workshop ended.

---

## Stage 1 Has a Different Meaning for Azure Practitioners

The HVE lifecycle opens at Stage 1: Setup. The tooling listed for this stage is the HVE installer skill and the memory agent, appropriate for configuring your Copilot environment and establishing project context.

But for anyone building AI workloads on Azure, Stage 1 has a second meaning that the framework does not explicitly address: you need to provision the infrastructure your AI application will run on before you write a single line of application code.

For a production Azure AI workload, that infrastructure typically includes:

- Azure AI Foundry project and model deployments
- Azure AI Search for retrieval-augmented generation
- Azure Container Apps or App Service for the application host
- Azure Key Vault for secrets management
- Networking: private endpoints, VNet integration, DNS configuration
- A management baseline: Log Analytics, Diagnostic Settings, Managed Identity

If you provision these resources manually through the portal, you have introduced the same class of problem at the infrastructure layer that RPI solves at the code layer. The resources drift. The configuration is undocumented. The environment cannot be reproduced. And Stage 9 (Operations) becomes a question of hoping the infrastructure holds rather than knowing how it was built.

---

## Azure IaC Is HVE Applied One Layer Down

The principles that make HVE Core work for application code apply directly to infrastructure, and Azure Verified Modules (AVM) is where those principles are implemented for Azure resources.

<img src="/assets/img/azureArchetype-scrollstop-1200x800.png" alt="HVE Core RPI phases mapped to AVM Bicep equivalents" style="width:100%; border-radius:8px; margin: 1.5rem 0;" />

The parallel is structural, not metaphorical:

| HVE Core Principle | AVM Bicep Equivalent |
|---|---|
| Task Researcher: verify existing patterns, cite sources | AVM module specification: published, tested, WAF-aligned resource definitions |
| Task Planner: sequence work before implementing | Bicep orchestrator: declarative composition of modules before any resource is created |
| Task Implementor: follow the plan, no ad-hoc decisions | AVM module execution: opinionated, convention-following, no manual steps |
| Task Reviewer: validate implementation against spec | `az deployment sub what-if`: validate the full change set before applying |

AVM modules do for Azure resources what HVE Core's coding instructions do for application code: they encode the opinionated, validated, Microsoft-standard pattern so you are not inventing conventions from scratch. An AVM module for Azure AI Search does not just create a search instance. It creates one with private endpoints wired in, managed identity enabled, diagnostic settings connected to Log Analytics, and naming that follows CAF abbreviations, the way Microsoft's own architects would build it if they were following their own published guidance.

The Bicep orchestrator is the implementation plan: declare what you need, compose the modules, review the what-if output before a single resource changes. Context is committed to files, not trapped in someone's memory of what they clicked.

The connection to HVE's RPI phases is not coincidental. Both frameworks are responses to the same underlying problem: AI and human practitioners alike make bad decisions when they conflate research with implementation, or when patterns exist only in their heads rather than in governed artifacts.

---

## What a Complete FTE Stack Looks Like on Azure

A Frontier Transformation Engineer building AI workloads on Azure needs both layers working together:

**HVE Core** governs how you build the application:
- GitHub Copilot CLI plugin: `copilot plugin marketplace add microsoft/hve-core`
- VS Code extension: search *HVE Core* in the Marketplace
- RPI workflow: Task Researcher → Task Planner → Task Implementor → Task Reviewer

**Azure IaC (AVM Bicep)** governs how you provision the platform:
- Azure Verified Modules as the module library: [aka.ms/avm](https://aka.ms/avm)
- Bicep as the orchestration language: declarative, reviewable, version-controlled
- GitHub Actions or Azure DevOps for pipeline-driven deployment
- `az deployment what-if` as the pre-apply review gate

Strip out the infrastructure layer and you have velocity without foundation: AI agents running on environments that drift, cannot be reproduced, and do not survive a disaster recovery exercise.

---

## The Insight I Took Away from the Workshop

HVE Core is a genuine contribution to how practitioners should work with AI coding assistants. The RPI methodology is the right answer to the "AI writes first and thinks never" problem.

But the Frontier Transformation Engineer profile, as Microsoft is defining it, spans the full delivery stack. For Azure practitioners, that stack goes below the application code. It includes the infrastructure your agents run on.

Azure IaC is not a separate discipline from HVE; it is the HVE principle applied one layer down. The same commitment to governed patterns, verified conventions, and traceable decisions that makes RPI work at the code layer is what makes AVM Bicep work at the infrastructure layer. Both are expressions of context engineering: designing what is known, governed, and repeatable at each layer of the stack.

If you are building toward the FTE profile and working on Azure, infrastructure as code is not optional. It is how you extend HVE velocity all the way to the bottom of the stack.

---

*The HVE Core repository is at [github.com/microsoft/hve-core](https://github.com/microsoft/hve-core). The VS Code extension installs in under 30 seconds. For Azure Verified Modules, start at [aka.ms/avm](https://aka.ms/avm).*
