---
data_status: placeholder
# Options: placeholder | plausible | verified
# placeholder = structure only, no real content yet
# plausible   = content exists but has not been confirmed by the team
# verified    = confirmed accurate by the document owner
# Agents: always declare data_status when creating or updating content.
#         Never present plausible content as verified.
---

# [Process Name] — Runbook

**Status:** <!-- draft | approved -->
**Owner:** <!-- GitHub username -->
**Trigger:** <!-- What starts this process? e.g. "Customer reports missing output", "Automated alert fires", "Scheduled run at HH:MM" -->
**Last reviewed:** <!-- YYYY-MM-DD -->

---

## What Is This Process?

<!-- One paragraph: what does this process do, and what is the expected end result?
     Example: "The settlement pipeline generates and delivers accounting files to customers.
     End result: customer receives their file within [timeframe]." -->

## Pipeline / Stages

<!-- If this process has sequential stages, describe them here as a flow.
     Use a code block for clarity. Example:

```
Stage 1: [Name] → Stage 2: [Name] → Stage 3: [Name] → Output
```
-->

```
<!-- [Stage 1] → [Stage 2] → [Stage 3] → [Expected output] -->
```

---

## Failure Definition

<!-- One sentence: what does failure look like from the end user's perspective?
     Example: "Customer does not receive their file." -->

---

## Failure Modes

<!-- Map each stage to its known failure modes. -->

| Stage | Failure Mode |
|-------|-------------|
| <!-- Stage 1 --> | <!-- What can go wrong here? --> |
| <!-- Stage 2 --> | <!-- What can go wrong here? --> |
| <!-- Stage 3 --> | <!-- What can go wrong here? --> |

---

## Detection

<!-- How does the team find out about failures?
     Be specific: automated alerts, customer complaints, manual checks, etc. -->

| Failure type | How detected | Alert recipient |
|-------------|-------------|----------------|
| <!-- e.g. Stage 1 failure --> | <!-- e.g. Automated email alert --> | <!-- Who gets notified? --> |
| <!-- e.g. Ghost failure (no alert) --> | <!-- e.g. Customer complaint / ticket --> | <!-- Who gets notified? --> |

**Detection gaps:**
<!-- List any failure types that currently have no automated detection — these are your blind spots. -->
- <!-- e.g. Secondary process failures have no alert — detected only via customer complaint -->

---

## Resolution Steps

<!-- Step-by-step instructions to diagnose and fix the failure.
     Work backwards from the expected output if helpful (reverse-engineering approach). -->

1. **Check [end state]** — <!-- e.g. "Check destination folder — is the expected file present?" -->
2. **Check [stage N]** — <!-- e.g. "Verify the delivery step completed successfully" -->
3. **Check [stage N-1]** — <!-- e.g. "Check file creation logs for errors" -->
4. **Check [stage 1]** — <!-- e.g. "Check database for stored procedure errors" -->

**Time estimate:** <!-- e.g. "~20 minutes per case under normal conditions" -->

**Who can run this:** <!-- e.g. "Any member of [Team Name] with server access" -->

---

## Verification

<!-- How do you confirm the fix worked? -->

- **Post-fix check:** <!-- e.g. "Manually verify output is present and correct" -->
- **Record:** <!-- e.g. "Log the resolution in [ticket system] with ticket number" -->

---

## Known Gaps

<!-- Document structural problems that aren't fixed yet — no automated detection,
     missing dashboards, manual steps that should be automated, etc.
     This section is honest about what the process can't do. -->

- <!-- e.g. "No automated post-fix verification — manual check required" -->
- <!-- e.g. "No single live view across all customers — must check individually" -->
- <!-- e.g. "Ghost failures have no detection mechanism" -->

---

## Related

<!-- Links to related processes, tools, or documentation -->

- <!-- e.g. [Overview](../1-overview.md) — team scope and responsibilities -->
- <!-- e.g. [Tool name] — link to internal tool or system -->
