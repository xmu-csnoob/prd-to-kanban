# Changelog

All notable changes are documented here.

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
