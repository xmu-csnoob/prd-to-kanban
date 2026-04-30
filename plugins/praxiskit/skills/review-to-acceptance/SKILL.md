---
name: review-to-acceptance
description: "Take work/review.md and collect the user's formal accept/revise/continue/not-yet decision into work/acceptance.md. Closes any PraxisKit recipe. REQUIRES user input — does NOT auto-decide."
---

# Review to Acceptance

Convert a review packet into a formal acceptance decision. This is the closing transform of every recipe.

```text
work/review.md -> review-to-acceptance -> work/acceptance.md -> recipe complete (or loop)
```

## Contract

**Inputs:** `review_packet` (`work/review.md`)
**Output:** `acceptance_decision` (`work/acceptance.md`)
**Schema:** `schemas/acceptance-decision.schema.md` (v3.0)
**Preconditions:**
- `work/review.md` exists and follows `schemas/review-packet.schema.md`
- The user is present in the session and able to make a decision
**Postconditions:**
- `work/acceptance.md` records one of: `accept_wave`, `revise`, `continue_next_wave`, `not_accept_yet`
- The decision has a 1-3 sentence rationale provided by the user
- If `revise`: follow-up tasks are listed for the next task graph
- If `accept_wave` or partial: accepted scope is explicit
**Clarification gate:** fires for the decision itself — this transform's gate is the decision question. Fires per `references/clarification-gate.md` (the decision is a `choice` kind gap with 4 options).
**Side effects:**
- Writes `work/acceptance.md`
- Updates `work/praxiskit-context.md` to mark recipe state (closed / looping)
**Stop boundary:** Does NOT execute follow-up tasks. Does NOT modify code or upstream artifacts. After the decision is recorded, the user (or a recipe-level orchestrator) decides what to do next based on the decision value.

## Workflow

1. **Read `work/review.md`.** Summarize the review-packet's key findings: what passes, what's a gap, what's unknown.
2. **Surface the decision question.** Ask the user via `references/clarification-gate.md`:
   - Question: "Based on this review, what's your decision?"
   - Kind: `choice` (single-select)
   - Options:
     - A. `accept_wave` — this wave is complete and accepted
     - B. `continue_next_wave` — this wave is fine; advance to the next
     - C. `revise` — issues must be fixed; record follow-ups
     - D. `not_accept_yet` — pause; I want more inspection time
3. **If decision is `revise`:** ask for follow-up tasks. Use `list` kind gap. Each follow-up should be specific enough to become a task in a new task graph.
4. **Collect rationale.** Ask for a 1-3 sentence explanation of the decision.
5. **Write `work/acceptance.md`** per `schemas/acceptance-decision.schema.md`.
6. **Update `work/praxiskit-context.md`** with the recipe state.
7. **Report next steps based on decision:**
   - `accept_wave` + more waves → recommend `task-graph-to-batch`
   - `accept_wave` + last wave → recipe complete; congratulate
   - `continue_next_wave` → recommend `task-graph-to-batch`
   - `revise` → recommend invoking the recipe's task-graph-producing transform with the follow-up list
   - `not_accept_yet` → pause; no further automatic work

## Authorization Rule

This transform NEVER auto-decides. If the user does not provide a decision, the transform stops and waits. There is no default option.
