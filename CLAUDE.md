# azureArchetype — Personal Site AI Assistant Instructions

> **About this file:** This file serves a dual purpose. It is the `CLAUDE.md`
> project instruction set loaded into Claude AI sessions for this repository,
> and it is also developer context for anyone reading the repo — documenting
> the site's conventions, structure, and working standards in one place.

You are an AI assistant working within the **azureArchetype** personal website
and blog project. This site is Matthew Collins's personal technical brand —
complementary to, but distinct from, frameType Solutions. The repository is
`azurearchetype/azurearchetype.github.io`, hosted on GitHub Pages, and synced
to this Claude project as the authoritative context source.

---

## Who You Are Working With

**Matthew Collins** — Azure infrastructure consultant, MCT, and founder of
frameType Solutions. This site is his personal technical voice.

| | |
|---|---|
| Brand handle | `azureArchetype` |
| Site | azurearchetype.github.io |
| GitHub | `azurearchetype` (personal) \| `frametypeSolutions` (company) |
| Specializations | Azure IaC (Bicep), Azure AI Foundry, agentic AI, AVD, networking |
| Certifications | AZ-700, AZ-140 \| Pursuing: AZ-104, AZ-305, GitHub Actions, Microsoft AI |
| Goals | Microsoft MVP nomination; personal brand as practitioner-voice in the Azure community |
| Brand tagline | *"Stay in the flow, avoid the noise."* |
| Sponsor | frameType Solutions \| frametypesolutions.com |

---

## Site Philosophy

This blog is Matthew's **practitioner's journal** — documenting real-world Azure
and AI experiences as they happen, with honesty about what works and what
doesn't. It is not polished marketing content; it is the voice of someone
actively building things and sharing what he learns.

The site exists at the intersection of three threads:

- **Azure cloud technologies** — IaC, networking, AVD, AI services, Marketplace
- **AI-driven development** — agentic AI, vibe-coding, GitHub Copilot, Claude
- **Personal experience** — the journey, the friction, the wins, the lessons

frameType Solutions is the **sponsoring entity** and the professional context
behind the content. The relationship is complementary: the blog is the
practitioner's voice; the company is the productized delivery. Content on this
site can reference and link to frameType Solutions work, but the site itself
is Matthew's personal platform, not a company marketing channel.

---

## Session Startup

Before starting any substantive work in a session:

1. Review the project knowledge base (synced repository files) to understand
   current site state — posts, layouts, config, open series
2. Note any posts with placeholder content (`<Enter amazing thoughts here>`)
   — these are open work items by definition
3. Confirm what work is in scope for the session before proceeding
4. **State your understanding before proceeding — do not assume**
5. Flag any gaps or ambiguities in the context

---

## Repository Structure

```
azurearchetype.github.io/
├── _posts/           # Blog posts (Markdown, dated filenames)
├── _layouts/         # Jekyll layout templates (default.html, home.html, post.html)
├── _includes/        # Reusable partials (left-sidebar.html, etc.)
├── _checklists/      # deploymentChecklist collection
├── assets/
│   └── img/          # Logos, badges, social icons
├── docs/             # Theme contribution docs (upstream Minimal theme — do not edit)
├── _config.yml       # Jekyll site configuration
└── index.md          # Home page content
```

**Platform:** Jekyll on GitHub Pages, Minimal theme (customized), dark theme,
three-column layout (left sidebar / center content / right sidebar).

---

## Naming Conventions

**camelCase is the organisational convention** — inherited from frameType
Solutions and applied consistently here:

- Post file names: `YYYY-MM-DD-descriptive-title-part-N.md`
  (dates use hyphens as required by Jekyll; descriptive segments use camelCase
  where multi-word, e.g. `deployNatGateway`)
- Series identifiers in frontmatter: camelCase (`deployNatGateway`,
  `IaC & App Dev`)
- Asset file names: camelCase
- Layout and include file names: camelCase

If a naming inconsistency is found, flag it rather than silently perpetuating it.

---

## Post Frontmatter Standard

Every post must include this frontmatter block:

```yaml
---
layout: post
title: "Series or Topic Name"
subtitle: "Specific Post Subtitle"
date: YYYY-MM-DD
author: Matthew
series: "seriesName"          # omit if standalone post
series_part: N                # omit if standalone post
series_title: "Full Series Title for Display"  # omit if standalone post
---
```

- `title` maps to the series or broad topic name
- `subtitle` carries the specific post identity
- Series posts require `series`, `series_part`, and `series_title`
- Standalone posts omit the series fields

Before creating any new post, verify this schema matches current `_config.yml`
collection definitions. If there is a discrepancy, flag it.

---

## Content Standards

### Voice and Tone
- First-person practitioner voice — Matthew is writing, not a company
- Conversational but technically precise — not dumbed down, not overly formal
- Honest about friction, failures, and unexpected outcomes — this is a journal,
  not a highlight reel
- The technical detail is the value — don't abstract away what actually happened
- Em dashes used sparingly — only where they genuinely improve clarity

### Structure
- Every post of more than three sections includes a **clickable Table of
  Contents** immediately after the frontmatter — always include it without
  being asked
- Use numbered heading hierarchy for posts intended for future RAG ingestion
- Introductory paragraph sets context — what is this about, why does it matter,
  what will the reader get from it
- Closing section: summary of key takeaways and what comes next (series) or
  invitation to engage (standalone)

### Technical Accuracy
- Azure CLI (`az`) is the preferred tooling reference — consistent with the
  frameType IaC standard and avoids Az PowerShell broker issues
- Bicep is the IaC authoring format; ARM JSON is a build artifact
- CAF and WAF alignment is a natural lens for architecture content — use it
  where relevant without being heavy-handed
- Code blocks use fenced Markdown with language identifiers

### Content Topics (primary)
- Azure IaC and Bicep — templates, patterns, Marketplace offer development
- Azure networking — NAT Gateways, Firewall, AVD networking
- Agentic AI and AI-driven development — GitHub Copilot, Claude, AI Foundry
- Azure Virtual Desktop — deployment, FSLogix, ARM patterns
- Personal journey content — MCT pursuit, MVP path, vibe-coding experiences

### Relationship to frameType Solutions Content
- Blog posts may reference and link to frameType Solutions work and offers
- Technical series on the blog (e.g. AVD observations) may draw from frameType
  Solutions engineering work — this is intentional and appropriate
- The sponsorship relationship (frameType logo + tagline in sidebar) is the
  explicit tie; content does not need to constantly call it out
- Do not reproduce internal frameType Solutions context, pricing, or build
  details on this public site

---

## Site Development Standards

### Jekyll and GitHub Pages
- All layout changes go in `_layouts/` — do not modify upstream Minimal theme
  files in `docs/`
- CSS changes go in `_layouts/` inline styles or a custom stylesheet — not
  in the upstream theme
- Test structural changes locally with `bundle exec jekyll serve` before
  committing if possible
- `_config.yml` is the source of truth for collections, navigation, and
  site-level settings

### GitHub Pages Workflow
- `main` branch is the production branch — GitHub Pages builds from it
- Post dating: Jekyll will not publish future-dated posts by default; use
  `future: true` in `_config.yml` during development if needed
- Images go in `assets/img/` — reference with `/assets/img/filename.ext`
  (absolute path, not relative) for reliable rendering across page depths

### Open Series Tracking
Posts with placeholder body content (`<Enter amazing thoughts here>`) are
open work items. Current open series:

| Series | Open Posts |
|---|---|
| deployNatGateway | Parts 3 and 4 |
| IaC & App Dev (GitHub Spark) | Part 1 |

Flag any additional placeholders discovered in the project knowledge base.

---

## Open Item Prefix

| Prefix | Scope |
|---|---|
| `SITE-0xx` | Site structure, layout, Jekyll config |
| `POST-0xx` | Blog post content, series, drafts |
| `ASSET-0xx` | Images, icons, static assets |

---

## Session Hygiene

- **Capture decisions** — if a choice is made about site structure, content
  direction, or conventions, note it explicitly
- **Capture reasoning** — the "why" behind decisions is as important as the
  "what"; future sessions (and future agents) depend on it
- **Name open items explicitly** — never leave gaps implicit
- At the close of a significant work session, summarize decisions made and
  open items remaining — this is the handoff for the next session

---

## Relationship to frameType Solutions Instructions

These instructions are the personal-site counterpart to the frameType Solutions
universal AI assistant instructions. The two sets are **complementary**:

| Dimension | frameType Solutions | azureArchetype (this site) |
|---|---|---|
| Voice | Company / product / offer | Personal practitioner |
| Audience | Partners, Microsoft, customers | Azure community, peers, learners |
| Content | Offer docs, IaC artifacts, RAG context | Blog posts, tutorials, personal musings |
| Repo pattern | Three-repo offer pattern | Single public GitHub Pages repo |
| Naming | camelCase org-wide | camelCase inherited |
| IaC standard | Bicep / az CLI | Same — consistent reference |
| Documentation standard | Canonical YAML frontmatter + ToC | Post frontmatter standard + ToC |

When working in this project, apply these personal-site instructions.
When a question of standards arises that isn't covered here (e.g. a specific
Bicep pattern, a naming edge case), the frameType Solutions conventions are
the parent reference — apply them unless they conflict with personal-site
context.
