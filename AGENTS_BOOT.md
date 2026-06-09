# ABQ's FACE — Agent Instructions

> Built on [ABQ's FACE](https://abq.institute/face) — the Framework for AI Context in Enterprise by [ABQ Institute](https://abq.institute).

You are operating inside an FACE-structured knowledge base. Apply the skills below for all read and write operations.

**Important:** Check `0-meta/kb-config.yaml`. If it still has `# TODO` markers, stop and read `AGENTS_SETUP.md` instead — setup is not complete.

---

## Skills Bundle Version

**Embedded version:** `2026-05-26`
**Canonical source:** [`abq-knowledge-base/3-products/face/7-skills/SKILLS_VERSION`](https://github.com/ABQ-Institute/abq-knowledge-base/blob/main/3-products/face/7-skills/SKILLS_VERSION)
**Marker file (machine-readable):** [`0-meta/.face-skills-version`](0-meta/.face-skills-version)

The four skills embedded below (`face-kb`, `face-kb-core`, `face-kb-write`, `face-kb-git`) are a snapshot of the canonical FACE skills as of the embedded version above. Skills evolve when an ADR shifts the architecture — for example, this snapshot reflects ADR-KBM-007's hybrid access model (FACE owns intent, GitHub owns enforcement, CODEOWNERS is projector-derived).

### Detecting drift

The KB Manager surfaces skill-bundle drift automatically as a non-dismissable banner with a one-click "refresh skills" PR (ADR-KBM-007 §6 drift-banner pattern).

If you're not using the KB Manager, check manually:

```bash
LOCAL=$(grep -v '^#' 0-meta/.face-skills-version | grep -v '^$' | tail -1)
CANONICAL=$(curl -s https://raw.githubusercontent.com/ABQ-Institute/abq-knowledge-base/main/3-products/face/7-skills/SKILLS_VERSION \
  | grep -v '^#' | grep -v '^$' | tail -1)
echo "Local:     $LOCAL"
echo "Canonical: $CANONICAL"
```

If they differ, the embedded skills below are outdated. Refresh by replacing each skill section with the matching `SKILL.md` from the canonical repo at
[`abq-knowledge-base/3-products/face/7-skills/`](https://github.com/ABQ-Institute/abq-knowledge-base/tree/main/3-products/face/7-skills),
then bump `0-meta/.face-skills-version` to match. Commit + PR; CODEOWNERS routes the review to the KB admins.

---

## Data Status Convention

Every content file in this KB has a `data_status` field in its YAML front matter. It declares how reliable the content is.

| Value | Meaning | Agent behaviour |
|-------|---------|----------------|
| `placeholder` | Structure only — no real content yet | Do not cite. Prompt the human to fill it in. |
| `plausible` | Content exists but has not been confirmed by the team | Cite only with explicit caveat: _"This content is unverified — please confirm with the document owner."_ |
| `verified` | Confirmed accurate by the document owner | Cite freely. |

**Rules:**
- When creating or updating a file, always declare `data_status` in the front matter.
- Never present `plausible` content as authoritative.
- When a human confirms content is accurate, update `data_status` from `plausible` to `verified` via the normal PR process.
- If `data_status` is missing from a file, treat it as `plausible` until confirmed.

---

# face-kb — Knowledge Base Agent Skill

## What This Skill Does

Single entry point for any AI agent that needs to work with an organisation's knowledge base. Loads the universal rules and the correct platform implementation automatically based on the organisation's configuration.

**This is the only KB skill you need to assign to an agent.** It handles the rest.

## When to Use It

Assign this skill to any agent that:
- Answers questions using KB content
- Writes or proposes changes to KB content
- Helps users find documents or policies
- Onboards new agents into the KB

## Boot Sequence

When an agent starts with this skill:

```
1. Read KB_LOCATION from agent config
         │
2. Fetch {KB_LOCATION}/0-meta/kb-config.yaml
         │
    ┌────┴──── Config found? ────┐
    │ YES                        │ NO
    │                            │
3a. Parse config              3b. Tell the human:
    │                             "KB config not found.
4a. Load face-kb-core             Use face-setup to
    (always)                      initialise."
    │                             STOP.
5a. Load face-kb-write
    (always)
    │
6a. Infer platform from KB_LOCATION
    │   (github.com / gitlab.com / bitbucket / azure DevOps URL → git;
    │    MCP server URL → mcp). Legacy configs may carry kb.platform;
    │    readers tolerate it but the shape above is authoritative.
    │
    ├── git → Load face-kb-git
    │
    └── mcp → Load face-kb-mcp
         │
7. Cache admins + owners + notifications
         │
    READY
```

## Agent Configuration

The agent needs exactly **one** setting:

```
KB_LOCATION: https://github.com/Acme-Corp/knowledge-base
```

Where to put it depends on the agent platform:
- **AGENTS.md / system prompt** — most AI tools
- **Environment variable** — programmatic agents
- **Skill config** — platforms with skill configuration

Everything else comes from `kb-config.yaml` inside the KB itself.

## What Gets Loaded

| Component | Source | Purpose |
|-----------|--------|---------|
| `face-kb-core` | Always loaded | Source of truth rules, KB structure, read/write protocol |
| `face-kb-write` | Always loaded | Content routing + extraction for incoming documents |
| `face-kb-git` | If `KB_LOCATION` is a git URL | Git-specific: branch, PR, SUMMARY.md, GitBook |
| `face-kb-mcp` | If `KB_LOCATION` is an MCP server | MCP server: Confluence, Notion, SharePoint, etc. |
| Admins + owners | From `kb-config.yaml` | KB-wide admins (`admins:`) + per-layer owners (`owners.<layer>`); see ADR-KBM-007 |
| Notification config | From `kb-config.yaml` | Where to announce PRs/changes |

## If kb-config.yaml Is Missing

The agent cannot function without it. Response:

> "This knowledge base doesn't have a configuration file yet. To set it up, the AI Lead should follow the FACE Setup Guide (`face-setup` skill). This takes about 15 minutes and only needs to be done once."

Provide the `face-setup` skill reference and stop. Do not guess configuration values.

## If KB_LOCATION Is Missing

The agent cannot function without it. Response:

> "I don't know where the knowledge base is. Please add `KB_LOCATION` to my configuration with the URL of your knowledge base."

## Companion Skills

| Skill | Role |
|-------|------|
| `face-kb-core` | Source of truth + structural rules (loaded automatically) |
| `face-kb-write` | Content routing + extraction (loaded automatically) |
| `face-kb-git` | Git implementation (loaded automatically if platform=git) |
| `face-kb-mcp` | MCP implementation (loaded automatically if platform=mcp) |
| `face-setup` | First-time KB setup (used once, by AI Lead) |

---

# KB Core — Source of Truth & Structural Rules

## The Prime Directive

**The KB is the only source of truth.**

If it is not in the KB, it does not officially exist. Slack messages, Word docs, emails, OneDrive folders, Notion pages, local files, verbal agreements — none of these are truth. They are working surfaces. Useful for getting work done, worthless as authoritative references.

An agent that cites a Slack message as truth is wrong. An agent that cites a Google Doc as truth is wrong. There is exactly one place where truth lives: the KB, on the main/published branch.

This is not a preference. It is a hard rule.

---

## What This Skill Does

Defines the universal rules every AI agent must follow when working with a knowledge base built on the Framework for AI Context in Enterprise — regardless of platform (Git, Confluence, Notion, SharePoint, or any other system).

This skill covers two domains:
- **Source of truth** — what counts as authoritative and how agents handle reads, writes, and conflicts
- **KB structure** — how content is organised (layers, folders, naming, numbering)

Platform-specific implementation belongs in companion skills (`face-kb-git`, `face-kb-mcp`). This skill contains only the rules that never change regardless of platform.

---

## Part A — Source of Truth

### A1. Two States. No Exceptions.

Every piece of information in an organisation exists in exactly one of two states:

| State | Where | What it means |
|-------|-------|---------------|
| **Truth** | KB — main/published branch | Approved, reviewed, authoritative. Cite freely. |
| **Work in Progress (WIP)** | Everywhere else | Draft, unreviewed, secondary. Always label as such. |

There is no middle ground. "Mostly done", "basically approved", "we all agreed on Slack" — still WIP. Until it is in the KB, it is not truth.

### A2. KB-First Query Rule

When anyone asks for information, a document, a policy, or any content:

1. **Search the KB first. Always.**
2. **Found in KB** → respond with KB content + reference. Mark as `[KB ✓]`.
3. **Not in KB, found elsewhere** → respond with the content but state explicitly:
   > "This is not in the KB — it exists in [source] as a working draft. It is not authoritative. Want me to open a proposal to bring it into the KB?"
4. **Not found anywhere** → say so directly. Offer to draft it.

**Never present WIP as authoritative.** Even if the WIP version is more recent than the KB version. Even if everyone in the room agrees it's correct. If it's not in the KB, it's not truth.

### A3. Conflict Resolution

When KB content and WIP content conflict, there is no conflict to resolve — **KB wins, always.**

Flag the discrepancy:
> "The KB says X. [Source] says Y. KB is authoritative. If Y is correct, we need to update the KB via the proper approval process."

Never silently use the WIP version. Never assume "the latest file is probably more accurate." The KB version is more accurate by definition — it went through review. If it's wrong, fix it through the proper process.

### A4. Citation Format

Every response that references KB content must include:

- **Source:** KB path or WIP location
- **Status:** `[KB ✓]` or `[WIP — not in KB]`
- **Last updated:** if available

Example:
> `1-company/policies/remote-work.md` [KB ✓] — last updated 2026-04-10

### A5. "Where Is the Latest Version?" Protocol

> • **KB version:** [title] ([link]) — last updated [date]. **This is the truth.**
> • **Most recent working version:** [title] ([link]) — last updated [date]. **This is WIP, not authoritative.**

If no KB version exists: "This is not in the KB yet — working version only. It should not be cited as official."

If they differ significantly: flag it, offer to open a proposal to sync.

---

## Part B — KB Structure

### B1. Layer Architecture

Every KB built on FACE uses the same four-layer structure. This is the standard. Do not invent new layers.

```
0-meta/          ← KB configuration (kb-config.yaml, owners, setup)
1-company/       ← Organisation-wide: mission, values, policies, strategy
2-departments/   ← Per department: processes, team context, responsibilities
3-products/      ← Per product: specs, architecture, decisions, roadmap
4-projects/      ← Per project: scope, timelines, deliverables, retrospectives
```

Optional layers (add only if needed):
```
5-agents/        ← Agent configurations, prompts, skill assignments
6-archive/       ← Deprecated content (never delete — archive)
```

**Rule:** If content doesn't fit a layer, ask — don't create a new top-level folder. The answer is almost always that the content fits somewhere existing, or belongs in a subfolder.

### B2. Folder and File Naming

- **Lowercase, hyphens only:** `remote-work-policy.md` ✅ `Remote Work Policy.md` ❌
- **Descriptive:** `context-broker.md` ✅ `notes.md` ❌
- **No version numbers in filenames** — use the KB platform's history/audit trail
- **Prefix with number** to control reading order: `3-context-broker.md` ✅

### B3. Numbering Convention

All folders and files use numeric prefixes to enforce reading order:

```
2-departments/
  1-engineering/
    1-overview.md
    2-processes.md
    3-team-context.md
  2-marketing/
    1-overview.md
    2-campaigns.md
```

Numbers reflect **logical reading order**, not creation order. Renumber when the logical order changes.

### B4. Every Folder Must Have a README.md

No exceptions. README.md is the landing page for the folder — it tells anyone (human or agent) what's inside and where to start.

Minimum README.md contents:
- One-sentence description of what the folder contains
- Numbered list of files/subfolders with one-line descriptions
- "Start here" pointer if the reading order matters

### B5. Page Title Convention

Every page's H1 title includes its `section.page` number:

```markdown
# 2.1. Engineering Overview
# 3.4. Context Broker Architecture
```

This makes the reading order explicit when viewing a single page outside its folder context.

### B6. Document Decisions as ADRs

When a significant decision is made — one that someone will ask "why did we do it this way?" in 3 months — document it as an Architecture Decision Record.

**What qualifies as an ADR:**
- A choice between alternatives with trade-offs (e.g. "PostgreSQL over MongoDB because...")
- A structural change to the KB itself (e.g. "merged two layers into one because...")
- A process change that affects how people work (e.g. "all PRs need 2 reviewers because...")
- A framework or architecture decision (e.g. "agents never merge — humans always approve")

**What does NOT need an ADR:**
- Routine content additions ("added Q1 metrics page")
- Minor corrections ("fixed a broken link")
- Anything where there was no real choice to make

**Where ADRs live:**

| Decision scope | Location |
|---------------|----------|
| KB-wide / FACE framework | `0-meta/decisions/` |
| Company-level | `1-company/decisions/` |
| Department-specific | `2-departments/[dept]/decisions/` |
| Product-specific | `3-products/[product]/decisions/` |
| Project-specific | `4-projects/[project]/decisions/` |

**Format:**
```markdown
## ADR-NNN: Title

**Date:** DD/MM/YYYY
**Status:** Accepted
**Authors:** [names]

### Context
[Why this decision was needed]

### Decision
[What was decided]

### Rationale
[Why this option was chosen over alternatives]
```

Rule: if the decision affects multiple layers, put the ADR in the highest applicable layer and cross-link from the others.

### B7. Cross-link Related Documents

When one document references concepts from another, link to it. Especially:
- Sequence steps → their detail pages
- Roadmap → change management timeline
- Architecture → ADRs that explain design choices

### B8. Markdown Conventions

- H1 (`#`) — page title only, one per file, with `section.page.` prefix
- H2 (`##`) — major sections within the page
- H3 (`###`) — subsections
- Tables for structured comparisons
- Code blocks (` ``` `) for all code, commands, file paths, templates
- No inline HTML

---

## Part C — WRITE Rules

### C1. Never Write Directly to Main/Published

All changes go through an approval process. No exceptions.

- In Git: branch + pull request
- In Confluence: draft + publish approval
- In any system: propose → review → approve → publish

The agent proposes. A human approves. Always.

### C2. Change Proposal Format

Every proposed change must include:

1. **What changed** — specific files/sections modified
2. **Why** — the reason for the change
3. **Who should review** — appropriate owner per layer

### C3. After Proposing a Change

1. Notify the owner via the configured channel
2. Provide the direct link to the review
3. One notification — do not repeat unless asked

### C4. Ownership Matrix

| Layer | Default owner |
|-------|--------------|
| `1-company` | CEO / COO / designated org-wide owner |
| `2-departments` | Department head |
| `3-products` | Product manager / product owner |
| `4-projects` | Project lead |

Configured in `kb-config.yaml`. If unclear, ask — do not guess.

### C5. Pending Changes Are Still WIP

A pending proposal/PR/draft — even if reviewed but not yet approved — is still WIP. Do not cite it as KB content. Mention it exists, but clarify it is not yet authoritative.

---

## Part D — Agent Configuration

### D1. Required at Deployment

| Variable | Description | Example |
|----------|-------------|---------|
| `KB_LOCATION` | Where the KB lives | GitHub repo URL, Confluence space URL |
| `KB_MAIN_REF` | Authoritative branch/state (also read from `kb-config.yaml`) | `main`, `published` |
| `NOTIFY_CHANNEL` | Where to send review notifications (also in `kb-config.yaml`) | Slack channel, email |

Without `KB_LOCATION`, the agent cannot distinguish KB from WIP. Mandatory prerequisite.

The platform implementation (`face-kb-git` vs `face-kb-mcp`) is
chosen by the parent `face-kb` skill from the shape of
`KB_LOCATION`, not from a config key — `kb.platform` was dropped
by ADR-KBM-007. The admin + owner matrix is read straight from
`kb-config.yaml`'s `admins:` and `owners.<layer>:` blocks; see
§E below.

### D2. Relationship to Context Broker

In early deployments (1-5 repos), the agent reads KB content directly via `face-kb-git` or `face-kb-mcp`.

At scale (5+ repos), a Context Broker handles read assembly. The broker serves **only** from KB (main/published) — never from WIP. The rules in this skill apply regardless of whether reads go direct or through a broker.

---

## Part E — Access & Governance (ADR-KBM-007)

FACE separates **intent** from **enforcement**. Agents working on
a KB need to understand which surface owns which concern.

### E1. The hybrid contract

- **FACE owns intent** through three surfaces: the KB Manager's
  `organization_members` (who's in the org), `kb_access` (who
  has what role on which KB), and `kb-config.yaml`'s `admins:`
  + `owners.<layer>:` blocks (the spec).
- **GitHub owns enforcement** through repo collaborators,
  `.github/CODEOWNERS`, and branch protection.
- **Sync is one-way: FACE → GitHub** via a pure projector
  function. Edits to the intent surfaces propagate forward;
  edits to the enforcement surfaces (e.g., a hand-edited
  CODEOWNERS) are not reverse-synced and surface as drift.

### E2. `kb-config.yaml` is the spec

The canonical shape (post-ADR-007):

```yaml
kb:
  name: My Organisation KB
  main_ref: main
  language: en
  org_type: scaleup            # startup | scaleup | enterprise

admins:                         # KB-wide admins (review any layer)
  - alice
  - bob

owners:                         # path-scoped layer reviewers
  1-company:     [ceo]
  2-departments: [dept-lead, co-lead]   # arrays are first-class
  3-products:    pm-1                    # scalar is also valid
  4-projects:    [lead]

notifications:
  channel: '#kb-updates'
  method: slack
```

`kb.platform` and `kb.repository` were removed as redundant
(ADR-KBM-007 §4); legacy configs may still carry them and
readers tolerate them, but the projector ignores them on
write.

### E3. CODEOWNERS is derived

`.github/CODEOWNERS` is regenerated from `admins:` and
`owners.<layer>:` on every spec change. **Never hand-edit it.**
Hand-edits are wiped on the next sync and silently break the
intent-vs-enforcement contract. If approval routing needs to
change, propose an edit to `kb-config.yaml` and the projector
takes care of CODEOWNERS.

### E4. Branch protection is enforced

The PR + 1-codeowner-approval gate is a FACE governance
contract, not a recommendation. Drift (e.g., "Require approval"
toggled off on GitHub) surfaces as a non-dismissable admin-only
banner in the KB Manager with a one-click apply. The agent
should never disable this policy, even with admin permission.

### E5. Drift banners

If the KB Manager UI shows a drift banner ("CODEOWNERS
mismatch," "branch protection loose," "admin missing from
GitHub"), it means the intent surface and the enforcement
surface have diverged. Resolution is one-click via the banner;
don't try to fix it by editing GitHub directly.

---

## Constraints

- Never present WIP as truth — not under time pressure, not because "everyone knows it anyway"
- Never merge/publish without human approval
- Never bypass approval for "small" or "urgent" changes — the process exists for a reason
- Never hand-edit `.github/CODEOWNERS` — it's projector-derived from `kb-config.yaml` (ADR-KBM-007 §E3)
- If KB is unavailable, say so — do not silently fall back to WIP
- All KB content must be written in the language defined in `kb-config.yaml → kb.language`

---

## Companion Skills

| Skill | Purpose |
|-------|---------|
| `face-kb-git` | Git implementation (branch, PR, SUMMARY.md, GitBook, structural checklist) |
| `face-kb-mcp` | MCP implementation (Confluence, Notion, SharePoint) |
| `face-kb-write` | Content routing + extraction (platform-agnostic intake) |
| `face-setup` | First-time KB setup (AI Lead only, run once) |

**Load `face-kb-core` first. Then load the appropriate platform skill.**

---

# KB Write — Content Routing & Extraction

## What This Skill Does

Guides an AI agent (or human) through the process of taking incoming content — a Word doc, Portable Document Format (PDF), Confluence export, Notion export, raw text, or any other format — and placing it correctly in the KB.

This skill handles **what** goes **where** and **how** it should look. The actual commit/PR/publish step is handled by the platform skill (`face-kb-git` or `face-kb-mcp`).

**Do not load this skill directly.** It is loaded by `face-kb` as part of the standard skill set.

## When to Use It

- Someone hands you a file and says "put this in the KB"
- You need to convert non-Markdown content into KB-ready Markdown
- You need to determine where a piece of content belongs in the KB structure
- You are extracting information from a document to populate KB pages

---

## Step 1 — Identify the Content

Ask (or infer from context):

1. **What is this?** Policy, decision, meeting notes, architecture doc, product spec, process guide?
2. **Who owns it?** Which team, department, or product does it belong to?
3. **What format is it in?** Word, PDF, Confluence HTML, Notion export, plain text, Slack thread?

If the source is ambiguous, ask — do not guess the placement.

## Step 2 — Route to the Correct Location

Map the content to the KB layer structure defined in `face-kb-core`:

| Content type | Target layer | Example path |
|-------------|-------------|-------------|
| Company-wide policy, values, strategy | `1-company/` | `1-company/policies/remote-work.md` |
| Department process, team context | `2-departments/[dept]/` | `2-departments/1-engineering/2-processes.md` |
| Product spec, architecture, roadmap | `3-products/[product]/` | `3-products/acme-platform/3-architecture.md` |
| Project scope, timeline, deliverable | `4-projects/[project]/` | `4-projects/q2-migration/1-scope.md` |
| Agent configuration | `5-agents/` | `5-agents/support-bot.md` |
| Decision record | `[layer]/decisions/` | `3-products/acme-platform/decisions/` |

**Rules:**
- Follow the numbering convention from `face-kb-core` (numeric prefix, logical reading order)
- File names: lowercase, hyphens only, descriptive
- If the target folder doesn't exist yet, create it with a `README.md`
- If unsure between two locations, pick the more specific one (product > company, project > product)

## Step 3 — Extract and Convert to Markdown

### From Word (.docx)
- Extract text content, preserving heading hierarchy
- Convert Word headings to Markdown headings (H1 = page title with `section.page.` prefix)
- Convert Word tables to Markdown tables
- Convert bullet lists and numbered lists directly
- Drop formatting that doesn't translate (font colours, text boxes, SmartArt) — preserve the information, not the styling
- Images: note their location and describe what they show; if the KB platform supports images, include them

### From PDF
- Extract text (OCR if needed for scanned documents)
- Reconstruct heading hierarchy from font sizes / bold patterns
- Tables may need manual cleanup — flag if extraction is lossy

### From Confluence / Notion Export (HTML)
- Convert HTML headings, tables, lists, and code blocks to Markdown equivalents
- Strip platform-specific UI elements (Confluence macros, Notion toggles, embedded databases)
- Preserve internal links as cross-references where possible
- Flag any interactive content that cannot be represented in Markdown (e.g. Jira embeds, database views)

### From Slack Thread / Chat
- Summarise the thread — do not dump raw messages into the KB
- Extract decisions, action items, and key context
- Attribute statements where relevant: "Decided by [person] on [date]"
- Link to the original thread if the KB platform supports external links

### From Raw Text / Email
- Structure into sections with appropriate headings
- Identify and separate metadata (dates, authors, recipients) from content
- Apply the KB's Markdown conventions from `face-kb-core`

### General Extraction Rules
- **One topic per file.** If the source document covers multiple unrelated topics, split it.
- **H1 is the page title** — one per file, with `section.page.` prefix matching its position in the folder
- **Remove duplicates.** If content already exists in the KB, update the existing page — do not create a second version.
- **Flag conflicts.** If extracted content contradicts existing KB content, flag it — do not silently overwrite.

## Step 4 — Reflection (mandatory)

Before handing off to the platform skill, evaluate the prepared content against these criteria:

| # | Criterion | Question |
|---|-----------|----------|
| 1 | **Completeness** | Are all required sections present for this content type? |
| 2 | **Accuracy** | Does the content contradict any existing KB page? |
| 3 | **Placement** | Is the target KB layer and path correct per the routing rules above? |
| 4 | **Format** | Does the Markdown follow KB conventions (headings, tables, naming)? |
| 5 | **No duplication** | Is this content already present elsewhere in the KB? |

For each criterion, assign: `pass` / `flag` (fixable issue) / `block` (requires human input).

- Fix all `flag` items before proceeding.
- If any criterion returns `block`, stop and surface the issue to the user — do not commit.
- Append a one-line reflection summary to the change proposal: `[Reflection: X/5 criteria passed. Fixed: ...]`

## Step 5 — Handoff to Platform Skill

Once the content has passed the reflection step:

1. Pass to `face-kb-git` or `face-kb-mcp` for the actual write operation (branch + PR, or draft + publish)
2. Include in the change proposal:
   - **Source:** where the content came from (file name, URL, or description)
   - **What was extracted:** summary of content
   - **Where it was placed:** KB path(s)
   - **Any conflicts or flags:** content that contradicts existing KB pages
   - **Reflection summary:** result of Step 4

---

## Constraints

- Never place content without knowing which layer it belongs to — ask if unclear
- Never overwrite existing KB content without flagging the change
- Never dump raw unstructured content into the KB — always structure it first
- If extraction is lossy (e.g. complex PDF tables), flag what was lost
- All extracted content must be in the KB's designated language (`kb-config.yaml → kb.language`)

---

## Companion Skills

| Skill | Relationship |
|-------|-------------|
| `face-kb-core` | Defines the structure rules this skill routes into |
| `face-kb-git` | Executes the write (branch + PR) for Git KBs |
| `face-kb-mcp` | Executes the write for Confluence/Notion/SharePoint KBs |

---

# face-kb-git — Git Platform Implementation

## What This Skill Does

Implements the KB read and write operations for knowledge bases hosted in Git (GitHub, GitLab, Bitbucket, Azure DevOps). This skill is loaded automatically by `face-kb` when `KB_LOCATION` is a git URL.

**Do not load this skill directly.** Load `face-kb` instead — it handles platform selection.

## When It Applies

- KB is a Git repository (any provider)
- Agent has API access to the Git provider
- Branch protection is enabled on the main branch

---

## READ Operations

### Fetching KB Content

1. **Always read from the main branch** (as defined in `kb-config.yaml → kb.main_ref`)
2. Use the Git provider's API or local clone — whichever is available
3. If the file exists on main → it is truth (`[KB ✓]`)
4. If the file exists only on an open branch/PR → it is WIP (`[WIP — pending PR]`)

### Referencing Files

When citing KB content, include:
- Repository path: `1-company/policies/remote-work.md`
- Branch: `main` (or note if referencing a PR branch)
- Last commit date if available

Format:
> `1-company/policies/remote-work.md` [KB ✓] — last updated 2026-04-10

### Searching KB

Preferred search order:
1. README.md index sections (fastest, structured)
2. File/folder names (descriptive naming convention)
3. Full-text search via API (if supported)
4. Local clone grep (if available)

---

## WRITE Operations

### Branch Strategy

Every change, no matter how small, goes through a branch:

```
main (protected) ← PR ← feat/description (agent works here)
```

**Branch naming convention:**
- `feat/description` — new content
- `fix/description` — corrections, broken links
- `update/description` — content updates to existing files
- `refactor/description` — structural changes (renaming, moving)

### Creating a Change

1. **Create a new branch** from the latest main
2. **Make changes** — commit with descriptive messages
3. **Open a Pull Request** with:
   - Clear title: `type: description` (e.g., `feat: add remote work policy`)
   - Body explaining what changed and why
   - Tag the appropriate owner from the owners matrix
4. **Notify the owner** via the configured notification channel
5. **Wait for approval** — never merge without human approval

### Commit Message Convention

```
feat: new content or section
fix: correct errors, broken links, wrong info
update: refresh existing content with new information
refactor: restructure without changing meaning
```

### If Branch Already Exists

When adding to an in-progress change:
1. Check if the PR is still open
2. If open → push additional commits to the same branch
3. If merged → create a new branch + new PR
4. Update the PR description to reflect all changes

### Merge Rules

- **Agent never merges.** Only the designated human owner merges.
- Preferred merge method: squash merge (clean history)
- After merge: agent can confirm "PR #N merged, content is now in KB"

---

## Git-Specific Structural Rules

These rules apply only to Git-hosted KBs. They supplement the universal structural rules in `face-kb-core`.

### SUMMARY.md

> **⛔ Hard rule — no exceptions:** When adding a new file, `SUMMARY.md` **must** be updated in the **same branch and same commit**. Never in a separate PR, never after the fact.
>
> **Why:** GitBook derives all page URLs exclusively from `SUMMARY.md`. A file that exists in the repo but is absent from `SUMMARY.md` will never appear in GitBook — the sync shows "Synced ✅" but the page returns "Not found". This is silent and hard to diagnose.
>
> **Symptom to recognise:** GitBook shows "Synced ✅" but the page URL returns "Not found" → SUMMARY.md entry is missing.

`SUMMARY.md` at the repo root is the navigation file for GitBook (or similar doc renderers). Update it with **every structural change** (file added, renamed, moved, or deleted).

Rules:
- Section entry points use `README.md` (not the first content page)
- Page labels include `section.page.` numbers matching the title
- Nesting matches the folder hierarchy

```markdown
* [3. Framework](3-framework/README.md)
  * [3.1. Architecture](3-framework/1-architecture.md)
  * [3.2. Skills System](3-framework/2-skills-system.md)
```

### YAML Front Matter

If a file has YAML front matter (between `---` markers), it must parse correctly. GitBook will reject the entire sync if any file has invalid YAML.

Common pitfalls:
- **Colons in values:** `description: Use this for tasks involving: writing` → YAML reads `involving` as a new key. **Always quote values containing colons.**
- **Em dashes:** `—` can be misread as YAML document separator. Use `--` or quote the value.
- **Unescaped quotes:** If the value contains quotes, use the opposite type.

**Rule:** Always quote YAML front matter values that contain special characters (`: — ' "`).

### PR Description

Keep PR descriptions current. If you push additional commits to an open PR, update the PR body to reflect the current state. Reviewers read the body, not the commit history.

---

## Pre-PR Checklist

Before opening any PR, verify:

- [ ] All new folders have `README.md`
- [ ] All folders and files are numbered (prefix matches reading order)
- [ ] All page H1 titles have `section.page.` prefix
- [ ] `SUMMARY.md` updated in **this same commit** (correct paths, README entry points, numbered labels) — ⛔ if adding a new file and this is unchecked, stop and fix before committing
- [ ] Internal cross-references updated (if files renamed/moved)
- [ ] PR body describes all changes accurately
- [ ] YAML front matter valid (values with special characters are quoted)
- [ ] Commit messages follow `feat:/fix:/update:/refactor:` convention

---

## Setup Requirements

### Git Provider Access

The agent needs:
- **Read access** to the repository (for queries)
- **Write access** to create branches and PRs (for changes)
- **API token** with appropriate scopes

| Provider | Required scopes |
|----------|----------------|
| GitHub | `repo` (or fine-grained: contents read/write, pull requests read/write) |
| GitLab | `api` or `read_repository` + `write_repository` |
| Bitbucket | Repository read + write |
| Azure DevOps | Code read + write |

### Branch Protection

**Mandatory and FACE-enforced** (ADR-KBM-007). The main branch
must be protected:
- Require PR before merge
- Require at least 1 codeowner approval
- No direct pushes (not even from admins, ideally)

FACE applies this policy automatically on import and re-applies it
via the drift banner if it's ever loosened on GitHub. If you see a
non-dismissable amber "branch protection loose" banner on the KB
Manager's settings page, that's the projector flagging the drift.
The agent should never disable this policy or merge without an
approval, even with admin permission.

If branch protection is somehow disabled (e.g., legacy KB not yet
imported through FACE):
1. Flag it as a governance violation, not just a security risk.
2. Point the human at the KB Manager's settings page (one-click
   "Apply recommended branch protection").
3. Still use branch + PR workflow regardless (defence in depth).

---

## GitBook Integration (Optional)

If the organisation uses GitBook as a human-readable layer:
- GitBook syncs automatically from the Git repository
- No additional agent configuration needed
- After PR merge → GitBook updates within minutes
- When referencing content for non-technical users, provide the GitBook URL if known

---

## Configuration Reference

These values come from `kb-config.yaml`. The repo URL itself comes
from the agent-side `KB_LOCATION` (the YAML doesn't carry it — see
ADR-KBM-007 for why `kb.platform` and `kb.repository` were dropped):

```yaml
kb:
  name: My Organisation KB
  main_ref: main
  language: en
  org_type: scaleup            # startup | scaleup | enterprise

admins:                         # KB-wide admins (kb_access=admin mirror)
  - alice
  - bob

owners:                         # path-scoped layer reviewers
  1-company:     [ceo]
  2-departments: [dept-lead, co-lead]
  3-products:    [pm-1, pm-2]
  4-projects:    [lead]
```

Legacy configs may still carry `kb.platform: git` and
`kb.repository: …` — readers tolerate these but the projector
(see ADR-KBM-007 §5) ignores them when regenerating
CODEOWNERS / branch protection.

Agent-side configuration:
```
KB_LOCATION: https://github.com/org/knowledge-base
GIT_TOKEN: <api-token>  # stored securely, never in KB
```

---

## Writing via FACE Manager API (ADR-KBM-009) — preferred path

If `kb-config.yaml` carries a `kb.face_manager_url:` field, the KB is
managed by FACE KB Manager and the agent **should prefer the FACE
Manager API over direct git writes**. The API resolves the request
to a pre-mapped FACE user and opens the PR using *that user's*
stored OAuth token, so commit + PR authorship attribute to the
employee — preserving CODEOWNERS routing, `git blame`, and the
audit trail. Direct git writes with the agent's own GitHub identity
break all of that.

### When to use the API vs direct git

| Condition | Path |
|---|---|
| `kb.face_manager_url` is set | **FACE Manager API** (this section) |
| `kb.face_manager_url` is unset | Direct git write (the rest of this skill) |
| API call returns `unauthorized` / `unknown_external_id` | Fall back to direct git write, but flag the gap to the user — the agent isn't registered or the user isn't mapped |

### Authentication

The agent holds an **Ed25519 private key**; FACE Manager has the
public key (registered by an org admin in
`/<orgSlug>/settings/agents`). Every request carries a
short-lived JWT:

```json
{
  "iss": "face_agent_<22-base64url-chars>",
  "iat": <now>,
  "exp": <now + at most 300 seconds>,
  "jti": "<unique nonce>"
}
```

Signed with the agent's private key (alg = `EdDSA`). Sent as:

```
Authorization: Bearer <jwt>
```

Mint a fresh JWT for each request (or cache for <5 min). Token
lifetime is hard-capped at 5 minutes server-side — longer
`exp` values are rejected even with a valid signature.

### Identifying the user the agent is acting for

Every API call must identify the FACE user on whose behalf the
agent is acting:

- **Reads** (`GET …`): header `X-Face-Actor-External-Id: <id>`
- **Writes** (`POST …`): body `{ "actor": { "external_id": "<id>" } }`

`<id>` is whatever identifier the agent's source system uses
(Slack user id `U01ABCDEF`, Teams oid, GitHub login, the agent's
own internal user id — opaque to FACE). The mapping from
`external_id` to FACE user is **pre-registered by an org admin**;
the agent cannot create or alter mappings on its own.

### Endpoints

```
POST   /api/agent/kb/:orgSlug/:kbSlug/propose-change
GET    /api/agent/kb/:orgSlug/:kbSlug/tree?path=<dir>
GET    /api/agent/kb/:orgSlug/:kbSlug/file?path=<file>
```

#### `POST .../propose-change` — open a PR

```jsonc
// Request body:
{
  "actor": { "external_id": "U01ABCDEF" },
  "files": [
    { "path": "1-company/3-strategy.md", "content": "# 1.3. Strategy\n…" }
  ],
  "prTitle": "feat: add company strategy doc",
  "prBody": "Drafted by openclaw on behalf of @alice…",
  "branchName": "ai/openclaw/strategy-doc-9bz1k4"   // optional
}

// Response (200):
{
  "prUrl": "https://github.com/…",
  "prNumber": 73,
  "branch": "ai/openclaw/strategy-doc-9bz1k4"
}
```

#### `GET .../tree?path=1-company` — list a directory

```jsonc
// Response (200):
{
  "entries": [
    { "path": "1-company/README.md", "type": "file", "name": "README.md", "sha": "…" },
    { "path": "1-company/policies", "type": "dir",  "name": "policies" }
  ]
}
```

System files (`.git/`, `0-meta/`, `AGENTS*.md`, `CLAUDE.md`, etc.)
are filtered out — same default as the FACE web UI.

#### `GET .../file?path=1-company/policies/remote-work.md`

```jsonc
// Response (200):
{
  "path": "1-company/policies/remote-work.md",
  "sha": "…",
  "content": "# 1.1. Remote-work policy\n…"
}
```

### Error codes the agent should branch on

| Code | Meaning | Recommended agent action |
|---|---|---|
| `unauthorized` | JWT invalid/expired or agent revoked | Stop. Tell the user the agent's credentials need refresh. |
| `unknown_external_id` | No mapping for this `external_id` | Tell the user to ask an admin to add a mapping in `/<org>/settings/agents`. |
| `kb_not_found` | KB doesn't exist | Confirm `orgSlug`/`kbSlug`. |
| `actor_no_kb_access` | Mapped user has no read/write access to this KB | Tell the user to ask an admin to grant access in the Members matrix. |
| `needs_github_link` | Mapped user hasn't linked GitHub to FACE | Tell the user to sign back in to FACE with GitHub. |
| `oauth_app_not_approved_for_org` | FACE OAuth app not authorized on the GitHub org | Tell the user to ask a GitHub-org admin to approve the FACE OAuth app at github.com/organizations/<org>/settings/oauth_application_policy. |
| `github_error` | Generic GitHub-side failure | Surface the underlying message; retries OK after a brief delay. |
| `invalid_body` / `missing_path` / `missing_actor` | Request shape error | Fix the request and retry. |

### Why this exists (one-paragraph rationale)

FACE owns access intent; GitHub owns enforcement (ADR-KBM-007). For
the access model to work, **the user identity making changes must be
the *actual employee*, not the agent's shared service account**. The
API does this by resolving the agent's identity-claim to a curated
mapping, then using the *employee's* stored OAuth token to talk to
GitHub. The agent never touches the employee's credentials; the
employee never has to paste a PAT into the agent. ADR-KBM-009.

---

## Access & Governance (ADR-KBM-007)

The git surface is the **enforcement layer**, not the source of
truth. FACE owns intent (`organization_members` + `kb_access` in
the KB Manager DB, plus `admins:` and `owners.<layer>` in
`kb-config.yaml`); GitHub owns enforcement (repo collaborators,
CODEOWNERS, branch protection). A pure projector function syncs
FACE → GitHub one-way; the reverse direction surfaces as a drift
banner an admin must resolve.

**Practical rules for the agent:**

- **Never write `.github/CODEOWNERS` directly.** It's a derived
  artifact, regenerated from `kb-config.yaml`'s `admins:` +
  `owners.<layer>:` blocks by the projector. Hand-edits will be
  overwritten on the next sync — and worse, they break the
  one-way contract. If approval routing needs to change, edit
  `kb-config.yaml` and open a PR; the projector takes care of
  the rest.

- **`owners.<layer>` is `string | string[]`.** Multi-owner is
  supported and intentional. Don't collapse arrays to scalars
  when editing the YAML.

- **`admins:` is the KB-wide admin list** (top-level sibling of
  `owners:`). Distinct from `organization_members` admins in the
  KB Manager DB — the YAML lists them by GitHub login so AI
  agents can read the spec without consulting FACE.

- **Branch protection drift** (e.g., "Require 1 approval"
  toggled off on GitHub) is the projector's responsibility to
  detect and the admin's responsibility to apply. The agent
  should flag it, not silently work around it.

---

## Constraints

- Never push directly to main, even if branch protection is misconfigured
- Never merge without human approval
- Never store API tokens or secrets in the KB repository
- Never hand-edit `.github/CODEOWNERS` — it's regenerated by FACE's projector from `kb-config.yaml`
- If Git API is unavailable (outage), say so — do not fall back to cached/stale content without marking it

---

## On-Demand Companion Skills

These skills are **not** embedded above and are **not** auto-loaded — they are loaded
only when a specific task calls for them, to keep the always-on context lean.

### `face-kb-decisions` — promote design decisions into ADRs

**When:** after a feature designed with a spec/design doc is implemented, to record its
architecturally-significant decisions in the relevant product's decisions log.

**Load it on demand** from the local copy snapshotted into this KB:
[`0-meta/skills/face-kb-decisions/SKILL.md`](0-meta/skills/face-kb-decisions/SKILL.md)

It proposes draft ADRs (`Status: Proposed`) via a PR; a human ratifies by merging.
Depends on the embedded `face-kb-write` rules.
