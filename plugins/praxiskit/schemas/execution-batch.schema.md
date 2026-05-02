---
schema_name: execution-batch
schema_version: "3.0"
used_by: task-graph-to-batch, batch-to-build
input_file: work/task-graph.md
output_file: work/execution-batch-{n}.md
---

# Execution Batch Schema v3.0

An `execution_batch` is a snapshot of unblocked `task_graph` work selected for one implementation wave. It is the execution contract handed to `batch-to-build`.

Template: `templates/execution-batch.md`.

## Required Fields

| Field | State | Notes |
|---|---|---|
| batch_id | inferable | Sequential integer |
| source_task_graph | filled-by-user (file path) | Usually `work/task-graph.md` |
| task_graph_fingerprint | inferable | Timestamp or hash at batch generation |
| baseline | filled-by-user (run output) | Preflight build/test command results |
| selected_tasks | inferable | IDs, titles, acceptance criteria, write scopes, dependencies, status |
| parallel_groups | inferable | Disjoint-write groups safe to run together |
| sequential_tasks | inferable | Tasks that must run alone |
| execution_mode | inferable | `subagent-driven` if any parallel group has 2+ tasks; otherwise `orchestrator-single-task` or `sequential` |
| max_parallel | filled-by-user or default | Default 3 |
| authorization | filled-by-user | `dry-run` or `execute` with scoped decision/chat approval |

## Validation Rules

- `work/task-graph.md` exists.
- Baseline is checked. If it fails, create only `T0.0 | Restore build baseline` with `baseline_repair = true`. If unavailable, keep dry-run.
- At least one selected task is `[ ]` and all dependencies are `[x]`.
- Parallel groups have disjoint write scopes.
- If any parallel group has 2+ tasks, `execution_mode = subagent-driven` and the batch lists one subagent dispatch per task.

## Authorization Rules

Execution can be authorized only by:

- host-native execution decision scoped to this exact batch
- direct yes/no chat confirmation scoped to this exact batch
- `batch-to-build` dry-run upgrade after freshness checks pass

User-facing text must not ask the user to type magic phrases or list phrase options. If no structured input is available, ask: "Execute Batch {n} now? yes/no". Ambiguous wording such as "advance", "continue", "next", or "proceed" does not authorize execution by itself.

If `execution_mode = subagent-driven`, authorization covers only subagent-driven execution. The orchestrator must not implement parallel-group tasks directly.

## Dry-Run Upgrade Checks

`batch-to-build` may upgrade a dry-run batch only when:

- the user explicitly authorizes execution in the current session by decision UI or direct yes/no chat confirmation
- the task graph fingerprint is unchanged, or selected task rows still match exactly
- selected tasks are still `[ ]` and unblocked
- baseline is `pass`, or this is an authorized baseline-repair batch

If any check fails, refuse and route back to `task-graph-to-batch`.

## Baseline Repair

Baseline repair is the only executable exception to `baseline.status = pass`.

- exactly one selected task: `T0.0 | Restore build baseline`
- `baseline.status = fail`
- `baseline_repair = true`
- no parallel groups

## Changelog

- v3.1 (2026-05-02): Compacted runtime schema and moved output skeleton to `templates/execution-batch.md`.
- v3.0 (2026-04-30): Initial v3 schema.
