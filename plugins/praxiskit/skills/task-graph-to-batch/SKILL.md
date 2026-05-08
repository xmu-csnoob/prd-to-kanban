---
name: task-graph-to-batch
description: "Select the next unblocked wave and write work/execution-batch-{n}.md. Planning only."
---

# Task Graph to Batch

Select the next executable wave from `work/task-graph.md`. This skill schedules; `batch-to-build` executes.

```text
work/task-graph.md -> task-graph-to-batch -> work/execution-batch-{n}.md -> batch-to-build
```

Batch schema: bundled `schemas/execution-batch.schema.md`. Template: bundled `templates/execution-batch.md`.

## Contract

**Inputs:** `work/task-graph.md`, `work/SUBAGENT.md`, optional `work/praxiskit-context.md`
**Output:** `work/execution-batch-{n}.md`
**Preconditions:** task graph and SUBAGENT exist; at least one `[ ]` task has all dependencies `[x]`; baseline is checked
**Postconditions:** batch lists selected task rows, acceptance criteria, write scopes, dependencies, baseline, validation commands, parallel groups, execution mode, and authorization state
**Stop boundary:** Does not execute, spawn agents, scaffold source, install dependencies, or modify project source.

## Bundled Resources

`schemas/`, `templates/`, and `references/` paths refer to files bundled with the PraxisKit plugin, relative to the plugin root. Do not create or copy those resource files into the project unless the user explicitly asks. Project writes belong under `work/` plus any requested source changes from later execution transforms.

## Selection Rules

- Parse statuses: `[ ]` pending, `[/]` in progress, `[x]` done, `[!]` blocked.
- A task is unblocked only when all dependencies are `[x]`.
- Prefer earliest dependency layer/wave.
- Use lookahead within the same milestone when it reduces churn: a batch may include tasks that become unblocked after earlier selected groups complete, as long as dependencies are represented by sequential groups and every write scope is known.
- Stop lookahead at milestone/review boundaries, blocked tasks, unclear validation, or any task whose write scope depends on implementation decisions from earlier groups.
- Put tasks with overlapping write scopes, frozen-contract writes, or critical-path conflicts in sequential groups.
- Put only disjoint write scopes in the same parallel group.
- Limit `max_parallel` to 3 unless the user requests more.
- If baseline fails, create only a baseline-repair batch for `T0.0 | Restore build baseline`.
- If no baseline command exists because the project has no executable code yet, set `baseline.status = pass` and `Baseline mode: fresh-start`; this first construction batch may be executable after scoped authorization.
- If no baseline command exists for an already-started project, set `baseline.status = unavailable` and keep dry-run.

## Execution Mode

- Any 2+ task parallel group must set `execution_mode: subagent-driven`.
- The batch must list one subagent dispatch expectation per task in every 2+ task parallel group.
- Authorization for a subagent-driven batch covers only subagent-driven execution; the orchestrator must not implement those parallel tasks directly.
- Single-task sequential groups default to orchestrator execution. Mark them as worker-dispatch only when the task is large, risky, explicitly requested, or the batch explains why a worker is required.
- Shared integration files such as package exports, CLI command registries, route indexes, or generated registries should be listed as `shared integration scope` owned by the orchestrator unless one task exclusively owns the file.

## Authorization Bridge

Write the batch as dry-run by default, then collect scoped execution authorization if the user is present.

Use host-native decision/input whenever available.

- Question: "Execute Batch {n} now?"
- Options:
  - `execute_now`: set `Mode: execute`, then hand off to `batch-to-build`
  - `keep_dry_run`: leave dry-run and stop
  - `revise_batch`: leave dry-run and ask what to change
- Prompt must include batch path, task IDs, parallel groups, subagent dispatch expectation, and validation commands.
- If the batch has any parallel group, the prompt must say execution will be subagent-driven.
- This prompt is the discuss-style interaction surface: prefer choice UI so the user can answer with keyboard selection instead of typing a slash command or phrase.

If no structured input is available, ask one direct chat question: "Execute Batch {n} now? yes/no". Treat only a clear yes for this exact batch as authorization.

Do not instruct the user to type specific command phrases. Ambiguous words such as "advance", "continue", "next", or "proceed" do not authorize execution by themselves.

## Workflow

1. Check repo status and baseline commands.
2. Read task graph and SUBAGENT.
3. Select unblocked tasks using the rules above.
4. Compute parallel/sequential groups and execution mode.
5. Write `work/execution-batch-{n}.md` from `templates/execution-batch.md`, including:
   - task graph fingerprint
   - exact selected task rows
   - exact acceptance criteria
   - dependencies and status at generation time
   - owned write scopes and relevant frozen contracts
   - validation commands
   - execution mode and subagent dispatch expectations
   - `Baseline mode: fresh-start` when applicable
   - `shared integration scope` when shared files may need orchestrator reconciliation
6. If execution is authorized by decision UI or direct yes/no chat confirmation, update authorization fields:
   - `Mode: execute`
   - `Approved by user: yes`
   - `Authorization source: decision-ui | chat-confirmation`
   - current timestamp
7. Update `work/praxiskit-context.md` with batch path and authorization state.
8. Report the batch path and whether execution was authorized now or remains dry-run.

## Routing

- Authorized now -> hand off to `batch-to-build`.
- Dry-run -> stop and wait for explicit authorization.
- Batch needs revision -> keep dry-run and ask for the requested change.

## Planning Guardrail

Do not create source directories, scaffold files, install packages, run formatters, modify source, or spawn implementation agents. Those actions belong only to `batch-to-build` after authorization.
