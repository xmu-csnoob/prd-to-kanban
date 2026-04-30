---
name: batch-to-build
description: "Execute an authorized work/execution-batch-{n}.md by spawning agents (or running tasks locally) and produce a build artifact. REQUIRES explicit user authorization to run; otherwise refuses and reports dry-run only."
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
**Clarification gate:** does NOT fire. Authorization gating is the only user interaction.
**Side effects:**
- Modifies project source code
- May spawn subagents per parallel group
- Writes `work/build-log-{n}.md`
- Updates `work/task-graph.md` task statuses
- Updates `work/praxiskit-context.md` with last-run validation
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

## Execution Workflow

1. **Authorization check** (above). Refuse if not authorized.
2. **Re-validate baseline.** Re-run preflight commands. If baseline fails and this is not a baseline-repair batch, refuse — the user should re-batch. If this is a baseline-repair batch, proceed only with `T0.0`.
3. **Mark selected tasks `[/]`** in `work/task-graph.md`.
4. **Dispatch parallel groups one at a time.**
   - For each task in a parallel group, spawn one subagent with:
     - `work/SUBAGENT.md` path
     - The exact task line and acceptance criteria from the batch
     - Dependencies already completed
     - Owned files/directories (from write_scope)
     - Instruction: not alone in the codebase; do not revert others' edits
     - Instruction: modify only the assigned task write scope. Do not update PraxisKit bookkeeping files; the orchestrator owns those.
     - Reporting format: `RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`
5. **Review returned changes** before integration: scope compliance, files changed, frozen-contract adherence.
6. **Run validation.** Narrowest meaningful command first; broader tests if shared behavior changed.
7. **Mark `[x]`** only after validation passes. Mark `[!]` with a short reason if blocked.
8. **Update `work/build-log-{n}.md`** with the run details.
9. **Refresh `work/praxiskit-context.md`**.
10. **Stop** when the batch is complete, validation fails, or all remaining work in the batch is blocked.

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
| T1.1 | ... | `path/` | A | done / blocked: reason |

## Validation
- `{command}`: {pass/fail}

## Review
- Files reviewed: {list}
- Scope compliance: {note any violations}
- Actual parallelism used: {N agents}

## Follow-Ups
- {blocked task, failed check, or next batch}
```

## Hand-Off

After the batch completes, the next step depends on remaining work:

- More unblocked tasks exist → invoke `task-graph-to-batch` for the next wave
- All tasks `[x]` → invoke `build-to-review-packet` to produce a review packet
- Blocked → invoke `build-to-review-packet` with partial completion noted
