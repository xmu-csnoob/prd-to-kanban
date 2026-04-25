# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Claude Code skill that converts PRDs/requirements into a living Kanban board (`kanban.md`). This is a planning-only skill — it decomposes work but does not execute it. Execution is handed off to `subagent-driven-development`.

Workflow chain: PRD → **prd-to-kanban** → `kanban.md` → subagent-driven-development → implemented code.

Sub-commands:
- `install [--project]` — writes auto-trigger rules to CLAUDE.md so the skill fires automatically on PRD/requirements detection

## File Structure

```
SKILL.md           # Full skill definition (trigger rules, format spec, execution rules, DoD)
CLAUDE.md          # This file — repo context for Claude Code
.claude/           # Local settings (permissions, MCP servers)
```

`SKILL.md` is the single source of truth for the skill's behavior. Everything — output format, task granularity rules, acceptance criteria patterns, update protocol, regeneration logic — lives there.

## How Skills Work

Claude Code skills are loaded from `~/.claude/skills/<name>/SKILL.md`. The `name` in frontmatter must match the directory name. Skills are invoked via the Skill tool (equivalent to `/skill-name`).

## Modifying This Skill

- All skill logic is in `SKILL.md` — edit that file directly
- The skill defines its own trigger conditions (when Claude should invoke it automatically)
- Settings in `.claude/settings.local.json` grant permissions needed during kanban generation (e.g., `open` for viewing the output file)
- Test changes by running the skill against a sample PRD and inspecting the generated `kanban.md`

## Key Design Constraints

- **Single output file**: Everything goes into `kanban.md` — no split formats, no HTML
- **Agents edit, humans read**: The board is a living document updated by orchestrating agents during execution
- **Topological layering drives parallelism**: Same layer + disjoint write scopes = safe to parallelize
- **Frozen interfaces first**: Schema/contract tasks are always Wave 0
