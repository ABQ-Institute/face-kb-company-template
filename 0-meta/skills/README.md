# FACE Companion Skills (snapshot)

On-demand FACE companion skills, snapshotted into this KB so agents can load
them locally without reaching ABQ's (private) canonical skills repo.

These files are part of the FACE skills bundle tracked by
[`0-meta/.face-skills-version`](../.face-skills-version). When that marker is
behind the canonical `SKILLS_VERSION`, the snapshot is stale and should be
refreshed — both the embedded skills in `AGENTS_BOOT.md` and the files in this
folder. The FACE KB Manager surfaces this drift in a banner; its one-click
refresh PR currently updates the embedded skills in `AGENTS_BOOT.md` (extending
it to also rewrite this folder is a tracked follow-up).

| Skill | Load | Purpose |
|-------|------|---------|
| [`face-kb-decisions`](face-kb-decisions/SKILL.md) | on-demand | Promote a design doc's architecturally-significant decisions into draft ADRs (`Status: Proposed`) via a PR. |
