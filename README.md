# FACE Company Knowledge Base

> Built on [ABQ's FACE](https://abq.institute/face) — the Framework for AI COntext in Enterprise by [ABQ Institute](https://abq.institute).

---

## What this is

A structured knowledge base template built on FACE. Clone it, fill it in, and your AI agents know who you are, what you build, and how you work — from day one.

No configuration wizards. No integrations to set up. Just a folder your agents understand.

## What's inside

```
0-meta/          Configuration — kb-config.yaml tells agents where they are
1-company/       Who you are: identity, structure, strategy, policies, brand
2-departments/   How your teams work: processes, responsibilities, decisions
3-products/      What you build: specs, architecture, roadmap, ADRs
4-projects/      What you're working on: scope, plan, retrospectives
5-agents/        Which agents run here and what they're allowed to do
```

Each layer has a `_template/` subfolder. Copy it to add a department, product, or project.

## How to start

1. Click **"Use this template"** → create your organisation's KB repo
2. Clone it locally
3. Open any AI tool (Claude Code, Cursor, Gemini, Copilot) in the repo root
4. Start with this prompt:
   > "Read AGENTS_SETUP.md and guide me through setting up this FACE knowledge base."
5. The agent checks `kb-config.yaml` → reads `AGENTS_SETUP.md` (setup mode) or `AGENTS_BOOT.md` (operations mode)
6. Fill in the template files with your actual content
7. Your agents have context. Start working.


## Two-file boot pattern

| File | When it's read | Delete after? |
|------|---------------|---------------|
| `AGENTS_SETUP.md` | First time only — walks you through `kb-config.yaml` setup | ✅ Yes — delete after setup |
| `AGENTS_BOOT.md` | Every session — all KB operations | ❌ Never |

## Supported tools

| Tool | Auto-loaded file |
|------|-----------------|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex | `AGENTS.md` |
| Gemini CLI | `GEMINI.md` |
| Cursor | `.cursor/rules/face.mdc` |
| GitHub Copilot | `.github/copilot-instructions.md` |

All files point to `AGENTS_BOOT.md` — one place to update, all tools stay in sync.

---

*Built by [ABQ Institute](https://abq.institute) · [ABQ's FACE](https://abq.institute/face)*
