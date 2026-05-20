# ADR-001: FACE Open-Source Licensing and Architecture Boundary Design

**Date:** 07/05/2026 (decision) — 13/05/2026 (licence file drafted) — 20/05/2026 (documented)
**Status:** Accepted
**Authors:** Val Muresan, Mihai, ABQ Institute
**Source:** Slack #eacf thread, 2026-05-07 to 2026-05-13

---

## Context

ABQ Institute decided to open-source the FACE (Framework for AI Context in Enterprise) framework. Before publishing, two interrelated questions needed resolution:

1. Which open-source licence to use
2. How to architect FACE so that the licence boundary maps cleanly onto the code structure

---

## Decisions

### 1. Licence: GNU Lesser General Public License v3.0 (LGPL-3.0)

FACE will be published under LGPL 3.0, with copyright held by **ABQ Institute**.

### 2. Mechanisms-only / policy-agnostic design invariant

FACE mechanisms encode *how* to enforce a rule — never *what* the rule is. Policy values, thresholds, jurisdictional settings, credentials, and org-specific identifiers are injected at runtime via a defined configuration interface. They never live inside FACE itself. This is a design invariant, not a convention.

### 3. Four-layer architecture

```
LAYER 3 — Runtime Injection (private, proprietary)
  org policy values, model credentials, tenant configs
         │ injected at runtime via config interface
LAYER 2 — EACF Mechanisms (LGPL core)
  · Onboarding mechanisms
  · Core knowledge-building mechanisms
  · Transversal mechanisms (GDPR, compliance, legal, audit, governance)
         │
  ┌──────┴──────┐
LAYER 1a        LAYER 1b
LLM Adapters    Utility Adapters
OpenAI, Azure,  Jira, Confluence,
Anthropic,      Git, SharePoint,
local LLMs…     Teams, MS Suite…
```

### 4. Transversal compliance mechanisms live in Layer 2 as cross-cutting interceptors

Compliance, GDPR, audit, and governance mechanisms are part of the LGPL core. They expose interceptor hooks / middleware contracts that other Layer 2 subtrees register against. The hook pattern — not hardwired calls — is what keeps them jurisdiction-neutral and policy-agnostic.

### 5. Compliance kit to accompany the licence

To make the LGPL boundary auditable and self-enforceable:
- SPDX headers (`LGPL-3.0-or-later`) on all mechanism files
- `face-manifest.json` — canonical list of mechanism files + hashes for diff auditing
- `CONTRIBUTING.md` — CLA requirement + mechanism boundary rules
- CLA bot on all PRs (GitHub CLA Assistant)

---

## Rationale

### Why LGPL over Apache 2.0

| Concern | Apache 2.0 | LGPL 3.0 |
|---|---|---|
| Enterprise adoption friction | Minimal | Low (understood by mature legal teams) |
| Prevents silent forks of mechanisms | ✗ | ✓ |
| Org proprietary configs protected | ✓ | ✓ |
| Improvements to mechanisms return to commons | ✗ | ✓ |
| Patent retaliation protection | ✗ | ✓ (via GPL 3.0 inheritance) |
| Maps to library linking intent | Loosely | Directly |

LGPL was designed for shared libraries that get linked into proprietary systems. FACE is exactly that use case in the AI context layer. The mechanisms-only / policy-agnostic design means copyleft lands precisely at the mechanism boundary and never touches the org's proprietary layer.

### Why LGPL 3.0 over 2.1

LGPL 3.0 adds explicit patent retaliation clauses — relevant for an AI governance framework where patent disputes in the compliance/governance domain are foreseeable. LGPL 3.0 also interacts better with GPL 3.0 licensed dependencies. The anti-tivoization clause (hardware lockdown) is not directly relevant but does no harm.

### Why split Layer 1 into 1a (LLM) and 1b (Utility)

- Different versioning cadence: LLM APIs change fast; Jira/Confluence are comparatively stable
- Different interface contracts: LLM adapters normalise prompt→response+tokens; utility adapters handle query/write ops with auth, pagination, rate limits
- Different contributor pools
- Allows orgs to substitute one adapter family without touching the other

---

## Consequences

1. Before publishing FACE as LGPL, the codebase must be audited to enforce the Layer 2/Layer 3 boundary — no org-specific values, model names, or policy defaults inside mechanism files.
2. A `LICENSE` file must be added to the repo root (FACE preamble + full LGPL 3.0 + GPL 3.0 texts).
3. `README.md` must reference the licence.
4. `CONTRIBUTING.md`, `face-manifest.json`, and CLA bot must be added as part of the compliance kit.
5. The adapter interface contracts (Layer 1a and 1b) must be formally defined before open-sourcing so forks know what the substitution boundary is.
6. Architectural review of mechanism subtrees is required — flagging any transversal compliance mechanism that is currently implemented as a hardwired call rather than an interceptor hook.

---

## Status History

| Date | Status | Note |
|---|---|---|
| 2026-05-07 | Proposed | Licensing discussion started in #eacf |
| 2026-05-07 | Accepted (LGPL) | Mihai +1, Val +1 |
| 2026-05-13 | Refined | LGPL 3.0 chosen over 2.1; FACE preamble drafted; compliance kit scoped |
| 2026-05-20 | Documented | Written to ADR by Luna |
