---
name: seed-to-idea
description: "Create work/idea.md from a rough seed; gate missing user-owned fields."
---

# Seed to Idea

Turn a rough spark into a short, useful idea brief. This is the first PraxisKit step:

```text
seed -> seed-to-idea -> work/idea.md -> idea-to-prd
```

Do not write a PRD, task graph, or implementation plan unless the user explicitly asks to continue to the next step.

## Contract

**Inputs:** `seed` (raw user input — text, file, or pasted prompt)
**Output:** `idea` (`work/idea.md`)
**Schema:** `schemas/idea.schema.md` (v2.0)
**Preconditions:**
- A seed (text or file path) is provided
**Postconditions:**
- Every field in `work/idea.md` has a source annotation (`[user]`, `[user via gate]`, `[user via clarify-seed]`, `[inferred from {field}]`, or `[default]`)
- No `## Assumptions` section exists; uncertainty lives in `## Open Questions` or inline `[inferred]`
- All `filled-by-user` and `forbidden-to-infer` schema fields trace to a user source
**Clarification gate:** fires per `references/clarification-gate.md` when required or forbidden-to-infer fields lack a user source. Cross-field rules in `idea.schema.md` may add gate fields based on seed content.
**Side effects:**
- Writes `work/idea.md`
- May write `work/clarify-seed.md` only as an async/bulk fallback (archived to `work/clarify-archive/` after gate completes)
- Updates `work/praxiskit-context.md`
**Stop boundary:** Does NOT write a PRD, task graph, or any downstream artifact. Hands off to `idea-to-prd` only when user requests.

## Workflow

1. **Load the seed.** If the user provides a path, read it. If the user pasted text, use it.
2. **Load schema.** Read `references/field-state-semantics.md` and `schemas/idea.schema.md`.
3. **Map seed to schema fields.** For each field in the schema, determine:
   - Is a user-sourced value present? -> mark `filled`
   - Is it `inferable` and can be derived from filled fields? -> mark `inferable`
   - Is it `filled-by-user` or `forbidden-to-infer` with no user source? -> add to `gaps`
4. **Apply cross-field rules** from `schemas/idea.schema.md`. Add any triggered fields to `gaps`.
5. **Call clarification-gate** (see `references/clarification-gate.md`):
   - 0 gaps -> proceed directly
   - Prefer host-native structured input for missing fields (`AskUserQuestion` / decision UI in Claude Code, Codex equivalent when available)
   - Use chat questions if no structured input tool is exposed
   - Write `work/clarify-seed.md` only for bulk/async clarification or long nested answers
6. **Write `work/idea.md`** only after all gaps are resolved. Annotate every field with its source.
7. **Update `work/praxiskit-context.md`** -- add `Pending Clarifications: none` if gate passed.
8. **Handoff.** State that `work/idea.md` is ready for `idea-to-prd` if the user wants a PRD.

## Removed Patterns (v1 -> v2)

These v1 behaviors are **explicitly removed**:

- Drafting before resolving schema gaps -- gaps go to the gate, not to Assumptions
- Limiting clarification to a single decision card -- ask as many as gaps require
- `## Assumptions` section -- replaced by `[inferred from X]` inline annotations and `## Open Questions`

## Output

Write `work/idea.md` from `templates/idea.md`.

Source annotations: `[user]`, `[user via gate]`, `[user via clarify-seed]`, `[inferred from {field}]`, `[default]`.

Do not write `## Assumptions`. Uncertainty lives in `## Open Questions` or inline `[inferred]` annotations.
