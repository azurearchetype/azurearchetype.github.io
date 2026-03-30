---
layout: post
title: "Finding Your Flow State"
subtitle: "in the Age of Vibe Coding and Agentic AI"
date: 2026-03-29
author: Matthew

series:
series_part:
series_title:

tags:
  - azure
  - avd
  - iac

excerpt: "What happens to flow state when the AI isn't just co-authoring the code, but running the whole deployment pipeline while you architect the next one?"
---

<svg width="100%" viewBox="0 0 680 200" xmlns="http://www.w3.org/2000/svg">
  <defs><clipPath id="card"><rect width="680" height="200" rx="12"/></clipPath></defs>
  <rect width="680" height="200" fill="#0A1F35" rx="12"/>
  <g clip-path="url(#card)">
    <g font-family="monospace" font-size="9" fill="#5BB8F5" opacity="0.25" text-anchor="end">
      <text x="648" y="18">resource "azurerm_key_vault" "main" {</text>
      <text x="648" y="30">  name = var.key_vault_name</text>
      <text x="648" y="42">  soft_delete_retention_days = 90</text>
      <text x="648" y="54">  purge_protection_enabled   = true</text>
      <text x="648" y="66">}</text>
      <text x="648" y="78">module "hub_network" {</text>
      <text x="648" y="90">  source = "./modules/networking/hub"</text>
      <text x="648" y="102">  address_space = var.hub_address_space</text>
      <text x="648" y="114">}</text>
    </g>
    <path d="M -10 132 C 40 112, 80 152, 130 135 C 180 118, 220 148, 270 128 C 320 108, 360 142, 410 124 C 460 106, 500 138, 550 120 C 600 104, 640 132, 690 118 L 690 200 L -10 200 Z" fill="#0D2D4F" opacity="0.95"/>
    <path d="M -10 148 C 50 128, 100 165, 160 148 C 220 131, 260 160, 320 144 C 380 128, 420 158, 480 142 C 540 126, 580 154, 640 140 C 660 134, 678 143, 690 140 L 690 200 L -10 200 Z" fill="#0F3660" opacity="0.75"/>
    <path d="M -10 163 C 60 150, 120 172, 185 160 C 250 148, 290 168, 360 158 C 430 148, 470 166, 540 156 C 600 147, 645 162, 690 155 L 690 200 L -10 200 Z" fill="#163D6A" opacity="0.55"/>
    <path d="M -10 176 C 80 168, 140 182, 210 174 C 280 166, 330 178, 410 172 C 490 166, 540 178, 620 172 L 690 170 L 690 200 L -10 200 Z" fill="#1A4575" opacity="0.35"/>
    <line x1="40" y1="155" x2="120" y2="155" stroke="#1A4A7A" stroke-width="0.5"/>
    <line x1="40" y1="125" x2="120" y2="125" stroke="#1A4A7A" stroke-width="0.5"/>
    <line x1="40" y1="95" x2="120" y2="95" stroke="#1A4A7A" stroke-width="0.5"/>
    <line x1="40" y1="65" x2="120" y2="65" stroke="#1A4A7A" stroke-width="0.5"/>
    <line x1="40" y1="38" x2="120" y2="38" stroke="#1A4A7A" stroke-width="0.5"/>
    <rect x="52" y="143" width="18" height="12" fill="#1F5C99" rx="2"/>
    <rect x="75" y="131" width="18" height="24" fill="#1F5C99" rx="2"/>
    <rect x="98" y="113" width="18" height="42" fill="#2A7BC8" rx="2"/>
    <path d="M 155 150 C 200 110, 240 130, 270 100 C 310 62, 360 78, 400 52 C 440 28, 480 42, 530 38 C 565 35, 595 38, 625 34" fill="none" stroke="#3A9EE8" stroke-width="2" stroke-linecap="round"/>
    <path d="M 155 150 C 200 110, 240 130, 270 100 C 310 62, 360 78, 400 52 C 440 28, 480 42, 530 38 C 565 35, 595 38, 625 34 L 625 200 L 155 200 Z" fill="#1A4A7A" opacity="0.08"/>
    <circle cx="270" cy="100" r="3.5" fill="#3A9EE8" opacity="0.8"/>
    <circle cx="400" cy="52" r="3.5" fill="#3A9EE8" opacity="0.8"/>
    <circle cx="530" cy="38" r="5" fill="#5BB8F5"/>
    <line x1="530" y1="38" x2="530" y2="148" stroke="#2A7BC8" stroke-width="0.5" stroke-dasharray="3 3" opacity="0.6"/>
    <rect x="490" y="130" width="80" height="22" fill="#091827" rx="4" opacity="0.85"/>
    <text x="530" y="145" text-anchor="middle" font-family="monospace" font-size="11" fill="#5BB8F5">flow state</text>
    <text x="145" y="30" font-family="monospace" font-size="10" fill="#3A9EE8" opacity="0.6">// agentic pipeline active</text>
    <rect x="143" y="33" width="190" height="0.5" fill="#1F5C99" opacity="0.4"/>
    <rect x="588" y="46" width="62" height="14" fill="#091827" rx="3" stroke="#1A4A7A" stroke-width="0.5" opacity="0.9"/>
    <rect x="590" y="48" width="34" height="10" fill="#2A7BC8" rx="2"/>
    <text x="648" y="58" text-anchor="end" font-family="monospace" font-size="9" fill="#5BB8F5">74%</text>
    <rect x="588" y="64" width="62" height="14" fill="#091827" rx="3" stroke="#1A4A7A" stroke-width="0.5" opacity="0.9"/>
    <rect x="590" y="66" width="54" height="10" fill="#1F5C99" rx="2"/>
    <text x="648" y="76" text-anchor="end" font-family="monospace" font-size="9" fill="#5BB8F5">92%</text>
    <circle cx="155" cy="150" r="3" fill="#1F5C99" opacity="0.7"/>
    <circle cx="625" cy="34" r="3" fill="#3A9EE8" opacity="0.7"/>
    <text x="40" y="192" font-family="monospace" font-size="10" fill="#5BB8F5" opacity="0.5">boredom</text>
    <text x="340" y="192" text-anchor="middle" font-family="monospace" font-size="10" fill="#5BB8F5" opacity="0.8">flow</text>
    <text x="640" y="192" text-anchor="end" font-family="monospace" font-size="10" fill="#5BB8F5" opacity="0.5">anxiety</text>
  </g>
</svg>


*Infrastructure as Code · Vibe Coding · Agentic AI · Developer Experience*

**Matthew Collins** · MCT · Azure Infrastructure · IaC · AI Foundry · 7 min read

---

> People who know me have heard me say it: **"Stay in the flow, avoid the noise."** In today's Frontier construct, where delivering Infrastructure as Code means working alongside systems that plan, execute, and adapt on your behalf, that principle is more true, and more actionable, than ever.

---

There's a particular kind of afternoon where everything clicks. Your templates are composing themselves almost before you think about them. The parameter files are structured, the CAF-aligned naming convention finally makes sense, and there's not a squiggle in sight from your VS Code session. You're not writing infrastructure. You're conducting it.

Psychologist Mihaly Csikszentmihalyi called this flow: "a state of deep focus where challenge and skill align so perfectly that time collapses and the work feels effortless." Athletes call it being "in the zone." Developers have been quietly chasing it since the first time a semicolon made everything compile.

The interesting question right now, in 2026, is: *what happens to flow when the AI isn't just co-authoring the code, but running the whole deployment pipeline while you architect the next one?*

> *"Flow doesn't mean easy. It means the friction is in the right place — between your intent and the outcome, not between you and the tooling."*

---

## First, a quick primer on the psychology

Csikszentmihalyi's research identified eight characteristics of flow: clear goals, immediate feedback, a balance of challenge and skill, a sense of personal control, loss of self-consciousness, altered time perception, intrinsic reward, and effortless concentration. The model is often illustrated as a channel. Too easy and you're bored; too hard and you're anxious; just right and you're in flow.

| Boredom Zone | Flow Zone | Anxiety Zone |
|---|---|---|
| Writing resource blocks no one should be writing by hand when an agent can do it in moments | Designing a modular IaC architecture with AI as your coding partner | Debugging a deployment that hallucinated and used old resource IDs, or dragged you down the path of an outdated technology |

The classic IaC process — using a transpilative (it's a word, look it up) language like Bicep to compile into ARM templates, managing idempotency, untangling dependency graphs — has always lived somewhere between flow and frustration. The syntax is unforgiving and the feedback loop (deploy, fail, read the error, Google the error, repeat) is long. Flow was possible, but it required serious experience to get there.

---

## Enter vibe coding — and the flow question it raises

"Vibe coding" is the loosely-defined practice of collaborating with an AI chatbot or agent on code in a more fluid, iterative, conversational way. Less specification, more direction. You describe intent; the model fills structure. The term captures something real: a shift in how developers, and especially architects, are working with AI today.

For IaC, this is genuinely transformative. A well-prompted AI interaction can scaffold a template module with proper parameter assignment, output definitions, and resource decorators in seconds. That frees you to operate at the architectural level, thinking about blast radius, reusability, compliance, and best practices, rather than fighting curly braces.

But here's the nuance that gets lost in the hype: vibe coding doesn't automatically create flow. It changes *where* the flow conditions need to come from.

Vibe coding can tick most of the flow boxes fast. Clear goal? Easy. Immediate feedback? Faster than ever. But challenge level and sense of control? Those depend entirely on whether you're actually reviewing the output — or just merging it blindly and hoping for the best.

---

## The trap: passive acceptance versus active design

Flow requires agency. It's not passive absorption; it's active navigation. The risk with vibe coding, especially for infrastructure work, is that the AI can produce code that looks right while quietly misunderstanding your environment. A Key Vault deployed with soft-delete enabled sounds harmless until every subsequent test deployment fails with a VaultAlreadyExists conflict the AI didn't anticipate and can't easily unwind. A policy assignment that silently ignores exemption scope can lead to a very bad Friday afternoon.

Experienced IaC architects who find flow with AI tools tend to share a common habit: they stay in the driver's seat conceptually. They use AI chat and agents to handle the syntax and dependencies of infrastructure while they focus on the design. The model writes the resource block; the architect decides the blast radius. That division of labor, when it's working well, is genuinely flow-inducing because both sides are doing what they're best at.

---

## Now add agentic AI — and watch the flow channel expand

Vibe coding is a conversation. Agentic AI is a delegation. That distinction matters enormously for flow.

An agentic AI system doesn't just respond to prompts. It pursues goals across multiple steps, using tools, making decisions based on retrieval-augmented generation (RAG), and handing results back to you at the level of abstraction you actually care about. In IaC terms: you describe the target state, the agent runs the what-if analysis, identifies drift, generates the corrective templates, validates against your policy definitions, and surfaces a summary for your review. You never touched the code. You made the architectural call.

*That's not laziness. That's flow at a higher state.*

> *"With vibe coding, the AI clears the syntax so you can think about the design. With agentic AI, it clears the pipeline so you can keep your eye on the outcome."*

Think about what Csikszentmihalyi's conditions look like when an agent is doing the heavy lifting:

**01 — Clear goals**
You define intent at the system level: "Validate this module set against our approved Bicep definitions across all three regions, identify any drift or Zero Trust gaps, and produce a remediation plan I can review before anything gets touched." The agent handles decomposition.

**02 — Immediate feedback — supercharged.** Custom agents running in Microsoft Foundry, integrated with your existing GitHub workflows, can run validation, surface drift, and return structured results in the same loop. The feedback is faster and richer than any manual cycle.

**03 — Challenge at the right level**
When the agent handles the template mechanics, the work that's left is the work that actually matters: design decisions, security posture, cost trade-offs, and the governance questions no prompt can answer for you. That's where experienced architects thrive.

**04 — Sense of control — maintained by design**
Well-designed agentic workflows surface decisions, not just outputs. The architect approves, redirects, and sets constraints. Autonomy without oversight is a liability; autonomy with checkpoints is flow.

**05 — Intrinsic reward**
When AI handles the grunt work, the time you spend is on the problems only you can solve. That's a fundamentally more satisfying workday and a more defensible career position.

---

## The conditions that make agentic flow work

Agentic AI doesn't hand you flow automatically any more than a CI/CD pipeline guarantees a clean deployment. The conditions still have to be right. A few things that consistently make the difference:

**Design for checkpoints, not just completion.** The best agentic IaC workflows treat the architect as the approver of consequential decisions, not a bystander to automation. "Here's what I'm about to deploy, confirm?" keeps you in control and in the loop. It also keeps you sharp.

**Give the agent a well-defined scope.** Agentic systems work best when the goal is specific and the constraints are clear. "Modernize our IaC" is an invitation for chaos. "Audit these ten modules against our CAF baseline and flag anything that needs remediation" is a job description. Precision in your intent is what lets the agent move confidently.

**Stay literate in what the agent is doing.** This is the non-negotiable. Flow requires competence, and in an agentic context, competence means understanding the decisions the agent is making on your behalf.

> *"Architects who lose the thread of what their agentic pipeline is doing aren't in flow; they're on autopilot. Those are very different states."*

**Use the reclaimed time intentionally.** When an agent handles a deployment workflow that used to take you half a day, the flow opportunity is in what you do with the other four hours. Design the next architecture. Refine your module library. Even ideate on the next big thing. The agent multiplies your output, but only if you're directing that multiplication somewhere meaningful.

---

## The bigger picture

Csikszentmihalyi's insight was that flow isn't about the tools. It's about the relationship between challenge and capability. Better tools don't guarantee flow; they change the shape of the challenge.

The progression from traditional IaC deployment to vibe coding to agentic AI isn't a story about automation replacing architects. It's a story about the challenge level rising to meet growing capability. Each layer clears a different kind of friction: syntax, then pipeline, then repetition, freeing you to work at the level where the real decisions live.

For cloud architects, that's a genuinely exciting place to be. The template structure is mostly solved. The pipeline is increasingly manageable. What's left is the architecture, the governance, the system design — the work that actually matters and where flow is absolutely available.

*Stay in the flow. Avoid the noise. The tools are finally good enough to let you find it.*

---

#InfrastructureAsCode #Bicep #AgenticAI #MicrosoftFoundry #VibeCoding #AzureDevOps #FlowState #CloudArchitecture #AIFoundry
