---
data_status: placeholder
# Options: placeholder | plausible | verified
# placeholder = structure only, no real content yet
# plausible   = content exists but has not been confirmed by the team
# verified    = confirmed accurate by the document owner
# Agents: always declare data_status when creating or updating content.
#         Never present plausible content as verified.
---

# [Product Name] — API Reference

**Status:** <!-- draft | approved -->
**Owner:** <!-- GitHub username -->
**Last reviewed:** <!-- YYYY-MM-DD -->
**Base URL:** <!-- https://api.example.com/v1 -->

---

## Authentication

<!-- How do clients authenticate? -->

## Endpoints

### [Endpoint Group]

#### `GET /endpoint`

**Description:** <!-- what it does -->
**Auth required:** <!-- yes / no -->

**Request:**
```json
{
  "param": "value"
}
```

**Response:**
```json
{
  "result": "value"
}
```

**Errors:**

| Code | Meaning |
|------|---------|
| 400 | <!-- bad request --> |
| 401 | <!-- unauthorized --> |
