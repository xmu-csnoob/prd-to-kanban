# Changelog

All notable changes are documented here.

## [0.3.0] — PraxisKit typed transforms

Replaces the v2 integrated-skill direction with typed A-to-B transforms and recipe composition.

- Added v3 transforms: `seed-to-task-graph`, `prd-to-task-graph`, `task-graph-to-batch`, `batch-to-build`, `build-to-review-packet`, and `review-to-acceptance`
- Kept `seed-to-idea` and `idea-to-prd` as typed planning transforms
- Added schemas for seed triage, task graphs, execution batches, review packets, and acceptance decisions
- Added light, standard, heavy, and from-prd recipes
- Removed v2 integrated skills: `shape-to-plan`, `plan-to-review`, and `plan.schema.md`
- Renamed v2 concepts: `kanban` -> `task-graph`, `build-to-review` -> `build-to-review-packet`
- Updated PraxisKit plugin prompts to point at recipes and explicit execution authorization

## [0.2.0] — PraxisKit integrated skills

Adds the two-skill integrated PraxisKit flow on top of the existing atomic skills.

Deprecated by v3.0.

- `shape-to-plan` — seed / idea / PRD → right-sized plan, with Light / Standard / Full mode selection
- `plan-to-review` — plan / Kanban → dry-run by default, execution only with explicit authorization, then review evidence
- `plan.schema.md` — compact `work/plan.md` schema so Light mode does not bypass v2 gates
- Updated PraxisKit plugin prompts and README to recommend the two-step flow while keeping atomic skills available

## [1.0.0] — prd-to-kanban

Initial release of the standalone `prd-to-kanban` skill and plugin.

- Two-output system: `work/kanban.md` (dynamic board) + `work/SUBAGENT.md` (static shared context)
- Dependency-driven parallelism: topological layers, executable batches, critical path
- `install` sub-command for Claude Code trigger rules
- Claude Code plugin (`plugins/prd-to-kanban/.claude-plugin/`)
- Codex plugin (`plugins/prd-to-kanban/.codex-plugin/`)
- Marketplace catalog entries

## [0.1.0] — praxiskit

Initial release of the PraxisKit multi-step intent-to-acceptance pipeline plugin.

- `seed-to-idea` — raw seed → `work/idea.md`
- `idea-to-prd` — idea brief → `work/PRD.md`
- `prd-to-kanban` — PRD → Kanban board (delegates to standalone prd-to-kanban skill)
- `kanban-to-agents` — Kanban plan → coordinated agent execution
- `build-to-review` — implemented build → user acceptance review (`work/review.md`)
- Claude Code plugin (`plugins/praxiskit/.claude-plugin/`)
- Codex plugin (`plugins/praxiskit/.codex-plugin/`)
- Marketplace catalog entries
