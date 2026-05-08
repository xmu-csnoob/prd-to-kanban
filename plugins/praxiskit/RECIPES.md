# PraxisKit Recipes

A recipe is a named ordered chain of transforms. Recipes are the user-facing entry point; transforms are the implementation primitive.

## How to Invoke

PraxisKit uses **manual chaining**. You (or a thin agent driver) invoke each transform's skill in order.

For each recipe:

1. Read the recipe section below.
2. Note the chain of transforms.
3. Invoke the first transform's skill. Wait for its output artifact and any clarification gate.
4. Once the first transform's stop boundary is reached, invoke the next transform.
5. Continue until the recipe completes (typically with `review-and-accept`).

The driver skill `next-iteration` can handle the chaining:

- Default mode: read `work/praxiskit-context.md`, choose the next transform, run one step, stop at that transform's boundary.
- Budget mode (`budget=N` or `cycles=N`): repeat until execution authorization, acceptance input, clarification, blocker, failed validation, completion, or budget stops the loop.

The driver does not change the recipe algebra — it is an orchestration wrapper around the same transform contracts.

## Authorization Boundaries (apply to every recipe)

- **Planning checkpoint** — after `task_graph` is produced. No code is touched yet.
- **Build checkpoint** — via host-native execution decision or direct yes/no confirmation for the exact batch. The batch may already have `Mode: execute` (carried from `task-graph-to-batch`); otherwise `batch-to-build` upgrades it after validating it is not stale.
- **Acceptance checkpoint** — `review-and-accept` Phase 2 REQUIRES user input; no auto-decide.

## Recursive Iterations

After `review-and-accept` Phase 3 records a decision other than `not_accept_yet`, it archives the completed loop under `work/archive/iterations/{timestamp}-{decision}/` and resets active `work/` to the minimal carry-forward set. This keeps recipes recursive: each iteration starts from a clean active work directory plus a durable archive pointer in `work/praxiskit-context.md`.

The next invocation should read `work/praxiskit-context.md` first:

- Remaining unblocked work → `task-graph-to-batch`
- Current dry-run or authorized batch → `batch-to-build`
- Completed build evidence → `review-and-accept` (Phase 1 produces the review; Phase 2 collects the decision)
- Follow-ups after `revise` → the recipe's task-graph-producing transform
- Completed acceptance with no remaining work → stop

Use `next-iteration` for one hop, or `next-iteration` with a budget for bounded repeated hops.

---

## Recipe: light

> Skip idea/PRD; produce a task graph directly from a seed. For toy projects and prototypes.

### Chain

1. **`seed-to-task-graph`** — `work/task-graph.md` + `work/SUBAGENT.md` directly from seed
2. **`task-graph-to-batch`** — selects the next executable batch into `work/execution-batch-{n}.md`
3. **`batch-to-build`** — executes the authorized batch; produces code + `work/build-log-{n}.md`
4. *(loop steps 2-3 if more batches remain)*
5. **`review-and-accept`** — produces `work/review.md`, collects decision into `work/acceptance.md`, archives the iteration

### When to Use

Use `light` when ANY of these apply:
- Toy projects, scripts, refactors, single-script automations
- The user can state "done = X" directly without a PRD
- No external stakeholders need an idea or PRD
- The work is reversible enough that gate-heavy planning is overhead

Do NOT use `light` when stakeholders need to align on intent, acceptance criteria require product reasoning, or the work has significant parallelizable scope.

### Notes

The light recipe is NOT a "skip the gate" recipe. `seed-to-task-graph` still fires the clarification gate per `references/clarification-gate.md`, honors cross-field rules from `idea.schema.md`, and refuses to fabricate domain-specific lists. **Sparse seeds (1-2 sentences) trigger eager gating** — that's the point of the gate, not its bypass.

Written: `work/seed.md` (if user provides a seed file), `work/task-graph.md`, `work/SUBAGENT.md`, `work/execution-batch-{n}.md`, `work/build-log-{n}.md`, `work/review.md`, `work/acceptance.md`, `work/praxiskit-context.md`.

Not written: `work/idea.md`, `work/PRD.md` (use `standard` if you want one).

---

## Recipe: standard

> Full pipeline from rough seed to acceptance, with idea + PRD documents for traceability. **Default recipe.**

### Chain

1. **`seed-to-idea`** — `work/idea.md`
2. **`idea-to-prd`** — `work/PRD.md` (No-New-Fields Rule applies)
3. **`prd-to-task-graph`** — `work/task-graph.md` + `work/SUBAGENT.md`
4. **`task-graph-to-batch`** — `work/execution-batch-{n}.md`
5. **`batch-to-build`** — executes the authorized batch
6. *(loop steps 4-5 if more batches remain)*
7. **`review-and-accept`** — `work/review.md` + `work/acceptance.md` + archive

### When to Use

Use `standard` when ANY of these apply:
- Multiple users / stakeholders need to align on intent
- Acceptance criteria require derivation from product goals
- The work touches product surface that benefits from explicit non-goals
- A PRD will outlive the implementation (e.g., handoff to another team)

Use `light` for one-offs; `heavy` for parallelizable scope; `from-prd` if a PRD already exists.

### Notes

Each transform that produces a new artifact has its own clarification gate per `references/clarification-gate.md`:
- `seed-to-idea` gates on missing required idea fields and forbidden-to-infer cross-field triggers — **eager on sparse seeds**
- `idea-to-prd` gates on PRD fields that have no `[user]` source in idea.md (No-New-Fields Rule)
- `prd-to-task-graph` does NOT gate but pre-flight validates blocking Open Questions and missing acceptance criteria

If a gate fires, prefer host-native multi-question UI (e.g. `AskUserQuestion` with up to 4 grouped questions). `work/clarify-{stage}.md` is bulk/async fallback only.

Inter-stage carry rules:
- idea.md Open Questions carry forward to PRD's Open Questions if not resolved
- PRD's blocking Open Questions must be resolved before `prd-to-task-graph` runs
- Task-graph blocking conditions (missing acceptance criteria, no milestones) route back to `idea-to-prd`

Written: full v3 artifact set (`idea.md`, `PRD.md`, `task-graph.md`, `SUBAGENT.md`, `execution-batch-{n}.md`, `build-log-{n}.md`, `review.md`, `acceptance.md`, plus any `clarify-*.md`).

---

## Recipe: heavy

> Standard pipeline + multi-batch parallel execution for projects with significant parallelizable scope.

### Chain

Same as `standard`, but `task-graph-to-batch` produces multiple parallel groups within one batch (or multiple batches across waves), and `batch-to-build` dispatches subagents for parallel groups.

### Heavy ≠ New Transforms

Heavy uses the SAME transforms as standard. Parallelism comes from `task-graph-to-batch` placing tasks with disjoint write scopes into parallel groups (`max_parallel` defaults to 3, user-overridable).

```
Wave 1: T1.1 ‖ T1.2 ‖ T1.3 (one batch, three parallel agents)
Wave 2: T2.1 ‖ T2.2        (one batch, two parallel agents)
```

### When to Use

Use `heavy` when:
- The task graph has multiple independent slices (different modules, different services)
- Tasks with disjoint write scopes can run concurrently
- Total wall-clock time matters and parallel execution will shorten it

Do NOT use `heavy` when the task graph is mostly sequential, the codebase has many shared files (parallel agents will conflict), or the project is small.

### Authorization

User must reauthorize each new batch — heavy does not auto-authorize subsequent batches. `batch-to-build` only upgrades a current dry-run batch after validating that the task graph and baseline are still current.

### Parallelism Rules (`task-graph-to-batch` enforces)

- Within a parallel group: all tasks have disjoint write scopes
- No task in a parallel group touches frozen contracts (per `work/SUBAGENT.md`)
- Overlapping write scopes → sequential group, not parallel
- The "integrated build" is just the project state after multiple batches complete; there is no separate `integrated_build` artifact type
- Each batch produces its own `work/build-log-{n}.md`; `review-and-accept` may aggregate across these

---

## Recipe: from-prd

> Skip seed/idea; start from an existing PRD document and run through to acceptance.

### Chain

1. *(prerequisite: `work/PRD.md` exists and follows `schemas/prd.schema.md`; if root `PRD.md` is the only obvious candidate, `prd-to-task-graph` may adopt it into `work/PRD.md`)*
2. **`prd-to-task-graph`** — `work/task-graph.md` + `work/SUBAGENT.md`
3. **`task-graph-to-batch`** — `work/execution-batch-{n}.md`
4. **`batch-to-build`** — executes the authorized batch
5. *(loop steps 3-4 if more batches remain)*
6. **`review-and-accept`** — `work/review.md` + `work/acceptance.md` + archive

### When to Use

Use `from-prd` when:
- A PRD document already exists (from another team, previous project, or external tool)
- The PRD follows or can be massaged to follow `schemas/prd.schema.md`
- Going through `seed-to-idea` → `idea-to-prd` would just re-derive what's already written

Do NOT use `from-prd` when the PRD is incomplete, has unresolved blocking Open Questions, or lacks acceptance criteria for FRs — run `idea-to-prd` first to gate-collect them.

### Notes

`prd-to-task-graph` runs its standard pre-flight checks on the imported PRD:
- All blocking Open Questions resolved
- All FRs have acceptance criteria
- At least 1 milestone defined

If any check fails, the recipe routes back to `idea-to-prd` to fix the PRD before retrying.

This recipe deliberately skips `idea.md`. If the user later wants product framing, they can manually write or extract an idea document — the chain doesn't require it. If the imported PRD doesn't follow `schemas/prd.schema.md` (e.g., a Confluence doc with different headings), the user must restructure it first.

---

## Mode = Recipe, Not Skill Flag

A core principle: **NO transform branches on a `light` / `standard` / `heavy` flag.** If you find yourself wanting to add a mode flag to a transform, the right answer is to write a new recipe (and possibly a new transform) instead.

## Adding a New Recipe

1. Identify the chain of existing transforms that fits your use case.
2. If a needed transform doesn't exist, write a new transform skill following the contract template (`schemas/<artifact>.schema.md` + `skills/<name>/SKILL.md` with a `## Contract` block).
3. Add a section to this file (chain, when-to-use, notes).
