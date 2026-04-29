---
schema_name: kanban
schema_version: "2.0"
used_by: prd-to-kanban
input_file: work/PRD.md
output_files: work/kanban.md, work/SUBAGENT.md
---

# Kanban Schema v2.0

Unlike idea and PRD schemas, kanban schema is used for **pre-flight validation only** -- it does not trigger a clarification gate. prd-to-kanban must refuse to proceed if blocking conditions are not met.

## Pre-flight Blocking Conditions

prd-to-kanban checks these conditions before decomposing tasks. Any `blocking` condition -> refuse and output remediation steps.

| Condition | Check | Remediation |
|---|---|---|
| No blocking Open Questions in PRD | Scan `## Open Questions` table for rows where `Blocks` column = `implementation` or `release` and status is not resolved | List the blocking questions; instruct user to resolve them via `idea-to-prd` or the clarification gate before retrying |
| All FRs have acceptance criteria | Check every FR row for non-empty `Acceptance Criteria` | List FRs without criteria; instruct user to add them |
| At least 1 milestone defined | Check `## Milestones` section | Instruct user to define milestones in PRD |

## Task Field Rules

Each kanban task row has these required fields:

| Field | State | Notes |
|---|---|---|
| task_id | inferable | Sequential T{wave}.{n} format |
| title | inferable | Derive from FR or milestone |
| acceptance_criteria | filled-by-user | Must trace to PRD FR acceptance criteria -- do not rewrite |
| write_scope | inferable | Files/dirs this task will modify |
| dependencies | inferable | Other task IDs this task depends on |

## What to Do With Non-Blocking Open Questions

Non-blocking Open Questions in the PRD may proceed to kanban. Add a corresponding task:

```
T{n} | Resolve: {question summary} | Owner: User | Blocks: {dependent task IDs}
```

## Changelog

- v2.0 (2026-04-29): Initial v2 schema. Adds pre-flight blocking checks.
