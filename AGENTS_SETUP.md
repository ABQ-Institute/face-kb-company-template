# ABQ's FACE — Setup Instructions

> This file is temporary. Delete it after setup is complete.
> Built on [ABQ's FACE](https://abq.institute/face) by [ABQ Institute](https://abq.institute).

You are in **setup mode**. The `0-meta/kb-config.yaml` in this repo has `# TODO` markers — setup is not complete.

Your job: guide the AI Lead through the 4-question platform assessment, generate the completed `kb-config.yaml`, and open a PR.

**After setup is complete:**
Tell the AI Lead: "Setup done. You can now delete `AGENTS_SETUP.md` from the repo — it's no longer needed. Going forward, your agents will use `AGENTS_BOOT.md`."

---

# face-setup — First-Time KB Setup Guide

## What This Skill Does

Guides an AI Lead through the complete first-time setup of a knowledge base using the Framework for AI Context in Enterprise. Covers platform selection, KB creation, configuration, and first agent initialisation.

**This skill is used once per organisation (Phase 1 only).** After setup is complete, uninstall this skill — agents use `face-kb` for all ongoing operations.

---

## Two-Phase FACE Deployment Model

FACE deployment is split into two distinct phases:

### Phase 1 — Company Setup *(this skill)*

**Who:** AI Lead (one person per organisation)
**When:** Once, before any agents go live
**Tool:** `face-setup` skill

Steps:
1. Answer 4 platform questions (existing docs, technical capability, scale, notifications)
2. Agent recommends platform (Git or MCP) and generates `kb-config.yaml`
3. AI Lead creates KB (repo or space) with the generated scaffold
4. AI Lead reviews and approves the configuration
5. KB is live and configured

Output: a working KB with `0-meta/kb-config.yaml` in place.

**Uninstall `face-setup` when Phase 1 is complete.** It is not needed again unless the organisation migrates platforms.

### Phase 2 — Agent Deployment *(face-kb skill)*

**Who:** Each agent or user that needs KB access
**When:** After Phase 1 is complete
**Tool:** `face-kb` skill

Steps:
1. Install `face-kb` on the agent
2. Set `KB_LOCATION` in the agent's config (URL of the KB repo/space)
3. Agent boots → reads `kb-config.yaml` → loads `face-kb-core` → `face-kb-write` → platform skill → ready

Output: agent reads and writes KB with full source-of-truth protocol.

---

## When to Use This Skill

- Organisation is adopting FACE for the first time
- AI Lead needs to set up the KB and configure the first agent
- During or after an FACE workshop

---

## Three Setup Routes

Depending on when and how the AI Lead sets up, there are three routes:

### Route A — Guided Setup During Workshop

**When:** During an FACE training session with ABQ facilitators present.

**How it works:**
1. AI Lead interacts with the workshop agent in a dedicated channel
2. The agent runs the Platform Assessment (see below) interactively
3. The agent generates the KB scaffold and `kb-config.yaml`
4. AI Lead reviews and approves
5. The agent configures the first production agent on the spot
6. AI Lead leaves the workshop with a working KB + active agent

**Advantages:** Real-time guidance, immediate troubleshooting, fastest path to production.

### Route B — Self-Service Post-Workshop

**When:** AI Lead sets up after the workshop, independently.

**How it works:**
1. AI Lead opens any AI tool (Claude, ChatGPT, Copilot, internal agent — anything)
2. Pastes the FACE Setup Prompt (provided below) into the conversation
3. The AI walks them through the Platform Assessment
4. AI Lead creates the KB repository/space manually (following the generated instructions)
5. AI Lead copies the generated `kb-config.yaml` into `0-meta/`
6. AI Lead configures their first agent with `KB_LOCATION` + `face-kb` skill

**Advantages:** No dependency on ABQ. Works with any AI tool. Self-paced.

### Route C — Pre-Workshop Assessment

**When:** Before the workshop. ABQ gathers requirements in advance.

**How it works:**
1. ABQ sends a short questionnaire to the organisation (5 questions, see below)
2. ABQ processes responses and prepares the platform recommendation
3. During the workshop, setup is pre-determined — skips directly to KB creation
4. Workshop time is spent on KB population and agent training, not platform decisions

**Advantages:** Maximum workshop efficiency. Decisions already validated. No surprises.

---

## Platform Assessment

Four questions. Same logic regardless of route.

### Question 0: Organisation Type

**Ask this first, before any platform questions.**

> "What type of organisation are you setting up this KB for?"
>
> a) **Startup** — founding team, likely has advisors, headcount under ~50
> b) **Scaleup** — founders still active + growing management layer, 50–500 people
> c) **Enterprise** — established management hierarchy, founders may not be in day-to-day, 500+ people

**What this controls** — the `1-company/2-structure/` files scaffolded:

| org_type | Files created |
|----------|---------------|
| `startup` | `1-org-chart.md`, `3-founders.md`, `4-advisors.md` |
| `scaleup` | `1-org-chart.md`, `2-leadership.md`, `3-founders.md`, `4-advisors.md` |
| `enterprise` | `1-org-chart.md`, `2-leadership.md`, `5-key-roles.md` |

Set `org_type` in `kb-config.yaml` accordingly. Delete structure files that don't apply.

---

### Pre-check: Git Detection

**Before asking Question 1**, check the environment:

Is there a `.git` folder in the root of this repository? Is `kb-config.yaml` already set to `platform: git`?

- **Yes (Git repo detected):** Skip Question 1. Say: *"I can see this is a Git repository — platform is already set to `git`. Moving to Question 2."* Proceed directly to Question 2.
- **No:** Ask Question 1 as normal.

### Question 1: Existing Documentation System

*(Skip this if Git was detected in the pre-check above)*

> "Does your organisation already use a documentation/wiki platform?"
>
> a) **Confluence** → recommended: `mcp` with Confluence
> b) **Notion** → recommended: `mcp` with Notion
> c) **Slite** → recommended: `mcp` with Slite
> d) **SharePoint wiki / OneNote** → recommended: `mcp` with SharePoint
> e) **Other system with API** → recommended: `mcp` (if MCP server available for that platform)
> f) **Nothing structured / scattered docs** → go to Question 2
> g) **Already using Git** → platform = `git` confirmed, go to Question 2

If `mcp` is recommended, also ask:
> "Is your installation Cloud-hosted or on-premise / Server / Data Center?"
>
> - **Cloud** → use official MCP server (Atlassian, Slite — remote, OAuth, vendor-hosted)
> - **On-premise / Server / DC** → use community MCP server (self-hosted, PAT auth)

### Question 2: Technical Capability

> "Does your organisation have technical staff comfortable with Git?"
>
> a) **Yes, we use Git daily** → recommended: `git` (GitHub/GitLab + GitBook)
> b) **Some people do** → recommended: `git` with GitBook as the human-readable layer
> c) **No** → recommended: `git` with GitBook — agent handles all Git operations, humans interact only through GitBook UI or chat

### Question 3: Scale

> "How many people will contribute to or consume the KB?"
>
> a) **< 20** → single repository/space is sufficient
> b) **20–100** → consider department-level separation early
> c) **100+** → plan for Context Broker from the start (see `3-framework/4-context-broker.md`)

### Question 4: Notifications

> "Where should KB update notifications go? Provide the channel name or ID."
>
> Examples: `#kb-updates`, `#general`, a Slack channel ID
>
> This is where the agent announces new PRs/drafts, approved changes, and flags. Always ask — do not assume a default.

### Decision Output

```
Platform:           git | mcp
MCP provider:       confluence | slite | notion | sharepoint (if mcp)
MCP server:         official | community (if mcp)
Write strategy:     branch+PR (if git) | draft | staging | comment (if mcp)
Reason:             [one sentence]
Recommended setup:  [specific tool names]
Scale:              single repo/space | multi-repo | broker planned
Notifications:      [channel] via [slack | email | teams]
```

---

## KB Scaffold

After platform decision, create the KB with this structure:

### For Git-based KB

Create a new repository with:

```
0-meta/
  kb-config.yaml          ← generate from Git template (see below)
  README.md               ← "This folder contains KB metadata and configuration."
1-company/
  README.md               ← "Company-wide context. Accessible to all."
2-departments/
  README.md               ← "Department-specific context."
3-products/
  README.md               ← "Product documentation and context."
4-projects/
  README.md               ← "Project-specific working context."
5-agents/
  README.md               ← "Agent configurations and skill assignments."
SUMMARY.md                ← GitBook navigation (if using GitBook)
README.md                 ← Repository landing page
```

Enable branch protection on `main`:
- Require pull request before merging
- Require at least 1 approval
- No direct pushes

**Reference script available:** After setup, agents can use `face-kb-git/scripts/kb_write.py` to create branches and PRs programmatically. See `face-kb-git` for details.

### For MCP-based KB

Create a space/workspace in the platform (Confluence, Slite, Notion) with:

```
0-meta (section/parent page)
  └── kb-config              ← page containing the YAML config below
1-company (section/parent page)
  └── README                 ← describes the section, lists children
2-departments (section/parent page)
  └── README
3-products (section/parent page)
  └── README
4-projects (section/parent page)
  └── README
5-agents (section/parent page)
  └── README
```

Each parent page acts as its own README — describes what the section contains and lists child pages.

**Write strategy selection:**
- If the platform supports drafts (Confluence, Slite) → use `draft` strategy (default)
- If drafts are not available or team prefers isolation → use `staging` with a separate staging space
- If the team wants minimal agent permissions → use `comment` (agent only adds comments, never creates pages)

---

## kb-config.yaml Templates

Two templates are available depending on the platform. Use the appropriate one.

### Git Template

Reference: `face-kb-git/templates/kb-config-git.yaml`

```yaml
kb:
  platform: git
  main_ref: main
  language: en
  repository: https://github.com/your-org/your-knowledge-base

owners:
  company: "CTO / COO"
  departments:
    engineering: "Engineering Lead"
  products:
    main-product: "Product Manager"
  projects:
    current-project: "Project Lead"

notifications:
  channel: "#kb-updates"
  method: slack

skills:
  version: "1.0"
```

### MCP Template

Reference: `face-kb-mcp/templates/kb-config-mcp.yaml`

```yaml
kb:
  platform: mcp
  main_ref: published
  language: en

mcp:
  provider: confluence          # confluence | slite | notion
  server: official              # official | community
  space: "TEAM-KB"             # Confluence space key / Slite channel name
  write_strategy: draft         # draft | staging | comment
  # staging_space: "TEAM-STAGING"  # uncomment if write_strategy = staging

owners:
  company: "CTO / COO"
  departments:
    engineering: "Engineering Lead"
  products:
    main-product: "Product Manager"
  projects:
    current-project: "Project Lead"

notifications:
  channel: "#kb-updates"
  method: slack

skills:
  version: "1.0"
```

---

## First Agent Configuration

After KB and config are in place:

### Step 1: Choose the Agent Platform

The first agent can be any AI tool that supports custom instructions:
- Claude (via Projects, system prompt, or AGENTS.md)
- ChatGPT (via Custom Instructions or GPTs)
- Internal agent (via system prompt or config file)
- OpenClaw agent (via AGENTS.md + skills)

### Step 2: Configure the Agent

Add to the agent's configuration:

```
KB_LOCATION: https://github.com/your-org/knowledge-base
```

And assign the `face-kb` skill. How this is done depends on the platform:
- **AGENTS.md:** Add skill reference and KB_LOCATION
- **System prompt:** Include the face-kb SKILL.md content + KB_LOCATION
- **Skill config:** Upload face-kb skill files

### Step 3: Verify

Ask the agent:
> "Where is the knowledge base and what platform is it on?"

Expected response:
> "The KB is at [URL], running on [git/mcp]. I've loaded face-kb-core, face-kb-write, and face-kb-[git/mcp]. Ready to work."

### Step 4: Populate

The agent is now ready to help populate the KB. Start with:
1. Company layer — organisation context, mission, structure
2. Whatever layer is most urgent for the team

For Git: the agent creates branches and PRs — AI Lead reviews and merges.
For MCP: the agent creates drafts — AI Lead reviews and publishes.

---

## Pre-Workshop Questionnaire (Route C)

Send this to the organisation before the workshop:

```
FACE Pre-Workshop Assessment
────────────────────────────

1. What documentation/wiki system do you currently use?
   (Confluence / Notion / Slite / SharePoint / Other: ___ / None)

2. If you use Confluence or similar: is it Cloud-hosted or on-premise?
   (Cloud / On-premise / Not applicable)

3. Do your teams use Git for anything?
   (Yes, daily / Some teams / No)

4. How many people will contribute to or read from the KB?
   (< 20 / 20-100 / 100+)

5. What communication platform do your teams use?
   (Slack / Microsoft Teams / Other: ___)
```

---

## FACE Setup Prompt (Route B)

AI Lead can paste this into any AI tool for self-guided setup:

```
I'm setting up a knowledge base using the Enterprise AI Context 
Framework (FACE). I need you to help me:

1. Choose the right platform (Git or MCP-based) by asking me 
   about my current documentation setup, whether it's Cloud or 
   on-premise, my team's technical capability, and team size.

2. Generate the folder structure for my KB.

3. Generate a kb-config.yaml file with my specific configuration,
   including the mcp: section if I'm using Confluence/Slite/Notion.

4. Tell me how to configure my first AI agent to use this KB.

Rules:
- The KB is the single source of truth. Everything else is WIP.
- All changes go through an approval process (PR for Git, 
  draft/staging for MCP platforms).
- The agent never publishes or merges without human approval.
- Use official MCP servers when available (Atlassian, Slite).

Start by asking me about my current documentation setup.
```

---

## Phase 1 Complete — Handoff Checklist

When setup is done, present this checklist to the AI Lead before closing:

```
✅ Phase 1 — KB Setup Complete

KB location:       [URL or space name]
Platform:          [git | mcp]
MCP provider:      [confluence | slite | notion | n/a]
MCP server:        [official | community | n/a]
Write strategy:    [branch+PR | draft | staging | comment]
Main ref:          [main | published]
kb-config.yaml:    ✅ in place at 0-meta/
Access control:    ✅ [branch protection enabled | space permissions set]

─────────────────────────────────────────
Next: Phase 2 — Deploy agents

For each agent that needs KB access:

1. Install the `face-kb` skill
2. Add to agent config:
      KB_LOCATION: [KB_URL]
3. Verify with: "Where is the KB and what platform is it on?"

Skills loaded automatically by face-kb:
  - face-kb-core  (source of truth + structural rules)
  - face-kb-write (content routing + extraction)
  - face-kb-git   (if platform=git)
  - face-kb-mcp   (if platform=mcp)

─────────────────────────────────────────
Remove `face-setup` from this agent — it is no longer needed.
If you ever migrate platforms, re-run face-setup at that time.
```

---

*Owner: Luna @ ABQ | Last updated: 16/04/2026*
