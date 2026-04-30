---
recipe_name: heavy
recipe_version: "3.0"
description: "Standard pipeline + multi-batch parallel execution for projects with significant parallelizable scope."
---

# Recipe: Heavy

Same pipeline as `standard`, but with multiple `execution_batch` artifacts running in parallel and an integrated build that consolidates the results.

## Chain

1. **`seed-to-idea`** — produces `work/idea.md`
2. **`idea-to-prd`** — produces `work/PRD.md`
3. **`prd-to-task-graph`** — produces `work/task-graph.md` + `work/SUBAGENT.md`
4. **`task-graph-to-batch`** — produces `work/execution-batch-1.md` for the first wave
5. **`batch-to-build`** — executes batch 1
6. *(loop: invoke `task-graph-to-batch` for the next wave; multiple batches may run in parallel within a wave when their write scopes are disjoint)*
7. **`build-to-review-packet`** — produces `work/review.md` over the integrated build
8. **`review-to-acceptance`** — collects user decision

## Heavy ≠ New Transforms

Heavy recipe uses the SAME transforms as standard. There is no `parallel-batch-to-build` transform. The parallelism comes from `task-graph-to-batch` producing multiple parallel groups within one batch artifact, and (optionally) the user invoking `task-graph-to-batch` repeatedly to produce concurrent batches across waves.

A common pattern:

```
Wave 1: T1.1 ‖ T1.2 ‖ T1.3 (one batch, three parallel agents)
Wave 2: T2.1 ‖ T2.2        (one batch, two parallel agents)
```

This is just `task-graph-to-batch` followed by `batch-to-build` for each wave, with `max_parallel` configured per batch.

## When to Use

Use `heavy` when:

- The task graph has multiple independent slices (different modules, different services)
- Tasks with disjoint write scopes can run concurrently
- Total wall-clock time matters and parallel execution will shorten it

Do NOT use `heavy` when:

- The task graph is mostly sequential (one critical path) — `standard` will be just as fast
- The codebase has many shared files (parallel agents will conflict) — `standard` is safer
- The project is small (overhead of coordinating parallelism > savings) — `light` or `standard`

## Authorization Boundaries

- Pre-`batch-to-build`: planning only. All upstream artifacts written without code changes.
- `batch-to-build`: REQUIRES explicit user authorization for each batch. The user must reauthorize for each new batch — heavy recipe does not auto-authorize subsequent batches. It may upgrade a current dry-run batch only after validating that the task graph and baseline are still current.
- `review-to-acceptance`: REQUIRES user input over the integrated build.

## Notes

### Parallelism rules

`task-graph-to-batch` enforces:
- Within a parallel group: all tasks have disjoint write scopes
- No task in a parallel group touches frozen contracts (per `work/SUBAGENT.md`)
- `max_parallel` defaults to 3; user may override

If write scopes overlap, the offending tasks are placed in a sequential group, not parallel.

### Integrated build

The "integrated build" is just the project state after multiple batches have completed. There is no separate `integrated_build` artifact type. `build-to-review-packet` runs over the current project state regardless of how many batches produced it.

### Run logs

Each batch produces its own `work/build-log-{n}.md`. The review packet may aggregate across these for the acceptance match table.
