---
schema_name: execution-batch
schema_version: "3.0"
used_by: task-graph-to-batch, batch-to-build
input_file: work/task-graph.md
output_file: work/execution-batch-{n}.md
---

# Execution Batch Schema v3.0

An `execution_batch` artifact is a snapshot of unblocked tasks selected from a `task_graph` for a single execution wave. It is the contract handed to `batch-to-build` for actual implementation.

## Required Fields

| Field | State | Notes |
|---|---|---|
| batch_id | inferable | Sequential integer; matches the wave being executed |
| source_task_graph | filled-by-user (file path) | Path to `work/task-graph.md` |
| task_graph_fingerprint | inferable | Timestamp or hash of `work/task-graph.md` at batch generation time |
| baseline | filled-by-user (run output) | Result of preflight build/test commands |
| selected_tasks | inferable | Snapshot of task IDs, titles, acceptance criteria, write scopes, dependencies, and status from the task graph |
| parallel_groups | inferable | Sub-groups within the batch that have disjoint write scopes (safe to parallelize) |
| sequential_tasks | inferable | Tasks that must run alone (e.g., critical-path or shared-write-scope) |
| max_parallel | filled-by-user or default | Number of parallel agents allowed; default 3 |
| authorization | filled-by-user | `dry-run` (default) or `execute` (requires explicit user approval) |

## Pre-flight Conditions

Before producing a batch, `task-graph-to-batch` must verify:

| Condition | Check | On Failure |
|---|---|---|
| task-graph.md exists | File present | Refuse; route to `prd-to-task-graph` |
| Baseline checked | Run narrowest test/build command from project scripts | If baseline fails, create a baseline-repair batch containing only `T0.0 | Restore build baseline`; mark `baseline.status = fail` and `baseline_repair = true`. If no baseline command exists, mark `baseline.status = unavailable` and keep the batch dry-run only. |
| At least 1 unblocked task exists | Scan task_graph for tasks with status `[ ]` AND all dependencies `[x]` | Refuse; output "all remaining work is blocked" with the blocker list |
| No write-scope conflicts in parallel groups | Cross-check `write_scope` per task | Reduce parallel groups until disjoint |

## Authorization Rule

`task-graph-to-batch` writes the batch artifact regardless. `batch-to-build` refuses to execute unless the user has explicitly authorized implementation in the current session.

Execution can be authorized in either of two auditable ways:

1. `task-graph-to-batch` writes the artifact with `authorization: execute`, or
2. `batch-to-build` upgrades an existing dry-run artifact after fresh explicit authorization, but only after validating that the batch is still current.

Ambiguous wording such as "advance", "continue", "next", or "proceed" does not authorize execution.

## Dry-Run Upgrade Rule

`batch-to-build` may upgrade an existing dry-run batch to execute only when all checks pass:

- User explicitly says an execution phrase such as "execute this batch", "implement this wave", "modify code for this batch", "run agents for this batch", or "delegate implementation for this batch"
- `work/task-graph.md` fingerprint is unchanged, or the selected task rows still match exactly for task IDs, acceptance criteria, dependencies, write scopes, and status
- Selected tasks are still `[ ]` and still unblocked
- Baseline has been rechecked and is `pass`, or this is an explicitly authorized baseline-repair batch

If any check fails, refuse and route back to `task-graph-to-batch`.

`baseline.status = unavailable` is allowed for dry-run planning only. It does not authorize source changes. Before execution, the user or agent must either add a baseline command and recheck to `pass`, or create an explicit baseline-repair batch if the project baseline is broken.

## Baseline Repair Rule

A normal execution batch requires baseline checks to pass before `batch-to-build` may run. `baseline.status = unavailable` means no baseline command was found; it may document a dry-run batch but must not be executed. The only exception to `pass` is a baseline-repair batch:

- It contains exactly one selected task: `T0.0 | Restore build baseline`
- `baseline.status` is `fail`
- `baseline_repair` is `true`
- It has no parallel groups

`batch-to-build` may execute this batch after explicit authorization. Its validation target is restoring the baseline to pass.

## Output Format (work/execution-batch-{n}.md)

```markdown
# Execution Batch {n}

## Source
- Task graph: `work/task-graph.md`
- Task graph fingerprint: `{mtime/hash}`
- Generated: {date}

## Baseline
- Test command: `{cmd}` → {result}
- Build command: `{cmd}` → {result}
- Status: pass | fail | unavailable
- Baseline repair: true | false

## Selected Tasks
| ID | Title | Acceptance Criteria | Write Scope | Dependencies | Status At Batch | Parallel Group |
|----|-------|---------------------|-------------|--------------|-----------------|----------------|
| T1.1 | ... | Given..., when..., then... | `path/` | [T0.1] | [ ] | A |
| T1.2 | ... | ... | `path/` | [T0.1] | [ ] | A |
| T1.3 | ... | ... | `path/` | [T1.1] | [ ] | sequential |

## Parallel Groups
- Group A: [T1.1, T1.2] (disjoint write scopes)
- Sequential: [T1.3]

## Authorization
- Mode: dry-run | execute
- Approved by user: yes / no
- Approval timestamp: {date or "not approved"}
- Upgraded by batch-to-build: yes / no

## Handoff
Next: `batch-to-build`. It executes only when `Mode: execute`, or when the user explicitly authorizes upgrading this dry-run batch and freshness checks pass.
```

## Changelog

- v3.0 (2026-04-30): Initial v3 schema. Splits scheduling concerns (this artifact) from execution (build artifact).
