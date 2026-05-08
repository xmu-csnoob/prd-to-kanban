---
name: next-iteration
description: "Advance PraxisKit by one or more steps from work/praxiskit-context.md. Single mode by default; budget mode loops until a checkpoint, blocker, completion, or budget. Never auto-authorizes execution or acceptance."
---

# Next Iteration

Advance PraxisKit from the current project state. Two modes:

- **Single step (default):** route to one transform, run it, stop at its boundary.
- **Budget mode** (when invoked with `budget=N` or `cycles=N`): repeatedly route + run, stopping at user checkpoints, blockers, validation failures, or budget. Replaces the former `auto-iterate` skill.

```text
work/praxiskit-context.md + active work/ artifacts -> next-iteration -> next transform(s)
```

## Contract

**Inputs:** `work/praxiskit-context.md` if present, active carry-forward files, latest archive manifest if needed; optional `budget` (`steps=N`, `cycles=N`)
**Output:** the selected transform's normal artifact(s), or a blocked/complete report
**Preconditions:** run from project root; at least one PraxisKit artifact or archive pointer exists
**Stop boundary:** Never bypasses any transform's stop boundary. Source changes only via `batch-to-build` after its checks pass. Stops at user-owned decisions, blockers, validation failures, unclear routes, or budget exhaustion.

## Interaction Mode

Prefer host-native choice input for continuation decisions. The user should not need to remember the next slash command when PraxisKit already knows the safe next step. Choice prompts may authorize planning steps (e.g., `task-graph-to-batch`); they NEVER authorize code execution or acceptance decisions unless the selected transform explicitly asks for that scoped decision.

## Routing

1. Read `work/praxiskit-context.md` first. If it names a next skill and exact files, trust that unless files are missing.
2. If context is missing/stale, infer from active artifacts:
   - `work/review.md` → `review-and-accept` (decision step)
   - current `work/build-log-*.md` with task graph → `review-and-accept` (review step)
   - current `work/execution-batch-*.md` → `batch-to-build`
   - `work/task-graph.md` + `work/SUBAGENT.md` → `task-graph-to-batch`
   - `work/PRD.md` → `prd-to-task-graph`
   - `work/idea.md` → `idea-to-prd`
   - seed only → ask which recipe (`seed-to-task-graph` for light, `seed-to-idea` for standard/heavy)
3. If context points at an archive, read only that archive's `manifest.md` unless a carry-forward file is missing.

## Routing Rules (post-acceptance)

- `continue_next_wave` or `accept_wave` with remaining tasks → `task-graph-to-batch`
- `revise` → use `work/follow-ups.md`; pick the recipe's task-graph transform from context. If unclear, ask with host-native choice input.
- Completed `accept_wave`, no remaining tasks → stop complete; do not start a new seed unless user asks.
- `not_accept_yet` → stop; user still owns the decision.
- Dry-run execution batches require `batch-to-build` authorization. Never convert a dry-run batch by inference.

## Single-Step Workflow (default)

1. Apply Routing.
2. Run the selected transform per its `SKILL.md`.
3. Stop at that transform's boundary unless it offers a host-native continuation choice and the user selects it.
4. Report: selected next skill · artifacts read · artifact written or blocker · recommended next entry point.

## Budget-Mode Workflow

Triggered when caller passes `budget=N` (steps) or `cycles=N` (acceptance cycles). Default budget when caller asks for a bounded loop without explicit numbers: **3 acceptance cycles or 20 transform steps, whichever comes first.**

1. Establish budget. Initialize step counter and cycle counter.
2. Run Single-Step Workflow.
3. Decrement step counter. If a `review-and-accept` produced an `accept_wave` or `continue_next_wave`, increment cycle counter.
4. Continue ONLY when:
   - The last step produced a clear next entry, AND
   - No user-owned decision is pending, AND
   - Budget remains.
5. Stop and surface to the user when ANY of these fire:
   - Execution authorization is needed (`task-graph-to-batch` dry-run, `batch-to-build` no `Mode: execute`)
   - Acceptance decision is needed (`review-and-accept`)
   - Blocker, failed validation, or unclear route
   - Budget exhausted
   - Recipe complete

## Must Rules (apply to both modes)

- Do not auto-accept review results.
- Do not auto-authorize dry-run batches.
- Do not start a new product seed after a completed `accept_wave` unless user asks.
- Use host-native choice input for every user-owned decision when available.
- Do not load old archives unless current context or carry-forward files are insufficient.

## Context Budget

Read the resume surface, not the whole archive history: `work/praxiskit-context.md`, the active artifact for the chosen step, and `work/SUBAGENT.md` only when the chosen step needs it.

## Summary Format (budget mode)

steps run · approvals requested/granted · acceptance cycles · latest archive path · current next skill or stop reason · files for fresh session.
