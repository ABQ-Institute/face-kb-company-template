# Knowledge Base — Agent Instructions

> Built on ABQ's FACE. Skills are served centrally and are always current.

## Boot

Read `0-meta/kb-config.yaml` (config, owners, language).

## Load the FACE skills (one call, cache for the session)

Fetch this ONCE at the start of your session and keep it in context — do **not**
re-fetch per operation:

    https://face.abq.institute/api/skills/face-kb/bundle?platform=git

It returns the full skill set (face-kb + face-kb-core + face-kb-write +
face-kb-git) in one document. The response is version-stamped (`ETag`
and `X-Skills-Version`). Re-fetch only when a new session starts or you learn
the version changed; a conditional request (`If-None-Match: "<version>"`)
returns `304 Not Modified` when it is unchanged.

(API agents: `curl` it. Web tools: open the URL and paste the contents in.)

## Non-negotiable rules

- Never write directly to `main`. All changes go through a branch + PR.
- Never merge or approve your own PR.
- Write KB content in the language set in `kb-config.yaml`.

---
*Skill content lives in FACE Studio (served at /api/skills/*), not in this file.*
