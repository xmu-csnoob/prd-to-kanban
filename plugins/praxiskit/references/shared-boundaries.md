---
name: shared-boundaries
description: "Canonical SUBAGENT write-scope boundary block. Insert verbatim into every generated work/SUBAGENT.md."
type: reference
---

# Shared Boundaries

Insert this block verbatim into every generated `work/SUBAGENT.md`:

```markdown
## Context Budget

Workers read this file plus their assigned `work/execution-batch-{n}.md` task entry. They should read only source files needed for their write scope and should not load full PRDs, full task graphs, previous build logs, or unrelated source trees unless the task explicitly requires it.

## Write-Scope Boundary

Spawned workers may modify only their assigned task write scope. They must not update PraxisKit bookkeeping files.

The orchestrator, and only the orchestrator, may update:
- `work/task-graph.md`
- `work/execution-batch-*.md`
- `work/build-log-*.md`
- `work/praxiskit-context.md`
- `work/review.md`
- `work/acceptance.md`
```
