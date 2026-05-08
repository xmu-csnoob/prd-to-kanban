---
name: next-iteration
description: "Advance one PraxisKit loop step from work/praxiskit-context.md or the active work/ artifacts. Use after acceptance archive/reset, after a stop boundary, or when resuming a PraxisKit recipe and you want the next transform chosen safely."
---

# Next Iteration

Advance exactly one PraxisKit step from the current project state. This is a thin driver; it routes to existing transform skills instead of duplicating their contracts.

```text
work/praxiskit-context.md + active work/ artifacts -> next-iteration -> one next transform
```

## Contract

**Inputs:** `work/praxiskit-context.md` if present, active carry-forward files, latest archive manifest if needed
**Output:** the selected transform's normal artifact, or a blocked/complete report
**Preconditions:** run from the project root; at least one PraxisKit artifact or archive pointer exists
**Postconditions:** at most one transform is run; execution and acceptance gates remain enforced
**Stop boundary:** Does not bypass any transform's stop boundary. Source changes can happen only through `batch-to-build` after its authorization checks pass.

## Interaction Mode

Prefer host-native choice input for continuation decisions. The user should not need to remember the next slash command when PraxisKit already knows the safe next step. Choice prompts may authorize planning steps such as `task-graph-to-batch`, but they never authorize code execution or acceptance decisions unless the selected transform explicitly asks for that scoped decision.

## Workflow

1. Read `work/praxiskit-context.md` first. If it names a next skill and exact files to read, trust that unless the files are missing.
2. If context is missing or stale, infer the next skill from active artifacts:
   - `work/review.md` -> `review-to-acceptance`
   - current `work/build-log-*.md` with task graph -> `build-to-review-packet`
   - current `work/execution-batch-*.md` -> `batch-to-build`
   - `work/task-graph.md` + `work/SUBAGENT.md` -> `task-graph-to-batch`
   - `work/PRD.md` -> `prd-to-task-graph`
   - `work/idea.md` -> `idea-to-prd`
   - seed only -> ask which recipe entry to use (`seed-to-task-graph` for light, `seed-to-idea` for standard/heavy)
3. If the context points at an archive, read only that archive's `manifest.md` unless a carried-forward file is missing.
4. Route to the selected transform skill and follow that skill's `SKILL.md` exactly.
5. Stop after that one transform reaches its boundary unless that transform offers a host-native continuation choice and the user selects it. Report:
   - selected next skill
   - artifacts read
   - artifact written or blocker
   - recommended next entry point

## Routing Rules

- `continue_next_wave`, or `accept_wave` with remaining tasks -> `task-graph-to-batch`.
- `revise` -> use `work/follow-ups.md`; pick the recipe's task-graph transform from context. If unclear, ask with host-native choice input.
- completed `accept_wave` with no remaining tasks -> stop complete; do not start a new seed unless the user asks.
- `not_accept_yet` -> stop; the user still owns the acceptance decision.
- Dry-run execution batches require `batch-to-build` authorization. Never convert a dry-run batch by inference.

## Context Budget

Read the resume surface, not the whole archive history: `work/praxiskit-context.md`, the active artifact for the chosen step, and `work/SUBAGENT.md` only when the chosen step needs it.
