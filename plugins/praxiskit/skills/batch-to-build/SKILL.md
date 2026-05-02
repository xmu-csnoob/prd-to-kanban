---
name: batch-to-build
description: "Execute an authorized work/execution-batch-{n}.md by dispatching subagents for parallel groups and produce a build artifact. REQUIRES explicit user authorization to run; otherwise refuses and reports dry-run only."
---

# Batch to Build

Execute an authorized batch and produce code, tests, and run logs. This is the second half of the v2-M `kanban-to-agents` skill — the execution half. Scheduling is in `task-graph-to-batch`.

```text
work/execution-batch-{n}.md (authorized) -> batch-to-build -> code + work/build-log-{n}.md
```

## Contract

**Inputs:** `execution_batch` (`work/execution-batch-{n}.md`) with `authorization: execute`, or a dry-run batch plus fresh explicit execution authorization that passes the upgrade checks below
**Output:** `build` (modified project source + `work/build-log-{n}.md` + updated task statuses in `work/task-graph.md`)
**Schema:** none for the build artifact itself; the run log follows the format below
**Preconditions:**
- Batch artifact exists at the specified path
- Batch authorization is `execute`, or the user explicitly authorizes upgrading this exact dry-run batch in the current session
- User has confirmed authorization in the current session with a narrow execution phrase
- Baseline build/test passes, unless this is a baseline-repair batch (`baseline_repair = true`) whose only selected task is `T0.0 | Restore build baseline`
- `baseline.status = unavailable` is not executable; route back to `task-graph-to-batch` after adding or selecting a baseline command
**Postconditions:**
- Selected tasks are either marked `[x]` (passed validation) or `[!]` (blocked, with reason)
- `work/task-graph.md` reflects updated statuses
- `work/build-log-{n}.md` records what was changed, by whom, and validation results
- Parallel groups respected: no two tasks in the same group write to overlapping paths
- Frozen contracts (per `work/SUBAGENT.md`) are not modified by any spawned worker
- Every multi-task parallel group is dispatched through subagents; the orchestrator does not implement those tasks itself
- Context usage is bounded: no repeated full-file reads of large artifacts, no full-source snapshots in prompts or logs, and task status updates are batched
- Batch closeout is completed: residual work is classified, transient notes are archived or summarized, and the next iteration entry point is explicit
**Clarification gate:** does NOT fire. Authorization gating is the only user interaction.
**Side effects:**
- Modifies project source code
- Spawns subagents for each multi-task parallel group
- Writes `work/build-log-{n}.md`
- Updates `work/task-graph.md` task statuses
- Updates `work/praxiskit-context.md` with last-run validation
- May archive transient clarification or scratch notes after summarizing them in the build log/context. Do not archive canonical artifacts (`work/task-graph.md`, execution batches, build logs, review packets, or acceptance decisions).
**Stop boundary:** Does NOT make acceptance decisions. Does NOT extend scope beyond the batch. Hands off to `build-to-review-packet` for inspection.

## Authorization Check (FIRST STEP)

Before any other action:

1. Read the batch artifact and find the authorization field.
2. If the batch is dry-run and the user did not explicitly authorize execution in the current turn, refuse. Output:
   ```
   Cannot execute: batch is dry-run only.
   To execute this exact batch, explicitly say "execute this batch" or "implement this wave".
   ```
   Stop.
3. If the batch is dry-run and the user explicitly authorized execution, run the dry-run upgrade checks:
   - Re-read `work/task-graph.md`
   - Confirm the task graph fingerprint is unchanged, or compare the selected task rows exactly
   - Confirm selected task IDs, acceptance criteria, dependencies, write scopes, and recorded statuses still match the batch
   - Confirm selected tasks are still `[ ]` and still unblocked
   - Re-check baseline
   - If any check fails, refuse and route back to `task-graph-to-batch`
   - If all checks pass, update the batch artifact's authorization section to `Mode: execute`, `Approved by user: yes`, current timestamp, and `Upgraded by batch-to-build: yes`
4. If the batch is already execute and the user has not confirmed in this session: ask once. If user says yes, proceed. Otherwise refuse.
5. If user has confirmed: proceed.

## Context Budget Rules

Execution should scale by keeping each agent's context narrow:

- Treat `work/execution-batch-{n}.md` as the canonical execution input. It should contain the selected task rows, acceptance criteria, dependencies, write scopes, baseline, and validation commands needed for this run.
- Read `work/task-graph.md` at most once during authorization/upgrade checks. If only statuses are needed, use targeted lookup of selected task IDs instead of repeatedly reading the full file.
- Do not read `work/PRD.md`, `work/idea.md`, or prior build logs during execution unless the batch artifact is missing required information. Route missing execution detail back to `task-graph-to-batch`.
- Do not paste full source files, full task graphs, full PRDs, or full build logs into subagent prompts. Pass paths plus the exact task row and acceptance criteria.
- For large or long-running builds, prefer one fresh session per batch. `work/praxiskit-context.md`, the current execution batch, and `work/SUBAGENT.md` are the resume surface.
- Use targeted file reads (`rg`, narrow `sed` ranges, or equivalent) for review and debugging. Avoid repeated full reads of files already inspected in the same turn.
- Update `work/task-graph.md` in at most two grouped edits: one to mark all selected tasks `[/]`, and one after validation to mark final `[x]` / `[!]` statuses. Do not edit the task table once per task.
- Keep `work/build-log-{n}.md` concise: file lists, summaries, validation, and residual risks. Do not embed full source or full command logs unless needed to explain a failure.

## Execution Workflow

1. **Authorization check** (above). Refuse if not authorized.
2. **Re-validate baseline.** Re-run preflight commands. If baseline fails and this is not a baseline-repair batch, refuse — the user should re-batch. If this is a baseline-repair batch, proceed only with `T0.0`.
3. **Mark selected tasks `[/]`** in `work/task-graph.md` with one grouped edit covering every selected task.
4. **Dispatch parallel groups one at a time.**
   - If a parallel group contains 2+ tasks, the orchestrator MUST NOT implement those tasks directly.
   - In Claude Code, use the `Task` tool once per task in the group before doing any implementation work. Dispatch all tasks in the group first, then wait for results.
   - In Codex or other environments, use the platform's subagent/delegation tool. If no subagent tool exists, stop and report that the batch requires subagent-capable execution; do not silently fall back to sequential orchestrator edits.
   - Only a single-task sequential group may be implemented by the orchestrator without subagents.
   - For each task in a parallel group, spawn one subagent with:
     - `work/SUBAGENT.md` path
     - The exact task line and acceptance criteria from the batch
     - Dependencies already completed
     - Owned files/directories (from write_scope)
     - Instruction: not alone in the codebase; do not revert others' edits
     - Instruction: modify only the assigned task write scope. Do not update PraxisKit bookkeeping files; the orchestrator owns those.
     - Instruction: read only the context files and source paths needed for the assigned task; do not read the entire task graph or PRD unless explicitly required by the task.
     - Reporting format: `RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`
   - Use this compact subagent prompt shape:
     ```text
     Implement only {T_ID}: {title}.
     Context paths: work/SUBAGENT.md and work/execution-batch-{n}.md.
     Task row: {exact row from batch}.
     Acceptance criteria: {exact criteria from batch}.
     Owned write scope: {paths}.
     Do not edit PraxisKit bookkeeping files. Do not touch files outside the write scope. Report: RESULT: ...
     ```
5. **Review returned changes** before integration: scope compliance, files changed, frozen-contract adherence. Prefer `git diff -- {owned_paths}` or equivalent targeted review.
6. **Run validation.** Narrowest meaningful command first; broader tests if shared behavior changed. Run validation once after the parallel group is integrated unless an individual worker reports a blocking failure.
7. **Mark `[x]` / `[!]`** only after validation, with one grouped edit covering final statuses for the batch.
8. **Run batch closeout** (below).
9. **Update `work/build-log-{n}.md`** with the run details and closeout summary.
10. **Refresh `work/praxiskit-context.md`** with the next-session resume surface.
11. **Stop** when the batch is complete, validation fails, or all remaining work in the batch is blocked.

## Batch Closeout

Before stopping, close the iteration loop:

1. **Reconcile selected tasks.** Confirm every selected task is `[x]` or `[!]`. If validation failed after partial work, mark affected tasks `[!]` with the failure reason and leave unrelated unstarted tasks `[ ]`.
2. **Classify leftovers.** Record each remaining item as one of:
   - `next_batch`: still in scope and unblocked
   - `blocked`: needs external/user/project decision
   - `deferred`: intentionally out of this recipe or wave
   - `cleanup`: documentation, test debt, or refactor follow-up discovered during execution
3. **Close documentation drift.** Update only the docs touched by the batch or required to explain the current state. Do not rewrite PRDs, task graphs, or prior logs for style.
4. **Archive transient notes.** If temporary clarification files, scratch plans, or superseded notes were created, move them under `work/archive/` only after summarizing any still-relevant content in `work/build-log-{n}.md`. Leave files in place when their lifecycle is unclear.
5. **Write the next entry point.** In `work/praxiskit-context.md`, keep a compact handoff:
   - last completed batch and validation result
   - open blockers and owners/decisions needed
   - recommended next skill: `task-graph-to-batch`, `build-to-review-packet`, or stop blocked
   - exact artifacts to read in a fresh session

Closeout is not user acceptance. Formal accept/revise/continue decisions still go through `build-to-review-packet` and `review-to-acceptance`.

## Build Log Format (work/build-log-{n}.md)

```markdown
# Build Log: Batch {n} — {timestamp}

## Scope
- Tasks: [T_ID, T_ID, ...]
- Mode: parallel / sequential / mixed
- Baseline: pass / fail

## Assignments
| Task | Agent | Write Scope | Parallel Group | Result |
|------|-------|-------------|----------------|--------|
| T1.1 | subagent-{n} / orchestrator-single-task | `path/` | A | done / blocked: reason |

## Validation
- `{command}`: {pass/fail}

## Review
- Files reviewed: {list}
- Scope compliance: {note any violations}
- Actual parallelism used: {N agents}
- Dispatch method: Task tool / platform subagent tool / orchestrator-single-task

## Follow-Ups
- {blocked task, failed check, or next batch}

## Closeout
- Leftovers: next_batch / blocked / deferred / cleanup
- Archived transient notes: {paths or none}
- Next entry point: task-graph-to-batch / build-to-review-packet / stop blocked
- Fresh-session resume: {minimal artifact list}
```

## Hand-Off

After the batch completes, the next step depends on remaining work:

- More unblocked tasks exist → invoke `task-graph-to-batch` for the next wave
- All tasks `[x]` → invoke `build-to-review-packet` to produce a review packet
- Blocked → invoke `build-to-review-packet` with partial completion noted

For long sessions, recommend starting the next batch in a fresh session using only:
- `work/praxiskit-context.md`
- the next `work/execution-batch-{n}.md`
- `work/SUBAGENT.md`
