---
name: task-graph-to-batch
description: "Select unblocked tasks from work/task-graph.md and write a work/execution-batch-{n}.md artifact for the next wave. Planning only; does NOT execute tasks. Hand off to batch-to-build for execution."
---

# Task Graph to Batch

Pick the next executable batch from a task graph. This is half of the v2-M `kanban-to-agents` skill — the scheduling half. Execution is now in `batch-to-build`.

```text
work/task-graph.md -> task-graph-to-batch -> work/execution-batch-{n}.md -> batch-to-build
```

## Contract

**Inputs:** `task_graph` (`work/task-graph.md` + `work/SUBAGENT.md`)
**Output:** `execution_batch` (`work/execution-batch-{n}.md`)
**Schema:** `schemas/execution-batch.schema.md` (v3.0)
**Preconditions:**
- `work/task-graph.md` exists and follows `schemas/task-graph.schema.md`
- `work/SUBAGENT.md` exists
- Project baseline has been checked. If it fails, create only a baseline-repair batch for `T0.0 | Restore build baseline`
- At least one task with status `[ ]` and all dependencies `[x]`
**Postconditions:**
- `work/execution-batch-{n}.md` lists selected tasks with parallel groups identified
- Disjoint write scopes within each parallel group
- Authorization defaults to `dry-run` unless the user explicitly authorizes implementation with a narrow execution phrase
**Clarification gate:** does NOT fire. Pre-flight-validated.
**Side effects:**
- Writes `work/execution-batch-{n}.md`
- Reads but does NOT modify `work/task-graph.md`
**Stop boundary:** Does NOT execute tasks, spawn agents, or modify project source code. Hands off to `batch-to-build`.

## Inputs

Require these unless the user provides equivalent content:

- `work/task-graph.md` — task list with statuses and dependencies
- `work/SUBAGENT.md` — frozen contracts and write scopes
- `work/praxiskit-context.md` — optional cross-skill index

If either required file is missing, stop and recommend invoking `prd-to-task-graph` (or `seed-to-task-graph` for light recipe).

## Preflight

Before selecting tasks, establish baseline health:

- Read project scripts from `package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`, or README
- Identify the narrowest test command and build/typecheck command
- Run baseline checks when they are local and reasonable
- Record failures as pre-existing blockers. If a baseline failure blocks release, write a baseline-repair batch with `T0.0 | Restore build baseline` as the only sequential task, `baseline.status = fail`, and `baseline_repair = true`
- If no baseline command exists, write `baseline.status = unavailable`, keep authorization in dry-run mode, and note that execution is blocked until a baseline command is added and rechecked to `pass`
- Do not attribute baseline failures to the current wave

## Selection Rules

1. Parse task status: `[ ]` pending, `[/]` in progress, `[x]` done, `[!]` blocked.
2. A task is unblocked only when all dependencies in `[T...]` are `[x]`.
3. Prefer the earliest dependency layer and wave.
4. Identify critical-path tasks; mark them sequential if they have shared write scope with other selected tasks.
5. Group tasks with disjoint write scopes into parallel groups. Never put two tasks into the same parallel group if their write scopes overlap or if either touches a frozen contract.
6. Limit `max_parallel` to 3 unless the user requests more.

## Authorization

The batch artifact is always written, but with `authorization: dry-run` by default.

Only these narrow phrases authorize execution:
- "execute this batch"
- "implement this wave"
- "modify code for this batch"
- "run agents for this batch"
- "delegate implementation for this batch"

If the user uses one of those exact phrases in the current turn:
- Set `authorization: execute` in the batch artifact
- Record the authorization timestamp

Otherwise, the batch is dry-run only. Words like "advance", "continue", "next", or "proceed" are ambiguous and MUST NOT set `authorization: execute` by themselves.

## Workflow

1. Check repo status and run preflight.
2. Read task-graph.md and parse statuses + dependencies.
3. Select unblocked tasks for the next wave.
4. Compute parallel groups and sequential tasks.
5. Write `work/execution-batch-{n}.md` per `schemas/execution-batch.schema.md`, including:
   - task graph fingerprint (mtime or hash)
   - selected task dependencies
   - selected task status at batch generation time
6. Update `work/praxiskit-context.md` with the batch path.
7. Report the generated path. State that `batch-to-build` is the next step (with execution requiring user authorization).
