---
schema_name: acceptance-decision
schema_version: "3.0"
used_by: review-to-acceptance
input_file: work/review.md
output_file: work/acceptance.md
---

# Acceptance Decision Schema v3.0

The `acceptance_decision` artifact is the user's final accept/revise/continue verdict on a review packet. It is the closing artifact of any recipe — every recipe ends with one of these.

## Required Fields

| Field | State | Notes |
|---|---|---|
| review_packet_path | filled-by-user (file ref) | Path to the review.md being decided on |
| decision | filled-by-user | One of: `accept_wave`, `revise`, `continue_next_wave`, `not_accept_yet` |
| decision_date | inferable | ISO date (today) |
| decision_rationale | filled-by-user | 1-3 sentences explaining the choice |
| follow_up_tasks | filled-by-user (if revise) | List of follow-ups the user wants — feeds into a new task graph |
| accepted_scope | filled-by-user (if accept_wave or partial) | Which parts are accepted; rest stays open |

## Decision Semantics

| Decision | Meaning | Next Recipe Step |
|---|---|---|
| `accept_wave` | This wave's deliverables meet acceptance criteria | If more waves remain in task graph, continue with `task-graph-to-batch`; if not, project closes |
| `revise` | The wave has issues that must be fixed before further work | Generate a new task graph (or amend existing) with follow-up tasks; loop back through `batch-to-build` |
| `continue_next_wave` | This wave is fine; move on to the next wave | Invoke `task-graph-to-batch` for the next batch |
| `not_accept_yet` | Decision deferred — user wants more inspection time | Recipe pauses; no further automatic work |

## Authorization

This transform REQUIRES user input. It MUST NOT auto-decide based on review-packet contents alone. The user explicitly states the decision.

## Output Format (work/acceptance.md)

```markdown
# Acceptance Decision: {project} — {date}

## Source
- Review packet: `work/review.md`
- Recipe: {light | standard | heavy | from-prd | bug-fix}

## Decision
**{accept_wave | revise | continue_next_wave | not_accept_yet}**

## Rationale
{1-3 sentences}

## Accepted Scope
- {what is accepted, if applicable}

## Follow-Up Tasks
- {if decision is revise, list tasks to add to a new task graph}

## Recipe Continuation
- Next step: {recipe-specific next transform}
```

## Changelog

- v3.0 (2026-04-30): Initial schema. Formalizes the closing step of every recipe.
