---
name: idea-to-prd
description: "Turn work/idea.md, rough product notes, or a clarified idea into a concise implementation-ready PRD. Use before prd-to-task-graph when requirements need goals, scope, acceptance criteria, constraints, and open questions."
---

# Idea to PRD

Convert an idea brief into a PRD that is specific enough for planning, but not yet a task board.

```text
work/idea.md -> idea-to-prd -> work/PRD.md -> prd-to-task-graph
```

## Contract

**Inputs:** `idea` (`work/idea.md`)
**Output:** `prd` (`work/PRD.md`)
**Schema:** `schemas/prd.schema.md` (v2.0)
**Preconditions:**
- `work/idea.md` exists and follows `schemas/idea.schema.md`
- Every field in idea.md has a source annotation
**Postconditions:**
- Every PRD field has a `[user]`, `[user via *]`, or `[inferred from {field}]` annotation
- No-New-Fields Rule holds: no PRD field exists without a traceable source in idea.md or this run's gate
- All idea.md Open Questions are processed (promoted, carried, or removed)
- Every FR has acceptance criteria in Given/When/Then form
**Clarification gate:** fires per `references/clarification-gate.md` for any PRD field that has no `[user]` or `[user via *]` source in idea.md and is not `inferable`.
**Side effects:**
- Writes `work/PRD.md`
- May write `work/clarify-prd.md` (archived after gate completes)
- Updates `work/praxiskit-context.md`
**Stop boundary:** Does NOT decompose into tasks. Hands off to `prd-to-task-graph`.

## Workflow

1. **Load source context.** Read `work/idea.md`. If `work/praxiskit-context.md` exists, read it first.
2. **Load schema.** Read `references/field-state-semantics.md` and `schemas/prd.schema.md`.
3. **Map idea.md to PRD fields.**
   - For each PRD field, find the matching idea.md field with a `[user]` or `[user via *]` annotation.
   - If a PRD field has no `[user]` source in idea.md -> add to `gaps`.
   - `inferable` PRD fields may be derived from `[user]` idea.md fields -- mark `[inferred from {field}]`.
4. **Process idea.md Open Questions.** For each Open Question in idea.md:
   - Check if the question's subject appears as a `[user]` or `[user via *]` annotated field elsewhere in idea.md. If yes → the question is answered; promote that field to a confirmed PRD field.
   - If no matching `[user]` field exists → the question is still open; carry it to PRD `## Open Questions` with the same `blocking`/`non-blocking` type.
5. **Apply No-New-Fields Rule.** Any field that would appear in the PRD but has no `[user]` or `[user via *]` source in idea.md and is not `inferable` -> add to `gaps`.
6. **Call clarification-gate** (see `references/clarification-gate.md`) for all gaps.
7. **Write `work/PRD.md`** only after all gaps are resolved. Annotate every field with its source.
8. **Update `work/praxiskit-context.md`**.
9. **Stop at product requirements.** Do not decompose into tasks.

## No-New-Fields Rule

**This is the core constraint of idea-to-prd v2.**

The agent MUST NOT write any value in the PRD that cannot be attributed to one of:
- A `[user]` or `[user via *]` annotated field in `work/idea.md`
- An answer collected through the clarification gate in this run
- An `inferable` derivation from the above (marked `[inferred from {field}]`)

**Violation example (v1 behavior, now prohibited):**
> idea.md has: `KV-cache support [user]`
> idea-to-prd writes: `FR4: KV-cache entries must carry model_id, session_id, layer, token_range, tensor_shape, dtype, lifecycle_hints`
>
> This violates the rule. The metadata field list is `forbidden-to-infer`. It must go to the gate.

**Compliant behavior:**
> Gate asks: "What metadata fields does a KV-cache entry need?"
> User answers: "layer, token_range, dtype"
> PRD writes: `FR4: KV-cache entries carry: layer, token_range, dtype [user via clarify-prd]`

## PRD Format

```markdown
# PRD: {project or feature}

## Summary
{2-4 sentences} [inferred from problem + desired_future]

## Goals
- {measurable outcome} [{source}]

## Repo Baseline
| Item | Evidence | Notes |
|------|----------|-------|
| Platform / stack | `{file}` | |
| Test command | `{command}` | Declared / inferred / unknown |
| Build command | `{command}` | Declared / inferred / unknown |

## Non-Goals
- {explicitly out of scope} [{source}]

## Users & Use Cases
| User | Need | Success Looks Like | Source |
|------|------|--------------------|--------|
| {user} | {need} | {success} | [{source}] |

## Functional Requirements
| ID | Requirement | Priority | Validation | Acceptance Criteria | Source |
|----|-------------|----------|------------|---------------------|--------|
| FR1 | ... | Must | unit | Given ..., when ..., then ... | [user] |

## Non-Functional Requirements
- Performance: [{source}]
- Reliability: [{source}]
- Security / Privacy: [{source}]

## UX / Workflow
1. {step} [inferred from core_loop]

## Edge Cases
- {case and expected behavior} [{source}]

## Milestones
| ID | Title | Outcome | Source |
|----|-------|---------|--------|

## Open Questions
| Question | Owner | Blocks | Type | Source |
|----------|-------|--------|------|--------|
| ... | User | implementation / release / future | blocking / non-blocking | [carried from idea.md] |

## Task Graph Handoff
Recommended next step: invoke the `prd-to-task-graph` skill.
```

Acceptance criteria must be observable. No vague criteria like "works well". Translate to concrete Given/When/Then behavior.
