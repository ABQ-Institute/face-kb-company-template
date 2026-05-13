---
data_status: placeholder
# Options: placeholder | plausible | verified
# placeholder = structure only, no real content yet
# plausible   = content exists but has not been confirmed by the team
# verified    = confirmed accurate by the document owner
# Agents: always declare data_status when creating or updating content.
#         Never present plausible content as verified.
---

# [Agent Name]

<!-- One sentence: what does this agent do? -->

## Configuration

**KB_LOCATION:** <!-- https://github.com/your-org/your-kb -->
**Platform:** <!-- OpenClaw / Claude Code / Cursor / Custom -->
**Deployed by:** <!-- GitHub username -->
**Deployed on:** <!-- YYYY-MM-DD -->

## Skills Loaded

| Skill | Purpose | Mode |
|-------|---------|------|
| `face-kb` | KB access + source-of-truth protocol | Always |
| `face-kb-core` | Structural rules | Auto-loaded |
| `face-kb-write` | Content routing | Auto-loaded |
| `face-kb-git` | Git operations | Auto-loaded |
| <!-- custom skill --> | <!-- purpose --> | <!-- when --> |

## Permissions

**Can read:** <!-- which layers -->
**Can propose (PR):** <!-- which layers -->
**Cannot access:** <!-- restricted layers or files -->

## Channels

<!-- Where does this agent operate? -->

- <!-- e.g. Slack #general -->
- <!-- e.g. Claude Code (local) -->

## Constraints

<!-- Any specific rules or limitations for this agent beyond the defaults? -->

- <!-- Constraint -->
