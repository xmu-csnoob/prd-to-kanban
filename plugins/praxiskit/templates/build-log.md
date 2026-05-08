# Build Log: Batch {n} - {timestamp}

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
- Next entry point: task-graph-to-batch / review-and-accept / stop blocked
- Fresh-session resume: {minimal artifact list}
