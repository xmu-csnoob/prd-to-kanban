# PRD to Kanban

A Claude Code skill that converts PRDs/requirements into a living Kanban board (`kanban.md`) — readable by humans, editable by agents.

## Workflow

```
PRD → prd-to-kanban → kanban.md → subagent-driven-development → implemented code
```

This is a **planning-only** skill — it decomposes work but does not execute it. After generating `kanban.md`, execution is handed off to `subagent-driven-development`.

## Features

- **Topological task layering** — same layer = no transitive dependencies = safe to parallelize
- **Critical path identification** — longest chain through dependency graph, marked with visual indicators
- **Computed fields** — progress %, dependency layers, parallelism windows auto-refreshed from task statuses
- **Multi-agent update protocol** — orchestrator-owned board, subagent-safe status updates
- **Definition of Done** — verifiable acceptance criteria by task type (schema, API, UI, feature, test, integration)
- **Auto-install** — `/prd-to-kanban install` writes trigger rules to CLAUDE.md for automatic invocation

## Install

```bash
# Clone into your skills directory
git clone https://github.com/xmu-csnoob/prd-to-kanban.git ~/.claude/skills/prd-to-kanban

# Auto-configure trigger rules
# (run in Claude Code)
/prd-to-kanban install
```

## Usage

```
/prd-to-kanban                  # Convert current PRD/requirements
/prd-to-kanban install          # Install auto-trigger rules to ~/.claude/CLAUDE.md
/prd-to-kanban install --project  # Install to project-level ./CLAUDE.md
```

## Output

A single file: `<project>/work/kanban.md`

## License

MIT
