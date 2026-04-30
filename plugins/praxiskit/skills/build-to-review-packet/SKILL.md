---
name: build-to-review-packet
description: "Inspect an implemented build and produce work/review.md — a structured review packet mapping delivered behavior to acceptance criteria. Use after batch-to-build or any implementation work, before review-to-acceptance."
---

# Build to Review Packet

Close the implementation loop by producing a structured, inspectable review packet. The review packet is a typed artifact (`schemas/review-packet.schema.md`); it does NOT make the acceptance decision — that's `review-to-acceptance`.

```text
build + (idea | PRD | task-graph) -> build-to-review-packet -> work/review.md -> review-to-acceptance
```

## Contract

**Inputs:** `build` (code + tests + run logs in repo) plus any of `idea`, `prd`, `task_graph` for context
**Output:** `review_packet` (`work/review.md`)
**Schema:** `schemas/review-packet.schema.md` (v3.0)
**Preconditions:**
- An implementation exists (code changes, tests, or runnable artifacts)
- At least one upstream artifact (idea, PRD, or task_graph) provides acceptance criteria to map against
**Postconditions:**
- `work/review.md` lists what was built, demo path, evidence, and gap analysis
- Every upstream acceptance criterion is mapped to one of: Pass / Gap / Unknown
- Every "Pass" has a concrete evidence pointer (validation command output, file path, screenshot, or "manual inspection: {note}")
- The review packet is read-only; this transform does not mutate code, tests, or upstream artifacts
**Clarification gate:** does NOT fire. This is a read-only inspection transform.
**Side effects:**
- Writes `work/review.md`
- Updates `work/praxiskit-context.md` with last-validation entry
**Stop boundary:** Does NOT make the acceptance decision; that belongs to `review-to-acceptance`. Does NOT modify project source.

## Inputs

Load the best available evidence:

- `work/idea.md` — original intent (if standard/heavy recipe)
- `work/PRD.md` — requirements and acceptance criteria (if standard/heavy recipe)
- `work/task-graph.md` — planned tasks and completion state
- `work/SUBAGENT.md` — shared execution constraints
- `work/praxiskit-context.md` — compact milestone, blockers, and validation index
- `work/build-log-{n}.md` — execution logs from `batch-to-build`
- Git diff, test output, screenshots, local app URL, or generated artifacts

If the implementation cannot be run or inspected, state that clearly under `## Cannot Inspect Because` and produce a review packet from static evidence.

## Workflow

1. **Recover the promise.** Summarize what the user originally wanted from the highest-priority upstream artifact (PRD if present, else idea, else task-graph intent).
2. **Determine review type.** Use task-graph completion and recipe context to choose:
   - `Wave Review` — one batch complete, more remaining
   - `Partial Build Review` — multiple batches complete, project not yet done
   - `Full Build Review` — all task-graph tasks `[x]`
   - `Regression Review` — verifying after a revision loop
3. **Inspect the result.** Review changed files, completed tasks, validation logs, and any runnable UI or CLI behavior.
4. **Build the acceptance match table.** For each FR (in PRD recipes) or task acceptance criterion (in light recipe), record:
   - Source: `PRD FR{N}` or `task-graph T{ID}`
   - Expectation: copy verbatim
   - Result: what the build actually does
   - Status: Pass / Gap / Unknown
   - Evidence: pointer to evidence (no fabrication; if no evidence, status is Unknown)
5. **Surface gaps.** List mismatches, incomplete tasks, risks. Each gap should link to a follow-up task ID if one exists in the task-graph.
6. **Write `work/review.md`** per `schemas/review-packet.schema.md`.
7. **Update `work/praxiskit-context.md`** with last-validation entry.
8. **Hand off** to `review-to-acceptance` for the formal decision.

## Output Format

See `schemas/review-packet.schema.md` for the canonical format. Key constraints:

- Every Pass must have evidence
- "Pass with caveats" → split into Pass + Gap rows
- If task-graph completion < 100%, do NOT label as Full Build Review
- Keep output user-facing; link to files rather than dumping implementation detail

## Stop Boundary

This transform writes the review packet and stops. The acceptance decision (accept_wave / revise / continue_next_wave / not_accept_yet) belongs to `review-to-acceptance`, which requires explicit user input.
