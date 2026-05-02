---
recipe_name: standard
recipe_version: "3.0"
description: "Full pipeline from rough seed to acceptance, with idea + PRD documents for traceability."
---

# Recipe: Standard

The default recipe. Produces both `idea` and `prd` artifacts so intent and engineering contract are explicit and auditable.

## Chain

1. **`seed-to-idea`** — produces `work/idea.md` from seed
2. **`idea-to-prd`** — produces `work/PRD.md` from idea (No-New-Fields Rule applies)
3. **`prd-to-task-graph`** — produces `work/task-graph.md` + `work/SUBAGENT.md` from PRD
4. **`task-graph-to-batch`** — selects the next executable batch into `work/execution-batch-{n}.md`
5. **`batch-to-build`** — executes the authorized batch
6. *(loop step 4-5 if more batches remain)*
7. **`build-to-review-packet`** — produces `work/review.md`
8. **`review-to-acceptance`** — collects user decision into `work/acceptance.md`

## When to Use

Use `standard` when ANY of these apply:

- Multiple users / stakeholders need to align on intent
- Acceptance criteria require derivation from product goals
- The work touches product surface that benefits from explicit non-goals
- A PRD will outlive the implementation (e.g., for handoff to another team)

Do NOT use `standard` when:
- The work is a script or one-off where idea/PRD are overhead → use `light`
- The project has significant parallelizable scope → use `heavy`
- A PRD already exists → use `from-prd`

## Authorization Boundaries

- Pre-`batch-to-build`: planning only. Steps 1-4 write artifacts but never modify project source.
- `batch-to-build`: REQUIRES explicit user authorization (default `dry-run`). It may upgrade a current dry-run batch only after validating that the task graph and baseline are still current.
- `review-to-acceptance`: REQUIRES user input.

## Notes

### Each step has its own gate

Every transform that produces a new artifact has its own clarification gate per `references/clarification-gate.md`:
- `seed-to-idea` gates on missing required idea fields and forbidden-to-infer cross-field triggers
- `idea-to-prd` gates on PRD fields that have no `[user]` source in idea.md (No-New-Fields Rule)
- `prd-to-task-graph` does NOT gate but pre-flight validates blocking Open Questions and missing acceptance criteria

If a gate fires, that step first uses the host's interactive decision/input mechanism (Claude Code decision UI, Codex equivalent, or direct chat fallback). It writes `work/clarify-{stage}.md` only for bulk/async clarification or long nested answers.

### Inter-stage carry rules

- idea.md Open Questions carry forward to PRD's Open Questions if not resolved
- PRD's blocking Open Questions must be resolved before `prd-to-task-graph` runs
- task-graph blocking conditions (missing acceptance criteria, no milestones) route back to `idea-to-prd`

### What gets written

All v3 artifacts: `idea.md`, `PRD.md`, `task-graph.md`, `SUBAGENT.md`, `execution-batch-{n}.md`, `build-log-{n}.md`, `review.md`, `acceptance.md`, plus any `clarify-*.md` files (archived after gate completes).
