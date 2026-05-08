---
name: auto-iterate
description: "Drive multiple PraxisKit steps or waves by repeatedly using next-iteration until a user checkpoint, blocker, validation failure, completion, or iteration budget is reached. Never auto-authorizes execution or acceptance."
---

# Auto Iterate

Run a bounded PraxisKit loop by repeatedly advancing the next safe transform. This is an orchestration driver, not a new transform.

```text
work/praxiskit-context.md -> auto-iterate -> repeated next-iteration steps -> checkpoint or complete
```

## Contract

**Inputs:** `work/praxiskit-context.md`, active `work/` carry-forward files, optional user budget
**Output:** updated PraxisKit artifacts plus a concise run summary
**Default budget:** stop after 3 acceptance cycles or 20 transform steps, whichever comes first
**Postconditions:** every execution still passed `batch-to-build` authorization; every acceptance still came from the user
**Stop boundary:** Stops at any required user input, blocker, failed validation, unclear route, or budget limit.

## Must Rules

- Do not auto-accept review results.
- Do not auto-authorize dry-run batches.
- Use host-native choice input for every user-owned decision when available. This keeps the loop in discuss mode: the user chooses the next safe action instead of typing the next slash command.
- Do not replace `batch-to-build` subagent rules; parallel windows remain subagent-driven.
- Do not load old archives unless the current context or active carry-forward files are insufficient.
- Do not start a new product seed after a completed `accept_wave` unless the user explicitly asks.

## Workflow

1. Establish the budget from the user request, or use the default.
2. Start each step by using `next-iteration` routing rules.
3. Continue automatically only when the last step produced a clear next entry and no user-owned decision is pending.
4. When `task-graph-to-batch` creates a dry-run batch, ask for execution with host-native decision input or one direct yes/no question. If not approved, stop.
5. When `build-to-review-packet` writes `work/review.md`, route to `review-to-acceptance`; ask the user for the decision. If the user is not ready, stop.
6. After `review-to-acceptance` archives/resets the loop:
   - continue for `continue_next_wave`
   - continue for `accept_wave` only if remaining tasks are carried forward
   - continue for `revise` only after follow-ups are captured and a clear graph-transform route exists
   - stop for completed `accept_wave` or `not_accept_yet`
7. Stop and summarize when complete, blocked, failed, unclear, or out of budget.

## Summary Format

Report only high-signal state:

- steps run
- execution approvals requested/granted
- latest archive path
- current next skill or stop reason
- files a fresh session should read
