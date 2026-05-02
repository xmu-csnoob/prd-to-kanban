---
name: review-to-acceptance
description: "Collect acceptance, archive the finished iteration, and reset work/ for the next loop."
---

# Review to Acceptance

Convert a review packet into a formal acceptance decision. This is the closing transform of every recipe.

```text
work/review.md -> review-to-acceptance -> archive iteration -> clean work/ for next loop
```

## Contract

**Inputs:** `review_packet` (`work/review.md`)
**Output:** `acceptance_decision` (`work/acceptance.md`, then archived unless decision is `not_accept_yet`)
**Schema:** `schemas/acceptance-decision.schema.md` (v3.0)
**Preconditions:**
- `work/review.md` exists and follows `schemas/review-packet.schema.md`
- The user is present in the session and able to make a decision
**Postconditions:**
- `work/acceptance.md` records one of: `accept_wave`, `revise`, `continue_next_wave`, `not_accept_yet`
- The decision has a 1-3 sentence rationale provided by the user
- If `revise`: follow-up tasks are listed for the next task graph
- If `accept_wave` or partial: accepted scope is explicit
- If decision is not `not_accept_yet`: current iteration artifacts are archived and active `work/` is reduced to the next-loop carry-forward set
**Clarification gate:** fires for the decision itself — this transform's gate is the decision question. Fires per `references/clarification-gate.md` (the decision is a `choice` kind gap with 4 options).
**Side effects:**
- Writes `work/acceptance.md`
- Moves completed PraxisKit work artifacts into `work/archive/iterations/{timestamp}-{decision}/`
- Writes archive manifest from `templates/archive-manifest.md`
- Rewrites `work/praxiskit-context.md` as the next-loop resume surface
**Stop boundary:** Does NOT execute follow-up tasks, modify project source, or delete archived history. It may move PraxisKit bookkeeping artifacts under `work/`.

## Workflow

1. **Read `work/review.md`.** Summarize the review-packet's key findings: what passes, what's a gap, what's unknown.
2. **Surface the decision question.** Ask the user via `references/clarification-gate.md`, using host-native choice input rather than a file whenever available:
   - Question: "Based on this review, what's your decision?"
   - Kind: `choice` (single-select)
   - Options:
     - A. `accept_wave` — this wave is complete and accepted
     - B. `continue_next_wave` — this wave is fine; advance to the next
     - C. `revise` — issues must be fixed; record follow-ups
     - D. `not_accept_yet` — pause; I want more inspection time
3. **If decision is `revise`:** ask for follow-up tasks with interactive list/freeform input. Each follow-up should be specific enough to become a task in a new task graph.
4. **Collect rationale.** Ask for a 1-3 sentence explanation of the decision with interactive text input.
5. **Write `work/acceptance.md`** from `templates/acceptance.md` per `schemas/acceptance-decision.schema.md`.
6. **Close the iteration** (below).
7. **Report next steps based on decision:**
   - `accept_wave` + more waves -> `task-graph-to-batch`
   - `accept_wave` + last wave -> recipe complete; active `work/` contains only context pointing at the archive
   - `continue_next_wave` -> `task-graph-to-batch`
   - `revise` -> recipe's task-graph-producing transform with `work/follow-ups.md`
   - `not_accept_yet` -> pause; no archive or cleanup

## Iteration Archive and Reset

Run this after writing `work/acceptance.md`, except when decision is `not_accept_yet`.

1. Create `work/archive/iterations/{YYYYMMDD-HHMMSS}-{decision}/`.
2. Move current PraxisKit artifacts into that archive:
   - `work/seed.md`
   - `work/idea.md`
   - `work/PRD.md`
   - `work/task-graph.md`
   - `work/SUBAGENT.md`
   - `work/execution-batch-*.md`
   - `work/build-log-*.md`
   - `work/review.md`
   - `work/acceptance.md`
   - `work/clarify-*.md`
   - `work/follow-ups.md`
   - any existing `work/clarify-archive/`
3. Do not move `work/archive/` itself. Do not move project source or build artifacts outside `work/`.
4. Write `{archive}/manifest.md` from `templates/archive-manifest.md`.
5. Recreate active `work/` with only next-loop carry-forward files:
   - `continue_next_wave`, or `accept_wave` with remaining tasks: copy `task-graph.md` and `SUBAGENT.md` back from the archive if present.
   - `revise`: write `work/follow-ups.md` from the user's follow-up list. Copy `task-graph.md` back only if the decision says to amend the existing graph; otherwise keep it archived.
   - completed `accept_wave`: carry forward no recipe artifacts.
6. Write a fresh `work/praxiskit-context.md` containing:
   - last archive path
   - decision and rationale summary
   - carried-forward files
   - next skill
   - exact files to read in a fresh session

This reset makes recipes recursive: each accepted loop leaves a clean active `work/` plus a durable archive of the previous loop.

## Authorization Rule

This transform NEVER auto-decides. If the user does not provide a decision, the transform stops and waits. There is no default option.
