---
recipe_name: light
recipe_version: "3.0"
description: "Skip idea/PRD; produce a task graph directly from a seed for toy projects and prototypes."
---

# Recipe: Light

The shortest path from rough seed to acceptance. Skips `idea` and `prd` artifacts entirely. Ideal when the user can directly state acceptance criteria without going through product framing.

## Chain

1. **`seed-to-task-graph`** — produces `work/task-graph.md` + `work/SUBAGENT.md` directly from seed
2. **`task-graph-to-batch`** — selects the next executable batch into `work/execution-batch-{n}.md`
3. **`batch-to-build`** — executes the authorized batch, produces code + `work/build-log-{n}.md`
4. *(loop step 2-3 if more batches remain)*
5. **`build-to-review-packet`** — produces `work/review.md` mapping build to acceptance criteria
6. **`review-to-acceptance`** — collects user's accept/revise/continue decision into `work/acceptance.md`

## When to Use

Use `light` when ANY of these apply:

- Toy projects, scripts, refactors, or single-script automations
- The user can state "done = X" directly without a PRD
- No external stakeholders need to read an idea or PRD
- The work is reversible enough that gate-heavy planning is overhead

Do NOT use `light` when:
- Multiple stakeholders need to align on intent → use `standard`
- Acceptance criteria require product reasoning → use `standard`
- The work involves significant parallelizable scope → use `heavy`

## Authorization Boundaries

- Pre-`batch-to-build`: planning only. `seed-to-task-graph` and `task-graph-to-batch` write artifacts but never modify project source.
- `batch-to-build`: REQUIRES explicit user authorization. The execution-batch artifact defaults to `dry-run`; `batch-to-build` may upgrade a current dry-run batch only after validating that the task graph and baseline are still current.
- `review-to-acceptance`: REQUIRES user input. Never auto-decides.

## Notes

### Schema gate still fires

Light recipe is NOT a "skip the gate" recipe. `seed-to-task-graph` still:
- Fires the clarification gate per `references/clarification-gate.md`
- Honors cross-field rules from `idea.schema.md` (e.g., cache_metadata_fields for cache-related seeds)
- Refuses to fabricate domain-specific lists

The light recipe just skips intermediate documents (idea, PRD), not the user-participation gate.

### What gets written

- `work/seed.md` (if user provides a seed file) — input
- `work/task-graph.md` + `work/SUBAGENT.md` — produced by step 1
- `work/clarify-seed.md` — only if the gate needs bulk/async fallback (archived after)
- `work/execution-batch-{n}.md` — produced by step 2
- `work/build-log-{n}.md` — produced by step 3
- `work/review.md` — produced by step 5
- `work/acceptance.md` — produced by step 6
- `work/praxiskit-context.md` — updated by every step

After acceptance, completed-loop artifacts move to `work/archive/iterations/`; active `work/` keeps only the next-loop carry-forward set.

### What does NOT get written

- `work/idea.md` (use standard recipe if you want one)
- `work/PRD.md` (use standard recipe if you want one)
