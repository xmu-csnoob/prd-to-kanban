---
name: idea-to-prd
description: "Create work/PRD.md from work/idea.md. Enforces No-New-Fields Rule; gates eagerly when idea.md is sparse."
---

# Idea to PRD

Convert an idea brief into a PRD that is specific enough for planning, but not yet a task board. **If idea.md is sparse, drift risk transfers to the PRD — gate every PRD field that lacks a `[user]` source in idea.md.**

```text
work/idea.md -> idea-to-prd -> work/PRD.md -> prd-to-task-graph
```

## Contract

**Inputs:** `work/idea.md` (must follow `schemas/idea.schema.md`, every field annotated)
**Output:** `work/PRD.md` per `schemas/prd.schema.md` (v2.0); fires `references/clarification-gate.md` for any PRD field that lacks a `[user]` or `[user via *]` source in idea.md and is not `inferable`
**Preconditions:** `work/idea.md` exists; every idea.md field has a source annotation
**Stop boundary:** Does NOT decompose into tasks. Hands off to `prd-to-task-graph`.

## Workflow

1. **Load source context.** Read `work/idea.md`. If `work/praxiskit-context.md` exists, read it first.
2. **Load gate reference + schema.** Read `references/clarification-gate.md` and `schemas/prd.schema.md`.
3. **Map idea.md to PRD fields.**
   - Each PRD field → matching idea.md field with `[user]` or `[user via *]`.
   - No `[user]` source in idea.md → add to `gaps`.
   - `inferable` PRD fields may be derived from `[user]` idea.md fields → mark `[inferred from {field}]`.
4. **Process idea.md Open Questions.** For each:
   - Subject already has a `[user]` annotated field elsewhere in idea.md → answered; promote to confirmed PRD field.
   - No matching `[user]` field → carry to PRD `## Open Questions` with the same `blocking` / `non-blocking` type.
5. **Apply No-New-Fields Rule** (see below). Any field that would appear in PRD with no `[user]` source AND not `inferable` → add to `gaps`.
6. **Call clarification-gate** for all gaps. Prefer host-native multi-question UI (up to 4 grouped questions per round) — especially when many fields are gaps at once.
7. **Write `work/PRD.md`** only after all gaps are resolved. Annotate every field.
8. **Update `work/praxiskit-context.md`**.
9. **Stop at product requirements.** Do not decompose into tasks.

## No-New-Fields Rule

**Core constraint of idea-to-prd.** No PRD value may appear unless it is:
- A `[user]` or `[user via *]` annotated field in `work/idea.md`,
- An answer collected through the clarification gate in this run, OR
- An `inferable` derivation from the above (marked `[inferred from {field}]`).

`forbidden-to-infer` lists (API params, metadata field names, performance targets, consistency guarantees) MUST NOT be inferred — they become blocking Open Questions if the user cannot be reached.

## Output

Write `work/PRD.md` from `templates/prd.md`.

Acceptance criteria must be observable. Do not use vague criteria like "works well"; translate them to concrete Given/When/Then behavior.
