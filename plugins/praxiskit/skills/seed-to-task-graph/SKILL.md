---
name: seed-to-task-graph
description: "Light recipe: create work/task-graph.md directly from a seed."
---

# Seed to Task Graph

Produce a task graph directly from a rough seed. This transform is the entry point for the **light recipe** — it skips `idea` and `prd` artifacts entirely and gates only on what a task graph minimally needs.

```text
seed -> seed-to-task-graph -> work/task-graph.md + work/SUBAGENT.md -> task-graph-to-batch
```

## Contract

**Inputs:** `seed` (raw user input — text, file path, or pasted prompt)
**Output:** `task_graph` (`work/task-graph.md` + `work/SUBAGENT.md`)
**Schema:** `schemas/task-graph.schema.md` (v3.0) + `schemas/seed.schema.md` (v3.0) for triage
**Preconditions:**
- A seed (text or file) is provided
- Seed contains at least an implicit goal statement (or one is collected via gate)
**Postconditions:**
- `work/task-graph.md` exists with at least one task
- Every task has acceptance criteria with a `[user]`, `[user via gate]`, or `[user via clarify-seed]` source
- No idea.md or PRD.md is produced (this is the explicit point of the light recipe)
- All `forbidden-to-infer` cross-field rules from `idea.schema.md` (e.g., cache_metadata_fields) are honored — gate fires if applicable
**Clarification gate:** fires per `references/clarification-gate.md` for any of:
- Goal statement missing → required gate
- Acceptance signal missing → required gate. If the seed already states done criteria, copy those criteria into task acceptance criteria and do not gate for this field. You may split or quote the user's wording across subtasks, but must not add new acceptance meaning.
- Domain-specific lists triggered by cross-field rules → required gate
**Side effects:**
- Writes `work/task-graph.md` and `work/SUBAGENT.md`
- May write `work/clarify-seed.md` only as an async/bulk fallback (archived after gate completes)
- Updates `work/praxiskit-context.md` recording light-recipe usage
**Stop boundary:** Does NOT execute tasks, write idea.md or PRD.md, or invoke other transforms.

## When to Use This vs `seed-to-idea`

Use `seed-to-task-graph` (light recipe) when:
- The work is a script, prototype, refactor, or bug fix where product framing is overhead
- The user can directly state acceptance criteria without going through idea→PRD
- No external stakeholders need to read an idea or PRD document

Use `seed-to-idea` → `idea-to-prd` (standard recipe) when:
- Multiple users / stakeholders need to align on intent
- Acceptance criteria require derivation from product goals
- The work touches product surface area that benefits from explicit non-goals

The user picks the recipe; this transform does NOT auto-classify.

## Workflow

1. **Load the seed.** If the user provides a path, read it. If they pasted text, use it.
2. **Load schemas.** Read `references/field-state-semantics.md`, `schemas/seed.schema.md`, and `schemas/task-graph.schema.md`.
3. **Triage seed against `seed.schema.md`.** Identify which triage dimensions are present.
4. **Apply cross-field rules** from `idea.schema.md` (distributed → consistency_model; caching with metadata → cache_metadata_fields). Add triggered fields to gaps.
5. **Required gate fields:**
   - `goal_statement` if not derivable from seed
   - `acceptance_signal` only if the seed does not already provide observable done criteria. If present, copy, quote, or split the seed's done criteria into task acceptance criteria and annotate `[user]`; do not add new acceptance meaning.
   - Any forbidden-to-infer field triggered by cross-field rules
6. **Call `references/clarification-gate.md`** with collected gaps.
7. **Decompose into tasks.** Use the goal + acceptance signals to identify minimum viable tasks. Each task gets:
   - Title (inferred from goal)
   - Acceptance criteria (sourced from seed or gate answers; split or quote only when that preserves the user's meaning)
   - Write scope (inferred from repo layout)
   - Dependencies (inferred from task ordering)
8. **Write `work/task-graph.md`** per `schemas/task-graph.schema.md` output format.
9. **Write `work/SUBAGENT.md`** (under 80 lines): goal summary, stack, write scopes per task, reporting convention, and the worker/orchestrator boundary below.
10. **Update `work/praxiskit-context.md`** with task-graph path and recipe = light.
11. **Report** the generated paths. Note that `task-graph-to-batch` is the next step.

## What This Transform Does Not Do

- Does NOT write `work/idea.md` (use `seed-to-idea` for that)
- Does NOT write `work/PRD.md` (use `idea-to-prd` for that)
- Does NOT execute tasks (use `task-graph-to-batch` then `batch-to-build`)
- Does NOT auto-pick a recipe (the user picks; this skill is invoked because they chose light)

## `work/SUBAGENT.md` Boundary Text

Every generated `work/SUBAGENT.md` must include this rule:

```markdown
## Write-Scope Boundary

Spawned workers may modify only their assigned task write scope. They must not update PraxisKit bookkeeping files.

The orchestrator, and only the orchestrator, may update:
- `work/task-graph.md`
- `work/execution-batch-*.md`
- `work/build-log-*.md`
- `work/praxiskit-context.md`
- `work/review.md`
- `work/acceptance.md`
```
