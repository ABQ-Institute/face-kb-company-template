---
data_status: placeholder
# Options: placeholder | plausible | verified
# placeholder = structure only, no real content yet
# plausible   = content exists but has not been confirmed by the team
# verified    = confirmed accurate by the document owner
# Agents: always declare data_status when creating or updating content.
#         Never present plausible content as verified.
---

# 1.4.2. Security Policy

**Status:** <!-- draft | approved -->
**Owner:** <!-- GitHub username -->
**Last reviewed:** <!-- YYYY-MM-DD -->

---

## Access Control

**Credential storage:** <!-- Where are credentials stored? e.g. 1Password, Bitwarden -->
**Sharing credentials:** <!-- Policy on sharing — never in Slack, etc. -->
**Offboarding:** <!-- What happens to access when someone leaves? -->

## Data Handling

**Sensitive data definition:** <!-- What counts as sensitive in your org? -->
**Where sensitive data can live:** <!-- e.g. "Only in approved cloud storage, not in chat" -->
**Agent data access:** <!-- What data can agents access? What is off-limits? -->

## Agent Security Rules

- Agents must NEVER store or log credentials
- Agents must NEVER send sensitive data to external services without explicit approval
- Agents must flag when they encounter data that appears sensitive

---

*Instructions for AI Lead: Be specific about what agents can and cannot access. Vague security policies create vague agent behaviour.*
