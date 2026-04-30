---
recipe_name: from-prd
recipe_version: "3.0"
description: "Skip seed/idea; start from an existing PRD document and run through to acceptance."
---

# Recipe: From PRD

For when a PRD already exists (written by humans, generated elsewhere, or carried over from a previous project iteration) and re-deriving intent would be wasted work.

## Chain

1. *(prerequisite: `work/PRD.md` exists and follows `schemas/prd.schema.md`)*
2. **`prd-to-task-graph`** — produces `work/task-graph.md` + `work/SUBAGENT.md`
3. **`task-graph-to-batch`** — produces `work/execution-batch-{n}.md`
4. **`batch-to-build`** — executes the authorized batch
5. *(loop step 3-4 if more batches remain)*
6. **`build-to-review-packet`** — produces `work/review.md`
7. **`review-to-acceptance`** — collects user decision

## When to Use

Use `from-prd` when:

- A PRD document already exists (e.g., from another team, previous project, or external tool)
- The PRD follows or can be massaged to follow `schemas/prd.schema.md`
- Going through `seed-to-idea` → `idea-to-prd` would just re-derive what's already written

Do NOT use `from-prd` when:

- The PRD is incomplete or has unresolved blocking Open Questions → run `idea-to-prd` to fix it
- The PRD lacks acceptance criteria for FRs → run `idea-to-prd` to gate-collect them
- Stakeholders haven't aligned on the PRD content → run `idea-to-prd` to validate

## Authorization Boundaries

- Pre-`batch-to-build`: planning only.
- `batch-to-build`: REQUIRES explicit user authorization. It may upgrade a current dry-run batch only after validating that the task graph and baseline are still current.
- `review-to-acceptance`: REQUIRES user input.

## Notes

### PRD adoption check

`prd-to-task-graph` runs its standard pre-flight checks on the imported PRD:
- All blocking Open Questions resolved
- All FRs have acceptance criteria
- At least 1 milestone defined

If any check fails, the recipe routes back to `idea-to-prd` to fix the PRD before retrying.

### No idea.md is produced

This recipe deliberately skips `idea.md`. If the user later wants product framing, they can manually write or extract an idea document — but the chain doesn't require it.

### Schema compatibility

If the imported PRD doesn't follow `schemas/prd.schema.md` (e.g., it's a Confluence doc with different headings), the user must restructure it first. `prd-to-task-graph` will refuse to consume a non-conforming PRD.
