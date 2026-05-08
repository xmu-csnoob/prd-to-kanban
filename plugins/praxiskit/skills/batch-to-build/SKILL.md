---
name: batch-to-build
description: "Execute an authorized batch with subagents, validate, update statuses, and write build log."
---

# Batch to Build

Execute one authorized batch. Scheduling belongs to `task-graph-to-batch`; this skill implements, validates, closes bookkeeping, and stops before acceptance.

```text
work/execution-batch-{n}.md -> batch-to-build -> code + work/build-log-{n}.md + task statuses
```

Build log template: `templates/build-log.md`.

## Contract

**Inputs:** `work/execution-batch-{n}.md` with `Mode: execute`, or dry-run plus fresh scoped authorization that passes upgrade checks
**Output:** modified source, `work/build-log-{n}.md`, updated `work/task-graph.md`, updated `work/praxiskit-context.md`
**Preconditions:** batch exists; baseline is `pass` unless this is a valid baseline-repair batch; authorization is scoped to this batch
**Postconditions:** selected tasks are `[x]` after validation or `[!]` with reason; parallel groups use subagents; closeout records next entry point
**Stop boundary:** Does not make acceptance decisions or extend scope. Hand off to `build-to-review-packet`.

## Must Rules

- Authorization is required in the current session by host-native decision or direct chat confirmation scoped to this batch.
- `baseline.status = failed` or `baseline.status = unavailable` is not executable unless the batch explicitly says `Baseline mode: fresh-start` or this is a baseline-repair batch. Route other cases back to `task-graph-to-batch`.
- Any 2+ task parallel group is `subagent-driven`. The orchestrator must not implement those tasks directly.
- Spawn all subagents for a parallel group before doing implementation work for that group.
- Single-task sequential groups default to orchestrator implementation. Dispatch a worker only when the task is large, risky, explicitly requested, or the batch says a worker is required.
- Workers may modify only assigned write scopes and must not update PraxisKit bookkeeping files.
- The orchestrator owns `work/task-graph.md`, `work/execution-batch-*.md`, `work/build-log-*.md`, `work/praxiskit-context.md`, `work/review.md`, and `work/acceptance.md`.
- After all subagents in a group complete, the orchestrator continues automatically through reconcile, review, validation, final task status updates, build log, and closeout.

## Authorization

1. Read the batch authorization block.
2. If dry-run and no current-turn authorization exists, ask once with host-native input if available:
   - question: "Execute this batch now?"
   - options: `execute_now`, `keep_dry_run`
   - include batch path, task IDs, parallel groups, subagent dispatch expectation, and validation commands
3. If still not authorized, refuse:
   ```text
   Cannot execute: batch is dry-run only.
   Execution requires a yes/no decision for this exact batch.
   ```
4. Never instruct the user to type specific command phrases. Use a host-native decision prompt or one direct yes/no chat question.
5. If upgrading dry-run, verify:
   - task graph fingerprint unchanged, or selected task rows still match exactly
   - selected tasks are still `[ ]` and unblocked
   - baseline rechecked and executable
6. If checks pass, update authorization to `Mode: execute`, `Approved by user: yes`, `Authorization source: decision-ui | chat-confirmation`, current timestamp, and `Upgraded by batch-to-build: yes`.
7. If any parallel group has 2+ tasks and the batch does not say `subagent-driven`, treat that as a contract defect and update/refuse before execution.

## Context Budget

- Treat the execution batch as the canonical input.
- Read `work/task-graph.md` at most once during authorization/upgrade checks; use targeted lookup for statuses.
- Do not read PRD, idea, or prior logs unless the batch lacks required execution detail.
- Do not paste full source, task graph, PRD, or logs into subagent prompts.
- Update `work/task-graph.md` in at most two grouped edits: mark selected tasks `[/]`, then final `[x]` / `[!]`.
- Keep logs concise: file lists, summaries, validation, risks. No full source dumps.
- Keep worker prompts and worker final reports terse. The required `RESULT:` line is enough unless a blocker or risk needs one extra sentence.
- For large builds, start the next batch in a fresh session with only `work/praxiskit-context.md`, the next batch, and `work/SUBAGENT.md`.

## Workflow

1. Authorize as above.
2. Re-run baseline. If it fails and this is not a baseline-repair batch, refuse and route to re-batching.
3. Mark all selected tasks `[/]` in one grouped edit.
4. Execute groups in dependency order:
   - For each 2+ task parallel group, dispatch one subagent per task before integrating results.
   - In Claude Code, use `Task`; in Codex or other hosts, use the platform subagent/delegation tool.
   - If a parallel group requires subagents and no subagent tool exists, stop and report the blocker.
   - Single-task sequential groups should be implemented by the orchestrator unless dispatch is justified by the batch or task risk.
5. Subagent prompt must include only:
   - `work/SUBAGENT.md`
   - current execution batch path
   - exact task row and acceptance criteria
   - dependencies already complete
   - owned write scope
   - "you are not alone; do not revert others; do not edit PraxisKit bookkeeping"
   - report format: `RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`
6. After the last subagent in a group completes, reconcile automatically:
   - collect every `RESULT`
   - inspect changed files and scope compliance with targeted diffs
   - record blockers and any scope violations
7. Run validation once after group integration unless a worker reported a blocking failure.
8. Mark final task statuses `[x]` / `[!]` in one grouped edit. Subagent completion notifications alone do not close tasks.
9. Write `work/build-log-{n}.md` from `templates/build-log.md`.
10. Refresh `work/praxiskit-context.md`.
11. Run closeout and present the continuation prompt below when the user is present.

## Closeout

Before stopping:

- Confirm every selected task is `[x]` or `[!]`.
- Classify leftovers as `next_batch`, `blocked`, `deferred`, or `cleanup`.
- Update only docs touched by the batch or required to explain current state.
- Archive transient scratch/clarify notes under `work/archive/` only after summarizing relevant content in the build log.
- In `work/praxiskit-context.md`, record last batch, validation result, blockers, recommended next skill, and exact fresh-session artifacts.
- If unblocked tasks remain, use host-native decision input as a discuss-style continuation prompt:
  - question: "What should PraxisKit do next?"
  - options:
    - `create_next_batch`: run `task-graph-to-batch` now; this is planning only and will show a separate execution authorization prompt
    - `review_current_build`: run `build-to-review-packet` now
    - `stop_here`: stop with the resume artifacts
- If host-native input is unavailable, ask one direct chat question listing those three options.
- Never interpret silence or a vague "continue" as execution authorization. Choosing `create_next_batch` authorizes only planning the next batch; `batch-to-build` still requires its own scoped execution authorization.

Formal accept/revise/continue decisions belong to `build-to-review-packet` and `review-to-acceptance`.
