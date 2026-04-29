---
name: field-state-semantics
description: "Defines the three field states used in PraxisKit v2 schemas: filled-by-user, inferable, forbidden-to-infer. Referenced by all schema files and the clarification-gate."
type: reference
---

# Field State Semantics

PraxisKit v2 assigns one of three states to every schema field. These states control whether an agent may infer a value, must ask the user, or may write a reasonable default.

## The Three States

### filled-by-user

The value MUST come from the user -- either from the seed/input text directly, or from an answer in the clarification gate. The agent MAY NOT infer, guess, or assume a value for this field.

**When violated:** Agent writes a value without user input -> this is a schema violation. The value must be removed and the field sent to the clarification gate.

**In output documents:** Annotate with `[user]` (direct from seed/input), `[user via gate]` (from AskUserQuestion), or `[user via clarify-{stage}]` (from clarify table).

**Typical fields:** target_user, problem statement, success criteria, any named list of domain-specific items (e.g., metadata field names, API endpoints, acceptance thresholds).

---

### inferable

The agent MAY derive a value from other filled-by-user fields using reasonable inference. The inferred value must be marked so users can see it and override it.

**Constraint:** Inference must be traceable -- the agent must be able to name which filled-by-user field(s) the inference comes from. "I assumed this because the seed mentions X" is valid. "This seemed reasonable" is not.

**In output documents:** Annotate with `[inferred from {source}]`.

**Typical fields:** project name (from problem description), core loop steps (from problem + desired future), tech stack details when one technology implies another.

---

### forbidden-to-infer

The agent MUST NOT write any value for this field unless the user has explicitly provided it. If missing, the field goes to the clarification gate -- never to Assumptions.

**Critical rule: All list-type fields are forbidden-to-infer by default.** Lists of items (metadata fields, API operations, user roles, acceptance criteria values) cannot be reliably inferred -- any inferred list will be wrong in ways the user won't notice until implementation.

**In output documents:** Annotate with `[user]` (same as filled-by-user, since the value came from gate answers).

**Typical fields:** metadata field lists, performance targets, consistency guarantees, any list where the user's domain knowledge determines membership.

---

## Usage in Schema Files

Each schema field table uses these states in the `State` column:

| Field | State | Notes |
|---|---|---|
| target_user | filled-by-user | Must come from seed or gate |
| core_loop | inferable | Derive from problem + desired_future |
| metadata_fields | forbidden-to-infer | Never infer a list of domain fields |

## Usage in SKILL.md Files

Skills reference this file when describing their schema-check step:

> Load `schemas/{stage}.schema.md`. For each field, check state against available input. Collect `filled-by-user` fields with no user source and all `forbidden-to-infer` fields with no user input. Pass these to `references/clarification-gate.md`.

## Changelog

- v2.0 (2026-04-29): Initial definition. Introduced to replace v1's "Assumptions" pattern.
