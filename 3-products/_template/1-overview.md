---
data_status: placeholder
# Options: placeholder | plausible | verified
# placeholder = structure only, no real content yet
# plausible   = content exists but has not been confirmed by the team
# verified    = confirmed accurate by the document owner
# Agents: always declare data_status when creating or updating content.
#         Never present plausible content as verified.
---

# [Product Name] — Overview

**Status:** <!-- draft | approved -->
**Owner:** <!-- GitHub username -->
**Last reviewed:** <!-- YYYY-MM-DD -->

---

## What It Is

<!-- One paragraph: what does this product do? Be concrete. -->

## Who It's For

**Primary users:** <!-- who uses this and why -->
**Secondary users:** <!-- if applicable -->

## Current Status

**Phase:** <!-- Alpha / Beta / GA / Sunsetting -->
**Availability:** <!-- Internal only / Limited access / Public -->
**Known limitations:** <!-- What doesn't work yet or is intentionally out of scope -->

## Key Metrics

| Metric | Current | Target |
|--------|---------|--------|
| <!-- e.g. Active users --> | <!-- value --> | <!-- value --> |
| <!-- e.g. Uptime --> | <!-- value --> | <!-- value --> |

## Purpose for Agents

> This section is for AI agents, not humans. It provides navigation shortcuts and behavioural rules specific to this product.

### Navigation shortcuts

| Question type | Where to look |
|--------------|---------------|
| How is [component] built? | `2-architecture.md` |
| What decisions shaped [design choice]? | `3-decisions/` |
| What’s planned for [feature]? | `4-roadmap.md` |
| What are the API endpoints for [function]? | `5-api.md` |

### Rules for agents working on this product

<!-- List any product-specific rules agents must follow. Examples: -->
<!-- - Never suggest changes to the public API without flagging backwards-compatibility impact -->
<!-- - All architectural decisions must be documented as ADRs in 3-decisions/ -->
<!-- - Do not modify the roadmap without explicit product owner approval -->

- <!-- Rule: [describe the rule and why it exists] -->
- <!-- Rule: [describe the rule and why it exists] -->

### When in doubt

<!-- Who should agents escalate to when they encounter an ambiguous situation? -->

Escalate to: <!-- [Name / Role] via [channel] -->
