---
data_status: placeholder
# Options: placeholder | plausible | verified
# placeholder = structure only, no real content yet
# plausible   = content exists but has not been confirmed by the team
# verified    = confirmed accurate by the document owner
# Agents: always declare data_status when creating or updating content.
#         Never present plausible content as verified.
---

# [Product Name] — Architecture

**Status:** <!-- draft | approved -->
**Owner:** <!-- GitHub username -->
**Last reviewed:** <!-- YYYY-MM-DD -->

---

## Tech Stack

| Layer | Technology | Reason |
|-------|-----------|--------|
| Frontend | <!-- e.g. Next.js --> | <!-- why --> |
| Backend | <!-- e.g. FastAPI --> | <!-- why --> |
| Database | <!-- e.g. PostgreSQL --> | <!-- why --> |
| Infra | <!-- e.g. AWS, Hetzner --> | <!-- why --> |

## Components

<!-- Describe the main components and how they interact -->

## Data Flow

<!-- How does data move through the system? -->

## External Dependencies

| Service | Purpose | Owner | SLA |
|---------|---------|-------|-----|
| <!-- Service --> | <!-- What we use it for --> | <!-- team --> | <!-- uptime guarantee --> |

## Deployment

**Environments:** <!-- dev / staging / prod -->
**Deploy process:** <!-- how do changes get to production? -->
**Rollback process:** <!-- how do we roll back if something breaks? -->
