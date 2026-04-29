---
schema_name: prd
schema_version: "2.0"
used_by: idea-to-prd
input_file: work/idea.md
output_file: work/PRD.md
---

# PRD Schema v2.0

Defines required fields for `work/PRD.md` and their state. idea-to-prd must map every PRD field to either an idea.md source or a gate answer -- never to agent inference.

## Required Fields

| Field | State | Source Allowed | Kind | Question for Gate |
|---|---|---|---|---|
| summary | inferable | idea.md: problem + desired_future | - | Infer: 2-4 sentences combining problem and desired future |
| goals | filled-by-user | idea.md: success_signals | list | "What are the measurable outcomes for this release?" |
| non_goals | filled-by-user | idea.md or gate | list | "What is explicitly out of scope for this release?" |
| users_and_use_cases | filled-by-user | idea.md: target_user | list | "For each user type, what do they need and what does success look like?" |
| functional_requirements | filled-by-user | idea.md: core_loop + constraints | list | See FR rules below |
| milestones | inferable | idea.md: core_loop | - | Infer from core loop phases; 2-4 milestones |

## FR Field Rules

Each functional requirement (FR) row has these sub-fields:

| Sub-field | State | Notes |
|---|---|---|
| FR.requirement | filled-by-user | Must trace to idea.md user story or gate answer |
| FR.priority | inferable | Infer from user story criticality |
| FR.acceptance_criteria | inferable | Draft from requirement; mark `[inferred]`; user can override |
| FR.metadata_fields (any list) | forbidden-to-infer | e.g., cache field names, API params -- must come from user |

## Forbidden-to-Infer Fields

These fields MUST NOT appear in PRD unless the user provided them in idea.md or gate:

| Field | Kind | Gate Question |
|---|---|---|
| Any domain-specific list not in idea.md | list | "The PRD needs [field]. Please list the items." |
| Performance targets not in idea.md | list | "What performance targets apply to this PRD?" |
| Consistency/durability guarantees | short_text | "What consistency and durability guarantees should this provide?" |

## No-New-Fields Rule

**idea-to-prd MUST NOT introduce any field that is not:**
1. Present in `work/idea.md` (with `[user]` or `[user via *]` annotation), OR
2. Collected through the clarification gate in this run

If the agent cannot point to a user source for a PRD field, the field goes to the gate. If it cannot be gated (e.g., it's a `forbidden-to-infer` list and the user hasn't provided it), it becomes a `blocking` Open Question.

## Assumptions -> Open Questions

idea.md in v2 has no `## Assumptions` section. But if the agent discovers an implicit assumption when writing the PRD, it must:

1. NOT silently write it as a fact
2. Mark it as an `Open Question` with type `blocking` or `non-blocking`
3. If `blocking`: stop and gate before continuing

## Output Annotation Rules

Every FR row must include a `Source` column:

```markdown
| ID | Requirement | Priority | Validation | Acceptance Criteria | Source |
|---|---|---|---|---|---|
| FR1 | ... | Must | unit | Given..., when..., then... | [user] |
| FR4 | ... | Must | unit | Given..., when..., then... | [user via clarify-prd] |
```

## Changelog

- v2.0 (2026-04-29): Initial v2 schema. Added No-New-Fields Rule and forbidden-to-infer FR sub-fields.
