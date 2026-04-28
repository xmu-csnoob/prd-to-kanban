---
name: prd-to-kanban
description: "Convert a PRD, design doc, feature requirements, or PraxisKit work/PRD.md into work/kanban.md and work/SUBAGENT.md for dependency-aware multi-agent implementation planning. Planning only; do not implement tasks."
---

# PRD to Kanban

This is the PraxisKit-packaged entry point for `prd-to-kanban`.

Invoke the `prd-to-kanban` skill and follow its instructions exactly. That skill is the source of truth for output format, task granularity, acceptance criteria, dependency layering, critical path, and subagent handoff rules.

Position in PraxisKit:

```text
work/PRD.md -> prd-to-kanban -> work/kanban.md + work/SUBAGENT.md -> kanban-to-agents
```

If the shared source is unavailable, use this fallback:

- Convert the PRD or requirements into `<project>/work/kanban.md` and `<project>/work/SUBAGENT.md`.
- Plan only. Do not implement tasks.
- Use verifiable acceptance criteria for every task.
- Put frozen contract/schema work in Wave 0.
- Compute dependency layers, parallelism windows, and the critical path.
- Keep `SUBAGENT.md` under 80 lines with project summary, stack, frozen contracts, write scopes, and subagent reporting conventions.
- Report the generated paths and wait unless the user asks for execution with `kanban-to-agents`.
