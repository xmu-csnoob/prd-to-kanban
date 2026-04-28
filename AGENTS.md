# AGENTS.md

This repository packages `prd-to-kanban` for both Claude Code and Codex.

## Purpose

`prd-to-kanban` converts PRDs and feature requirements into:

- `work/kanban.md` - dynamic, agent-editable Kanban board
- `work/SUBAGENT.md` - static shared context for subagents

It is planning-only. Do not implement tasks as part of the skill workflow.

## Compatibility Surfaces

### prd-to-kanban (standalone)
- Standalone skill entry: `SKILL.md`
- Shared skill source of truth: `plugins/prd-to-kanban/SKILL.md`
- Claude Code plugin: `plugins/prd-to-kanban/.claude-plugin/plugin.json` + `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md`
- Codex plugin: `plugins/prd-to-kanban/.codex-plugin/plugin.json` + `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md`

### praxiskit (full pipeline)
- Claude Code plugin: `plugins/praxiskit/.claude-plugin/plugin.json` + `plugins/praxiskit/skills/`
- Codex plugin: `plugins/praxiskit/.codex-plugin/plugin.json` + `plugins/praxiskit/skills/`
- Skills: `seed-to-idea`, `idea-to-prd`, `prd-to-kanban`, `kanban-to-agents`, `build-to-review`

### Marketplace catalogs
- Claude Code: `.claude-plugin/marketplace.json`
- Codex: `.agents/plugins/marketplace.json`

PraxisKit is an intent-to-acceptance chain: capture idea, write PRD, generate Kanban, orchestrate agents, then present the final result for user acceptance. Both plugins are listed in the marketplace catalogs.

`plugins/prd-to-kanban/SKILL.md` is the source of truth for prd-to-kanban behavior. Keep root `SKILL.md` and `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md` as thin entries.

**Note on skill naming:** `plugins/praxiskit/skills/prd-to-kanban/` and `plugins/prd-to-kanban/skills/prd-to-kanban/` both register a skill named `prd-to-kanban`. Install one plugin or the other — not both. The PraxisKit plugin bundles all pipeline steps (including prd-to-kanban); the standalone plugin is for users who only need prd-to-kanban without the full chain.
