---
name: decompose-task-graph
description: "Shared decomposition workflow used by prd-to-task-graph and seed-to-task-graph after their pre-flight/gate completes."
type: reference
---

# Decompose Task Graph

Shared workflow for both task-graph producers. Invoke after pre-flight (PRD path) or gate (seed path) has resolved all blockers and you have either:
- a validated `work/PRD.md` (PRD path), or
- a goal + acceptance signals collected from seed/gate (seed path).

## Steps

1. **Identify minimum viable tasks.** PRD path: each FR becomes one or more tasks. Seed path: derive from goal + acceptance signals.
2. **Copy acceptance criteria verbatim.** PRD path: copy from FR row exactly — do not rewrite. Seed path: copy / quote / split the seed or gate answer; never add new acceptance meaning.
3. **Assign waves.** Wave 0 = frozen contracts and schema definitions. Wave 1+ = implementation tasks ordered by dependency.
4. **Compute dependencies.** A task is unblocked only when all dependencies are `[x]`. Mark as `[T{wave}.{n}]` references.
5. **Identify parallelism windows.** Same-wave tasks with disjoint write scopes can run in parallel. Note safe parallel pairs.
6. **Mark shared integration scope.** Files touched by multiple tasks (package exports, CLI registries, route indexes, generated registries) → orchestrator-owned shared integration scope, not assigned to one worker.
7. **Write `work/task-graph.md`** per `schemas/task-graph.schema.md` output format. Each row:
   ```
   | T{wave}.{n} | {title} | [ ] | {acceptance criteria} | {write scope} | [T{dep},...] |
   ```
8. **Write `work/SUBAGENT.md`** (under 90 lines):
   - project / goal summary
   - stack
   - frozen contracts (PRD path) or none (seed path)
   - write scopes per task
   - shared integration scope (if any)
   - subagent reporting convention: `RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`
   - boundary block from `references/shared-boundaries.md` (insert verbatim)
9. **Update `work/praxiskit-context.md`** with task-graph path, source recipe, current milestone, open blockers, next skill (`task-graph-to-batch`).
10. **Report** the generated paths and present a discuss-style continuation prompt when the user is present:
    - `create_first_batch`: run `task-graph-to-batch` now
    - `stop_here`: stop with resume artifacts

## Notes

- Workers never read this file — it is orchestrator-side. Workers only read `work/SUBAGENT.md` + their batch entry.
- The bilingual format requirement was removed in v3.2; titles and acceptance criteria match the source language (PRD or seed).
