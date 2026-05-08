---
name: seed-to-task-graph
description: "Light recipe: create work/task-graph.md directly from a seed. Eager-gate on sparse seeds before decomposing."
---

# Seed to Task Graph

Produce a task graph directly from a rough seed. **Skips `idea` and `prd` artifacts entirely.** Most seeds are 1-2 sentences — the gate fires eagerly precisely because there's no intermediate idea/PRD to absorb the drift.

```text
seed -> seed-to-task-graph -> work/task-graph.md + work/SUBAGENT.md -> task-graph-to-batch
```

## Contract

**Inputs:** `seed` (raw user input — text, file path, or pasted prompt)
**Output:** `work/task-graph.md` + `work/SUBAGENT.md` per `schemas/task-graph.schema.md` (v3.0); fires `references/clarification-gate.md` on missing goal/acceptance/forbidden-to-infer fields
**Preconditions:** seed (text or file) provided; seed contains at least an implicit goal (or one is collected via gate)
**Stop boundary:** Does NOT execute tasks, write idea.md or PRD.md, or invoke other transforms.

## When to Use This vs `seed-to-idea`

Light recipe (this skill) — script, prototype, refactor, bug fix; user can directly state acceptance criteria; no external stakeholders need an idea/PRD.

Standard recipe (`seed-to-idea` → `idea-to-prd`) — multiple stakeholders, acceptance criteria require derivation from product goals, surface area benefits from explicit non-goals.

The user picks the recipe; this transform does NOT auto-classify.

## Workflow

1. **Load the seed.** If user provides a path, read it. If pasted text, use it.
2. **Load gate reference + schemas.** Read `references/clarification-gate.md`, `schemas/seed.schema.md`, and `schemas/task-graph.schema.md`.
3. **Triage seed length.** A 1-2 sentence seed means most goals/acceptance signals are gaps. Apply the Sparse-Input UX Rule from clarification-gate: gate first, draft after.
4. **Triage seed against `seed.schema.md`.** Identify which triage dimensions are present.
5. **Apply cross-field rules** from `idea.schema.md` (distributed → consistency_model; caching with metadata → cache_metadata_fields). Add triggered fields to gaps.
6. **Required gate fields:**
   - `goal_statement` if not derivable from seed
   - `acceptance_signal` only if the seed does not already provide observable done criteria. If present, copy/quote/split the seed's done criteria into task acceptance criteria and annotate `[user]`.
   - Any forbidden-to-infer field triggered by cross-field rules
7. **Call clarification-gate** with collected gaps. Prefer host-native multi-question UI (group up to 4).
8. **Decompose and write artifacts.** Follow `references/decompose-task-graph.md` (steps 1-10). Frozen contracts step is typically skipped on light recipe.
