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
- Core transforms: `seed-to-idea`, `idea-to-prd`, `seed-to-task-graph`, `prd-to-task-graph`, `task-graph-to-batch`, `batch-to-build`, `review-and-accept`
- Driver skill: `next-iteration` (single-step default; budget mode for bounded multi-step loops)

### Marketplace catalogs
- Claude Code: `.claude-plugin/marketplace.json`
- Codex: `.agents/plugins/marketplace.json`

PraxisKit is an intent-to-acceptance chain: capture idea, write PRD, generate a task graph, schedule batches, orchestrate agents, then present the result for user acceptance. Acceptance archives the finished loop under `work/archive/iterations/` and resets active `work/` for the next loop. Both plugins are listed in the marketplace catalogs.

`plugins/prd-to-kanban/SKILL.md` is the source of truth for prd-to-kanban behavior. Keep root `SKILL.md` and `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md` as thin entries.

**Note on skill naming:** the standalone `prd-to-kanban` plugin is for users who only need PRD-to-Kanban planning. PraxisKit v3 uses `prd-to-task-graph` and related typed transforms for the full recursive loop.
