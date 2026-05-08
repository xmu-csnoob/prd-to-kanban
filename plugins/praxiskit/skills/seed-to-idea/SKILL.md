---
name: seed-to-idea
description: "Create work/idea.md from a rough seed; gate missing user-owned fields. Eager-gate on sparse seeds."
---

# Seed to Idea

Turn a rough spark into a short, useful idea brief. **Most seeds are sparse (1-2 sentences) — drift risk is highest here, so gate eagerly.**

```text
seed -> seed-to-idea -> work/idea.md -> idea-to-prd
```

Do not write a PRD, task graph, or implementation plan unless the user explicitly asks to continue.

## Contract

**Inputs:** `seed` (raw user input — text, file, or pasted prompt)
**Output:** `work/idea.md` per `schemas/idea.schema.md` (v2.0); fires `references/clarification-gate.md` for missing user-owned fields
**Preconditions:** seed (text or file path) provided
**Stop boundary:** Does NOT write a PRD, task graph, or any downstream artifact.

## Workflow

1. **Load the seed.** If the user provides a path, read it. If pasted text, use it.
2. **Load gate reference + schema.** Read `references/clarification-gate.md` and `schemas/idea.schema.md`.
3. **Triage seed length.** If seed is < ~200 chars or 1-2 sentences, default to treating most `filled-by-user` fields as gaps. Do NOT draft first.
4. **Map seed to schema fields.** For each schema field:
   - User-sourced value present? → `filled`
   - `inferable` and derivable from a `[user]` field? → `inferable` (must name source)
   - `filled-by-user` or `forbidden-to-infer` with no user source? → `gaps`
5. **Apply cross-field rules** from `idea.schema.md`. Add triggered fields to gaps.
6. **Call clarification-gate** with collected gaps. Prefer host-native multi-question UI (group up to 4 related questions); use chat or `work/clarify-seed.md` only as fallback per gate procedure.
7. **Write `work/idea.md`** only after all gaps are resolved. Annotate every field with its source.
8. **Update `work/praxiskit-context.md`** — `Pending Clarifications: none` if gate passed.
9. **Handoff.** State that `work/idea.md` is ready for `idea-to-prd` if the user wants a PRD.

## Output

Write `work/idea.md` from `templates/idea.md`. Source annotations: `[user]`, `[user via gate]`, `[user via clarify-seed]`, `[inferred from {field}]`, `[default]`.

Do not write `## Assumptions`. Uncertainty lives in `## Open Questions` or inline `[inferred]` annotations.
