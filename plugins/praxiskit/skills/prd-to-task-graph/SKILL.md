---
name: prd-to-task-graph
description: "Convert work/PRD.md into work/task-graph.md and work/SUBAGENT.md for dependency-aware multi-agent implementation. Planning only; do not implement tasks. Replaces v2-M prd-to-kanban."
---

# PRD to Task Graph

Position in PraxisKit:

```text
work/PRD.md -> prd-to-task-graph -> work/task-graph.md + work/SUBAGENT.md -> task-graph-to-batch
```

## Contract

**Inputs:** `prd` (`work/PRD.md`)
**Output:** `task_graph` (`work/task-graph.md` + `work/SUBAGENT.md`)
**Schema:** `schemas/task-graph.schema.md` (v3.0)
**Preconditions:**
- `work/PRD.md` exists and follows `schemas/prd.schema.md`
- All blocking Open Questions in PRD are resolved
- Every FR has non-empty acceptance criteria
- At least 1 milestone is defined
**Postconditions:**
- Every task has ID, acceptance criteria copied verbatim from PRD, write scope, dependencies
- Tasks are organized into waves; Wave 0 holds frozen contracts/schemas
- Critical path is identified
- `work/SUBAGENT.md` is under 80 lines, lists frozen contracts, and distinguishes spawned-worker write scopes from orchestrator bookkeeping files
**Clarification gate:** does NOT fire. This is a pre-flight-validated transform. If preconditions fail, refuse with remediation routing back to `idea-to-prd`.
**Side effects:**
- Writes `work/task-graph.md` and `work/SUBAGENT.md`
- Updates `work/praxiskit-context.md` with task-graph path, current milestone, and open blockers
**Stop boundary:** Does NOT execute tasks, spawn agents, or modify project source code. Hands off to `task-graph-to-batch`.

## Pre-flight: Blocking Check

Before decomposing any tasks, load `schemas/task-graph.schema.md` and run these checks. If any check fails, **stop immediately** — do not write task-graph.md.

### Check 1: No Blocking Open Questions

Scan `work/PRD.md` → `## Open Questions` table. Find rows where `Blocks` = `implementation` or `release` AND the question is unresolved.

If any exist, output:

```
Cannot generate task graph: the following blocking Open Questions must be resolved first.

Blocking questions:
1. [{Question}] -- blocks: {Blocks value}
2. ...

Resolution options:
- Re-run `idea-to-prd` to answer these questions
- Or answer them directly and update work/PRD.md, then retry
```

Stop. Do not write task-graph.md.

### Check 2: All FRs Have Acceptance Criteria

Scan `## Functional Requirements` table. Find FR rows where `Acceptance Criteria` is empty or vague (contains "TBD", "TODO", or is blank).

If any exist, output:

```
Cannot generate task graph: these FRs have missing acceptance criteria:
- {FR_ID}: {Requirement}
...

Resolution options:
- Re-run `idea-to-prd` — it will gate on missing acceptance criteria and collect them from you
- Or add Given/When/Then criteria directly to these rows in work/PRD.md, then retry
```

### Check 3: At Least 1 Milestone

Check that `## Milestones` has at least one row. If not, output:

```
Cannot generate task graph: work/PRD.md has no milestones defined.
Add at least one milestone to ## Milestones, then retry.
```

## Main Workflow

After all pre-flight checks pass, decompose the PRD into a dependency-aware task graph:

1. **Load `schemas/task-graph.schema.md`** for task field rules.
2. **Map FRs to tasks.** Each FR becomes one or more tasks. Copy acceptance criteria verbatim from the FR row — do not rewrite them.
3. **Assign waves.** Wave 0 = frozen contracts and schema definitions. Wave 1+ = implementation tasks ordered by dependency.
4. **Compute dependencies.** A task is unblocked only when all tasks it depends on are complete. Mark dependencies as `[T{wave}.{n}]` references.
5. **Identify parallelism windows.** Tasks in the same wave with disjoint write scopes can run in parallel. Note safe parallel pairs.
6. **Write `work/task-graph.md`** using this format per task row:
   ```
   | T{wave}.{n} | {title} | [ ] | {acceptance criteria} | {write scope} | [T{dep},...] |
   ```
7. **Write `work/SUBAGENT.md`** (under 90 lines): project summary, stack, frozen contracts, write scopes per task, context budget rules, subagent reporting convention (`RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`), and the worker/orchestrator boundary below.
8. **Update `work/praxiskit-context.md`** with task-graph path, current milestone, and open blockers.
9. **Report** the generated paths and wait unless the user invokes `task-graph-to-batch`.

## `work/SUBAGENT.md` Boundary Text

Every generated `work/SUBAGENT.md` must include this rule:

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
