---
data_status: placeholder
# Options: placeholder | plausible | verified
# placeholder = structure only, no real content yet
# plausible   = content exists but has not been confirmed by the team
# verified    = confirmed accurate by the document owner
# Agents: always declare data_status when creating or updating content.
#         Never present plausible content as verified.
---

# AI Usage Policy

**Status:** <!-- draft | approved -->
**Owner:** <!-- GitHub username -->
**Last reviewed:** <!-- YYYY-MM-DD -->
**Review cadence:** Quarterly

---

## Approved AI Tools

| Tool | Approved use | Restrictions |
|------|-------------|--------------|
| <!-- e.g. Claude --> | <!-- Code, writing, analysis --> | <!-- No confidential data --> |
| <!-- e.g. GitHub Copilot --> | <!-- Code completion --> | <!-- No production secrets --> |

## Data Classification for AI Use

| Classification | Definition | AI use permitted? |
|----------------|-----------|------------------|
| Public | Information already public | ✅ Yes |
| Internal | Non-sensitive internal information | ✅ Yes, with approved tools |
| Confidential | Client data, financials, PII | ⚠️ Approved enterprise tools only |
| Restricted | Legal, regulatory, HR, NDA-covered | ❌ No AI processing |

## What AI Agents Should Always Do

- Load the relevant KB section before answering operational questions
- Respect data classification above
- Flag uncertainty rather than guess
- All AI-assisted changes go through human review before taking effect

## What AI Agents Should Never Do

- Process Restricted data under any circumstances
- Take external actions (send emails, payments, communications) without explicit human approval
- Override a human decision

## Notes for Agents

This policy applies to all AI agents operating within this organisation. When uncertain whether a task falls within approved scope, default to flagging and asking rather than proceeding.
