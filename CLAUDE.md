# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repo packages two Claude Code / Codex plugins for the marketplace:

1. **prd-to-kanban** (standalone) — converts PRDs/requirements into `work/kanban.md` and `work/SUBAGENT.md`. Planning-only; does not execute tasks.
2. **praxiskit** (full pipeline) — intent-to-acceptance chain: `seed-to-idea → idea-to-prd → prd-to-kanban → kanban-to-agents → build-to-review`.

Install one or the other, not both (PraxisKit bundles prd-to-kanban).

## File Structure

```
SKILL.md                                           # Standalone prd-to-kanban skill entry (thin wrapper)
plugins/prd-to-kanban/SKILL.md                    # prd-to-kanban source of truth
plugins/prd-to-kanban/skills/prd-to-kanban/       # Plugin-packaged entry (thin wrapper)
plugins/prd-to-kanban/.claude-plugin/             # Claude Code plugin manifest
plugins/prd-to-kanban/.codex-plugin/              # Codex plugin manifest
plugins/praxiskit/skills/seed-to-idea/            # PraxisKit step 1
plugins/praxiskit/skills/idea-to-prd/             # PraxisKit step 2
plugins/praxiskit/skills/prd-to-kanban/           # PraxisKit step 3 (thin wrapper)
plugins/praxiskit/skills/kanban-to-agents/        # PraxisKit step 4
plugins/praxiskit/skills/build-to-review/         # PraxisKit step 5
plugins/praxiskit/.claude-plugin/                 # PraxisKit Claude Code plugin manifest
plugins/praxiskit/.codex-plugin/                  # PraxisKit Codex plugin manifest
.claude-plugin/marketplace.json                   # Claude Code marketplace catalog
.agents/plugins/marketplace.json                  # Codex marketplace catalog
examples/                                         # Sample PRD input and kanban output
CLAUDE.md                                          # This file
AGENTS.md                                          # Repo context for Codex
README.md                                          # User-facing install/usage docs
CHANGELOG.md                                       # Version history
```

## Modifying Skills

- **prd-to-kanban behavior:** edit `plugins/prd-to-kanban/SKILL.md` — the single source of truth. Keep root `SKILL.md` and `plugins/prd-to-kanban/skills/prd-to-kanban/SKILL.md` as thin wrappers.
- **PraxisKit steps:** edit the respective `plugins/praxiskit/skills/<step>/SKILL.md` directly. Each step is self-contained.
- **Plugin manifests:** `interface` block in `.claude-plugin/plugin.json` controls marketplace presentation.
- Test by running the skill against `examples/sample-prd.md` and inspecting generated `work/kanban.md` and `work/SUBAGENT.md`.

## Key Design Constraints

- **Two output files**: `kanban.md` (dynamic status tracker) + `SUBAGENT.md` (static shared context, under 80 lines). Orchestrator passes `SUBAGENT.md` to every subagent — subagents read it themselves.
- **Agents edit, humans read**: The board is a living document updated by orchestrating agents during execution
- **Topological layering drives parallelism**: Same layer + disjoint write scopes = safe to parallelize
- **Frozen interfaces first**: Schema/contract tasks are always Wave 0
