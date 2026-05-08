---
name: review-and-accept
description: "Produce work/review.md from build evidence, surface the accept/revise/continue/not-yet decision, archive the iteration, and reset work/ for the next loop."
---

# Review and Accept

Close the implementation loop. Produces a structured review packet, asks the user to decide, then archives the completed iteration.

```text
build + (idea | PRD | task-graph) -> review-and-accept -> work/review.md + work/acceptance.md -> archive iteration
```

## Contract

**Inputs:** `build` (code + tests + run logs in repo) + at least one of `work/idea.md`, `work/PRD.md`, `work/task-graph.md`
**Output:** `work/review.md` per `schemas/review-packet.schema.md` (v3.0); `work/acceptance.md` per `schemas/acceptance-decision.schema.md` (v3.0); archived iteration under `work/archive/iterations/{timestamp}-{decision}/` unless decision is `not_accept_yet`
**Preconditions:** implementation exists; user is present (acceptance step requires user input — no auto-decide, no default)
**Stop boundary:** Does NOT execute follow-up tasks or modify project source. Does NOT delete archived history. May move PraxisKit bookkeeping artifacts under `work/`.

## Two Phases

### Phase 1 — Build the Review Packet (read-only)

Load the best evidence available:

- `work/idea.md` — original intent (standard/heavy recipe)
- `work/PRD.md` — requirements + acceptance criteria (standard/heavy recipe)
- `work/task-graph.md` — planned tasks + completion state
- `work/SUBAGENT.md` — shared execution constraints
- `work/praxiskit-context.md` — milestone, blockers, validation index
- `work/build-log-{n}.md` — execution logs from `batch-to-build`
- Git diff, test output, screenshots, local app URL, generated artifacts

Workflow:

1. **Recover the promise.** Summarize what the user originally wanted from the highest-priority upstream artifact (PRD if present, else idea, else task-graph intent).
2. **Determine review type:**
   - `Wave Review` — one batch complete, more remaining
   - `Partial Build Review` — multiple batches complete, project not yet done
   - `Full Build Review` — all task-graph tasks `[x]`
   - `Regression Review` — verifying after a revision loop
3. **Inspect.** Review changed files, completed tasks, validation logs, runnable UI/CLI behavior.
4. **Build the acceptance match table.** For each FR (PRD recipe) or task acceptance criterion (light recipe):
   - Source: `PRD FR{N}` or `task-graph T{ID}`
   - Expectation: copy verbatim
   - Result: what the build actually does
   - Status: Pass / Gap / Unknown
   - Evidence: pointer (validation command output, file path, screenshot, or "manual inspection: {note}"). No fabrication — if no evidence, status is Unknown.
5. **Surface gaps.** List mismatches, incomplete tasks, risks. Each gap should link to a follow-up task ID if one exists.
6. **Write `work/review.md`** from `templates/review.md` per `schemas/review-packet.schema.md`. If implementation cannot be inspected, state that clearly under `## Cannot Inspect Because`.

Constraints: every Pass must have evidence. "Pass with caveats" → split into Pass + Gap rows. If task-graph completion < 100%, do NOT label as Full Build Review.

### Phase 2 — Collect Acceptance Decision (requires user)

7. **Surface the decision question** via `references/clarification-gate.md`, using host-native choice input rather than a file whenever available:
   - Question: "Based on this review, what's your decision?"
   - Kind: `choice` (single-select)
   - Options:
     - A. `accept_wave` — this wave is complete and accepted
     - B. `continue_next_wave` — this wave is fine; advance to the next
     - C. `revise` — issues must be fixed; record follow-ups
     - D. `not_accept_yet` — pause; I want more inspection time
8. **If decision is `revise`:** ask for follow-up tasks via interactive list/freeform input. Each follow-up should be specific enough to become a task in a new task graph.
9. **Collect rationale.** Ask for a 1-3 sentence explanation via interactive text input.
10. **Write `work/acceptance.md`** from `templates/acceptance.md` per `schemas/acceptance-decision.schema.md`.

### Phase 3 — Archive and Reset

Run after writing `work/acceptance.md`, **except when decision is `not_accept_yet`**.

11. Create `work/archive/iterations/{YYYYMMDD-HHMMSS}-{decision}/`.
12. Move current PraxisKit artifacts into the archive:
    - `work/seed.md`, `work/idea.md`, `work/PRD.md`, `work/task-graph.md`, `work/SUBAGENT.md`
    - `work/execution-batch-*.md`, `work/build-log-*.md`
    - `work/review.md`, `work/acceptance.md`
    - `work/clarify-*.md`, `work/follow-ups.md`, any existing `work/clarify-archive/`
13. Do NOT move `work/archive/` itself. Do NOT move project source or build artifacts outside `work/`.
14. Write `{archive}/manifest.md` from `templates/archive-manifest.md`.
15. Recreate active `work/` with only next-loop carry-forward files:
    - `continue_next_wave`, or `accept_wave` with remaining tasks: copy `task-graph.md` + `SUBAGENT.md` back from the archive.
    - `revise`: write `work/follow-ups.md` from the user's follow-up list. Copy `task-graph.md` back only if amending the existing graph; otherwise keep it archived.
    - completed `accept_wave`: carry forward no recipe artifacts.
16. Write a fresh `work/praxiskit-context.md`: last archive path, decision and rationale summary, carried-forward files, next skill, exact files for a fresh session.

## Routing After Decision

- `accept_wave` + more waves → `task-graph-to-batch`
- `accept_wave` + last wave → recipe complete; active `work/` contains only context pointing at the archive
- `continue_next_wave` → `task-graph-to-batch`
- `revise` → recipe's task-graph-producing transform with `work/follow-ups.md`
- `not_accept_yet` → pause; no archive or cleanup

## Authorization Rule

This transform NEVER auto-decides. If the user does not provide a decision, Phase 2 stops and waits. There is no default option. Phase 1 may run without the user (review packet is read-only); Phase 2 and Phase 3 require the user.
