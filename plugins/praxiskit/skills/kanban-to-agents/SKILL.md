---
name: kanban-to-agents
description: "Orchestrate implementation from work/kanban.md and work/SUBAGENT.md. Use when the user asks to execute, advance, coordinate, delegate, or parallelize a Kanban plan with agents; supports dry-run planning and explicit multi-agent execution."
---

# Kanban to Agents

Advance a dependency-aware Kanban plan through controlled agent execution.

```text
work/kanban.md + work/SUBAGENT.md -> kanban-to-agents -> code + updated kanban
```

The orchestrator owns scheduling, integration, verification, and Kanban updates. Subagents implement scoped tasks and report results.

## Modes

- **Dry-run mode:** Use when the user asks "what's next", "how would you execute this", or has not explicitly authorized agent execution. Do not spawn agents or edit task status. Output the next wave, blockers, write scopes, and proposed assignments.
- **Execution mode:** Use only when the user explicitly asks to execute, advance, coordinate, delegate, or parallelize the plan with agents. Spawn agents only for bounded sidecar work that can run in parallel.

## Inputs

Require both files unless the user gives equivalent content:

- `work/kanban.md` - dynamic task board and dependency graph
- `work/SUBAGENT.md` - static shared context for implementation agents
- `work/praxiskit-context.md` - compact cross-skill index, if present

If either file is missing, stop and recommend invoking the `prd-to-kanban` skill.

## Preflight

Before selecting tasks, establish baseline health:

- Read project scripts from `package.json`, `pyproject.toml`, `Cargo.toml`, `Makefile`, or README.
- Identify the narrowest test command and build/typecheck command.
- Run baseline checks when they are local and reasonable.
- Record failures as pre-existing blockers. If a baseline failure blocks release, add or mark an early preflight task such as `T0.0 | Restore build baseline`.
- Do not attribute baseline failures to the current wave unless the failure appears after changes.

## Scheduling Rules

1. Parse task status: `[ ]` pending, `[/]` in progress, `[x]` done, `[!]` blocked.
2. A task is unblocked only when all dependencies in `[T...]` are `[x]`.
3. Prefer the earliest dependency layer and wave.
4. Keep critical-path tasks moving. The main orchestrator should handle the immediate blocking task locally when waiting would stall the whole run.
5. Delegate only tasks with disjoint write scopes. Never assign two agents to overlapping files or contract files marked frozen in `SUBAGENT.md`.
6. Limit a default wave to three parallel workers unless the user asks for more.

## Execution Workflow

1. Check repo status and run preflight.
2. Select the next executable batch, not just the next wave.
3. Announce the next executable batch before work:
   - unblocked tasks
   - dependencies already satisfied
   - safe parallel pairs
   - tasks kept local and reason
4. Mark selected tasks `[/]` only after work starts.
5. For each delegated task, provide:
   - `SUBAGENT.md` path
   - exact Kanban task line and acceptance criteria
   - dependencies already completed
   - owned files/directories
   - instruction that the worker is not alone in the codebase and must not revert others' edits
   - report format: `RESULT: {T_ID} | files: [...] | summary: ... | validation: ...`
6. Review returned changes before integration, including scope compliance and files changed.
7. Run the narrowest meaningful validation, then broader tests if shared behavior changed.
8. Mark `[x]` only after verification passes. Mark `[!]` with a short reason when blocked.
9. Refresh overview counts, update `work/praxiskit-context.md`, and write a run log when execution occurred.

## Run Log

For execution mode, append or create:

```text
work/agent-runs/YYYYMMDD-HHMM.md
```

Use this format:

```markdown
# Agent Run: {timestamp}

## Scope
- Tasks:
- Mode:
- Baseline:
- Parallel Batch:

## Assignments
| Task | Agent | Write Scope | Parallel Batch | Result |
|------|-------|-------------|----------------|--------|

## Validation
- {command}: {result}

## Review
- Files reviewed:
- Scope compliance:
- Actual parallelism used:

## Follow-Ups
- {blocked task, failed check, or next wave}
```

Stop when the selected wave is complete, validation fails, all remaining work is blocked, or the user requested a bounded run.

---

## `work/praxiskit-context.md` Format

A compact cross-skill index updated by each skill in the pipeline. Keeps the pipeline coherent across sessions.

```markdown
# PraxisKit Context: {project}

## Source Files
- idea: `work/idea.md`
- prd: `work/PRD.md`
- kanban: `work/kanban.md`
- subagent: `work/SUBAGENT.md`

## Current Milestone
{ID} — {title} ({status})

## Canonical Constraints
- {frozen contract path} — {what it defines}

## Open Blockers
- {T_ID or gate}: {reason}

## Latest Validation
- `{command}` → {result} ({date})

## Last Updated By
{skill-name} on {date}
```

Keep it under 40 lines. Only list what is current — remove resolved blockers and outdated validation.
