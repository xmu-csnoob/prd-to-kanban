---
name: prd-to-kanban
description: "Convert a PRD, design doc, feature requirements, or PraxisKit work/PRD.md into work/kanban.md and work/SUBAGENT.md for dependency-aware multi-agent implementation planning. Planning only; do not implement tasks."
---

# PRD to Kanban

Position in PraxisKit:

```text
work/PRD.md -> prd-to-kanban -> work/kanban.md + work/SUBAGENT.md -> kanban-to-agents
```

## Pre-flight

### Blocking Check

Before decomposing any tasks, load `schemas/kanban.schema.md` and run these checks. If any check fails, **stop immediately** -- do not write kanban.md.

### Check 1: No Blocking Open Questions

Scan `work/PRD.md` -> `## Open Questions` table. Find rows where `Blocks` = `implementation` or `release` AND the question is unresolved.

If any exist, output:

```
Cannot generate kanban: the following blocking Open Questions must be resolved first.

Blocking questions:
1. [{Question}] -- blocks: {Blocks value}
2. ...

Resolution options:
- Re-run `idea-to-prd` to answer these questions
- Or answer them directly and update work/PRD.md, then retry
```

Stop. Do not write kanban.md.

### Check 2: All FRs Have Acceptance Criteria

Scan `## Functional Requirements` table. Find FR rows where `Acceptance Criteria` is empty or vague (contains "TBD", "TODO", or is blank).

If any exist, output:

```
Cannot generate kanban: these FRs have missing acceptance criteria:
- {FR_ID}: {Requirement}
...

Resolution options:
- Re-run `idea-to-prd` — it will gate on missing acceptance criteria and collect them from you
- Or add Given/When/Then criteria directly to these rows in work/PRD.md, then retry
```

### Check 3: At Least 1 Milestone

Check that `## Milestones` has at least one row. If not, output:

```
Cannot generate kanban: work/PRD.md has no milestones defined.
Add at least one milestone to ## Milestones, then retry.
```

## Main Workflow

After all pre-flight checks pass, decompose the PRD into a dependency-aware task board:

1. **Load `schemas/kanban.schema.md`** for task field rules.
2. **Map FRs to tasks.** Each FR becomes one or more tasks. Copy acceptance criteria verbatim from the FR row — do not rewrite them.
3. **Assign waves.** Wave 0 = frozen contracts and schema definitions. Wave 1+ = implementation tasks ordered by dependency.
4. **Compute dependencies.** A task is unblocked only when all tasks it depends on are complete. Mark dependencies as `[T{wave}.{n}]` references.
5. **Identify parallelism windows.** Tasks in the same wave with disjoint write scopes can run in parallel. Note safe parallel pairs.
6. **Write `work/kanban.md`** using this format per task row:
   ```
   | T{wave}.{n} | {title} | [ ] | {acceptance criteria} | {write scope} | [T{dep},...] |
   ```
7. **Write `work/SUBAGENT.md`** (under 80 lines): project summary, stack, frozen contracts, write scopes per task, subagent reporting convention (`RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`).
8. **Update `work/praxiskit-context.md`** with kanban path, current milestone, and open blockers.
9. **Report** the generated paths and wait unless the user asks for execution with `kanban-to-agents`.
