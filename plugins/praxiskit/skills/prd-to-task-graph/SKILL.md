---
name: prd-to-task-graph
description: "Create work/task-graph.md and work/SUBAGENT.md from work/PRD.md. Planning only."
---

# PRD to Task Graph

Position in PraxisKit:

```text
work/PRD.md -> prd-to-task-graph -> work/task-graph.md + work/SUBAGENT.md -> task-graph-to-batch
```

## Contract

**Inputs:** `prd` (`work/PRD.md`)
**Output:** `task_graph` (`work/task-graph.md` + `work/SUBAGENT.md`)
**Schema:** bundled `schemas/task-graph.schema.md` (v3.0)
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

## Bundled Resources

`schemas/`, `templates/`, and `references/` paths refer to files bundled with the PraxisKit plugin, relative to the plugin root. Do not create or copy those resource files into the project unless the user explicitly asks. Project writes from this transform are `work/PRD.md` when adopting an existing PRD, `work/task-graph.md`, `work/SUBAGENT.md`, and `work/praxiskit-context.md`.

## PRD Adoption

If `work/PRD.md` is missing but exactly one obvious project PRD exists, such as root `PRD.md`, copy it to `work/PRD.md`, record `PRD source: root PRD.md` in `work/praxiskit-context.md`, and continue the normal pre-flight. This is a planning artifact adoption, not a source change.

If multiple candidate PRDs exist, ask with host-native choice input. If no candidate exists, stop and route to `seed-to-idea`, `idea-to-prd`, or the `from-prd` recipe setup.

## Pre-flight: Blocking Check

Before decomposing any tasks, load bundled `schemas/task-graph.schema.md` and run these checks. If any check fails, **stop immediately** — do not write task-graph.md.

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

1. **Load bundled `schemas/task-graph.schema.md`** for task field rules.
2. **Map FRs to tasks.** Each FR becomes one or more tasks. Copy acceptance criteria verbatim from the FR row — do not rewrite them.
3. **Assign waves.** Wave 0 = frozen contracts and schema definitions. Wave 1+ = implementation tasks ordered by dependency.
4. **Compute dependencies.** A task is unblocked only when all tasks it depends on are complete. Mark dependencies as `[T{wave}.{n}]` references.
5. **Identify parallelism windows.** Tasks in the same wave with disjoint write scopes can run in parallel. Note safe parallel pairs.
6. **Mark shared integration scope.** If multiple tasks may need shared files such as package exports, CLI registries, route indexes, or generated registries, list those files as orchestrator-owned shared integration scope instead of assigning them to one worker.
7. **Write `work/task-graph.md`** using this format per task row:
   ```
   | T{wave}.{n} | {title} | [ ] | {acceptance criteria} | {write scope} | [T{dep},...] |
   ```
8. **Write `work/SUBAGENT.md`** (under 90 lines): project summary, stack, frozen contracts, write scopes per task, shared integration scope, context budget rules, subagent reporting convention (`RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`), and the worker/orchestrator boundary below.
9. **Update `work/praxiskit-context.md`** with task-graph path, PRD source, current milestone, open blockers, and next skill.
10. **Report** the generated paths and present a discuss-style continuation prompt when the user is present:
   - `create_first_batch`: run `task-graph-to-batch` now
   - `stop_here`: stop with resume artifacts

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
