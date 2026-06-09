# face-kb-decisions — Promote Design Decisions into ADRs

## What This Skill Does

Takes a **design document** (a Superpowers spec, a PRD, or any design note) and
promotes its *architecturally-significant* decisions into **draft ADRs** in the
KB's product decisions log. The agent **proposes**; a human **ratifies** by merging
the PR. This closes the gap between where a decision is made (the repo) and where it
must live durably (the KB).

**This is an on-demand companion skill.** It is NOT auto-loaded by `face-kb`. Load it
when you have a design document whose decisions should be recorded — typically at the
end of a feature, after the work is implemented and merged.

**Guiding principle:** Repo = where a decision is born and implemented. KB = where it
is ratified and lives durably. Implementation plans and code never cross into the KB —
only the distilled decision + rationale does.

## When to Use It

- A feature was designed with a spec/design doc and is now implemented.
- Someone says "record these decisions" / "add this to the decisions log."
- You finished a development branch that had a design document.

Do NOT use it for: reversible tuning, pure implementation detail, or content that
belongs in product/architecture pages rather than the decisions log.

## Inputs

- **Design document path** — given by the user, or the spec of the feature just finished.
- **KB location** — from the agent's `KB_LOCATION` / `kb-config.yaml` (same as `face-kb`).

If no design document is provided and none can be located, ask for one. Do not invent decisions.

## Procedure

### Step 1 — Locate the design document

Use the path provided. If finishing a feature and no path was given, look for a spec
under the project's design-doc location (e.g. `docs/superpowers/specs/*-design.md`).
If you cannot find exactly one, ask the user which document to promote.

### Step 2 — Surface candidate decisions

This skill is **Superpowers-aware but not locked**: it reads a Superpowers `## Decisions (locked)` table when one is present, and otherwise reasons over any design document's prose.

- **Superpowers spec (happy path):** read the `## Decisions (locked)` table. Each row
  is a candidate.
- **Any other design doc (fallback):** reason over the prose and list each distinct
  decision the document commits to. Present the candidate list and proceed.

### Step 3 — Filter for architectural significance

Keep a candidate ONLY if it satisfies **at least one** of these (Nygard):

1. Affects **structure** or **boundaries** (module split, layering, what-talks-to-what).
2. Changes an important **non-functional** characteristic — privacy, security, cost, latency.
3. Adds or removes an external **dependency** or **contract**.
4. Is **costly to reverse**, or had **serious rejected alternatives**.

DROP: reversible tuning (timeouts, sizes, constants), implementation detail, local
naming. These stay in the source document, not the decisions log.

State, per candidate, which test(s) it passed or why it was dropped. Never drop silently.

### Step 4 — Discover the target decisions log

- Find the product folder for the work (ask if ambiguous).
- Locate its decisions log: canonical KBs use `<product>/8-decisions-log/decisions.md`;
  template-derived KBs use `<product>/3-decisions/`. (This product-specific location supersedes the generic `[layer]/decisions/` entry in `face-kb-write`'s routing table.) Read the existing entries to learn:
  - the **numbering prefix** (e.g. `ADR-KBM`, `ADR`) and the next free number,
  - the exact **section format** in use.
- If no decisions log exists, follow `face-kb-write` to create one (with a `README.md`).

### Step 5 — Distill each kept decision into a draft ADR

For each kept decision, write an entry **in the exact format Step 4 discovered** — match the
existing log's field names and section order. Do not impose a different template (some logs use
`**Decider:**`, others `**Authors:**`; some include a `### Rationale` section, others fold the
why into Context). When the log already has entries, copy their shape.

If the product has **no** decisions log yet, follow the canonical template in `face-kb-core`
section B6 (`## ADR-NNN: Title`, then `**Status:**` / `**Date:**` / `**Authors:**`, then
`### Context` / `### Decision` / `### Rationale`, adding `### Consequences` and
`### Rejected alternatives`).

In all cases:
- Set `**Status:** Proposed` and use the next free number for the discovered prefix.
- The Context (or equivalent) carries the business need from the document's Goal/Problem/Constraints — WHY this came up.
- The decision body is plain language — **no code, no file paths, no test contracts.**
- Capture the rejected alternatives from the document or the product's `rejected-alternatives` page.
- End with a source link: `*Promoted from [<spec title>](<repo-relative-path>) @ <commit-sha>.*`

Distill — never paste the spec verbatim. The reader is a human skimming the decisions log.

### Step 6 — Open the PR (delegate the write)

Hand the prepared entries to `face-kb-write`, which routes to the platform skill
(`face-kb-git` / `face-kb-mcp`). Branch + PR, **never** `main`. PR title:
`docs(decisions): propose ADRs from <spec name>`. PR body lists each proposed ADR and
each decision intentionally left in the spec.

### Step 7 — Annotate the source + report

- In the source document, append to each promoted decision:
  `→ Proposed as <PREFIX>-0NN (PR #…)`.
- Report to the human: which ADRs were proposed, which decisions were left in the spec
  and why. No silent drops.

## Human ratification (outside this skill)

The PR is the ratification gate:
- **Merge** → flip each `Status: Proposed` to `Accepted` (in the PR or a trivial follow-up).
- **Close** → record the decision under the product's `rejected-alternatives` so it is not re-litigated.

## Companion Skills

| Skill | Relationship |
|-------|--------------|
| `face-kb-core` | Defines the ADR format/template (section B6) + source-of-truth rules; KB = truth. |
| `face-kb-write` | Routes and executes the KB write (branch + PR). Required. |
| `face-kb-git` / `face-kb-mcp` | Platform write implementation, invoked via `face-kb-write`. |

---

*Owner: Adrian Erimescu | On-demand companion of `face-kb`.*
