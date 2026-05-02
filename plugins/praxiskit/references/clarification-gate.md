---
name: clarification-gate
description: "Shared PraxisKit decision procedure for missing user-owned fields. Uses host-native decision/input UI first; writes clarify files only for bulk or async fallback."
type: reference
---

# Clarification Gate

Use this gate after mapping available user input to a stage schema. It resolves fields that are `filled-by-user` or `forbidden-to-infer` before writing the downstream artifact.

Clarification is an interaction, not a default file workflow. Prefer the host's structured input surface; use Markdown only when the answer shape is too large or async.

## Input

Pass:

- `stage`: `seed` | `idea` | `prd` | `task-graph` | `acceptance`
- `gaps`: missing user-owned fields, each with:
  - `field`
  - `kind`: `choice` | `short_text` | `list` | `free_text` | `multi_field`
  - `question`
  - `options` for `choice`

## Decision Procedure

1. If `gaps` is empty, proceed and write the document.
2. If host-native input is available, use it:
   - `choice`: single-select or multi-select
   - `short_text`: short freeform input
   - `list`: compact newline or comma-separated input
   - `free_text`: concise paragraph unless the user needs a worksheet
   - `multi_field`: compact structured text or a few focused prompts
3. If no structured input is available and there are at most two shallow gaps, ask concise chat questions and wait.
4. Otherwise use the clarify-file fallback.

Ask no more than three related questions in one interaction if the host supports grouped input; otherwise ask one question at a time. Do not create `work/clarify-*.md` just because a field is missing.

## Clarify-File Fallback

Use this path only for long lists, matrices, nested structures, many independent fields, user-requested async filling, or hosts without adequate interactive input.

1. Write `work/clarify-{stage}.md` from `templates/clarify.md`.
2. Stop. Tell the user to fill `<TBD: ...>` placeholders and say **continue**.
3. Do not write the downstream document in the same turn.

On **continue**:

1. Re-read `work/clarify-{stage}.md`.
2. Find remaining `<TBD:` placeholders.
3. If any remain, list the fields and stop.
4. If none remain:
   - parse answers
   - move the file to `work/clarify-archive/{stage}-{YYYY-MM-DD}-{HHMM}.md`
   - write the downstream document with answers annotated `[user via clarify-{stage}]`

For `stage = acceptance`, the downstream document is `work/acceptance.md`. The decision must always come from the user; there is no default. Prefer interactive choice input unless the user asks for an async decision file.

## Source Annotations

- Direct seed/input: `[user]`
- Host-native input or chat clarification: `[user via gate]`
- Clarify-file fallback: `[user via clarify-{stage}]`

## Changelog

- v2.1 (2026-05-02): Compacted runtime guidance; moved file template to `templates/clarify.md`.
- v2.0 (2026-04-29): Initial implementation.
