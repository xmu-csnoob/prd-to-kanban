---
name: clarification-gate
description: "Field states, source annotations, and decision procedure for missing user-owned fields. Strong UX rules for sparse input. Single runtime reference for every gate-firing transform."
type: reference
---

# Clarification Gate

Single runtime reference for every gate-firing transform. Use this to decide which fields go to the user, how to ask, and how to annotate sources in the output document.

## Field States (set per field in each schema)

| State | Meaning | What the agent may do |
|---|---|---|
| `filled-by-user` | Value MUST come from user (seed text or gate answer) | Never infer. If missing → gate. |
| `inferable` | Value MAY be derived from a `[user]` field | Annotate `[inferred from {field}]`. Must name the source field. "I assumed this because seed mentions X" is valid. "This seemed reasonable" is not. |
| `forbidden-to-infer` | Value MUST NOT be written without explicit user input | Never infer. If missing → gate. If gate not possible → blocking Open Question. |

**All list-type fields default to `forbidden-to-infer`** unless the schema explicitly marks them `inferable`. Domain-specific lists (metadata fields, API params, performance targets, consistency guarantees) are never inferred — any inferred list is wrong in ways the user won't notice until implementation.

## Source Annotations (apply to every output field)

- `[user]` — direct from seed/input text
- `[user via gate]` — host-native UI or chat clarification
- `[user via clarify-{stage}]` — bulk async clarify file
- `[inferred from {field}]` — derivable from a `[user]` field
- `[default]` — explicit schema default

## Sparse-Input UX Rule (CRITICAL)

**Most seeds are 1-2 sentences. Most early ideas are sparse. This is exactly when drift hurts most.** Apply these rules without exception:

- A seed of < ~200 chars or 3 sentences means MOST `filled-by-user` fields are gaps. Treat them as gaps unless they trace explicitly to seed text.
- Do NOT draft the artifact first and infer the rest. **Gate first, draft after.**
- Use host-native multi-question UI (e.g. `AskUserQuestion` with up to 4 questions in one prompt). One round of grouped questions beats four rounds of single-question chat.
- Forbidden-to-infer lists never get inferred — gate or block.
- When uncertain whether a value should be `[user]` or `[inferred]`: gate. Inference is silent drift; gating is one extra question.
- When a seed names a domain (e.g. "build a chat app", "add caching") but does not specify the shape, the cross-field rules in the relevant schema (`idea.schema.md`, `seed.schema.md`) trigger. Honor them — they exist precisely because that domain has fields the user must own.

## Decision Procedure

1. If `gaps` is empty → proceed and write the document.
2. Host-native input available → batch related questions (up to 4) in one prompt; pick `choice` / `short_text` / `list` / `free_text` / `multi_field` per gap kind.
3. No structured input AND ≤ 2 shallow gaps → concise chat questions, one at a time.
4. Long lists, nested matrices, many independent fields, or async filling → write `work/clarify-{stage}.md` from `templates/clarify.md`, stop, wait for "continue", parse, archive to `work/clarify-archive/`.

Never instruct the user to type specific command phrases. Treat ambiguous words like "continue", "proceed", "next" as non-authorizing.

## Per-Stage Notes

- `seed` / `idea`: seeds are usually sparse → default to eager gating with grouped host-native questions.
- `prd`: No-New-Fields Rule — every PRD field must trace to idea.md or this run's gate.
- `task-graph`: pre-flight only, no gate firing.
- `acceptance`: the gate IS the user decision. `choice` kind, 4 options, never auto-decide, no default.

## Changelog

- v3.0 (2026-05-08): Consolidated `field-state-semantics.md` into this single runtime reference. Added Sparse-Input UX Rule.
- v2.1 (2026-05-02): Compacted runtime guidance.
- v2.0 (2026-04-29): Initial implementation.
