---
schema_name: idea
schema_version: "2.0"
used_by: seed-to-idea
output_file: work/idea.md
---

# Idea Schema v2.0

Defines required fields for `work/idea.md` and their state (see `references/clarification-gate.md`).

## Required Fields

| Field | State | Kind | Question for Gate | Notes |
|---|---|---|---|---|
| name | inferable | short_text | - | Infer from seed subject |
| target_user | filled-by-user | short_text | "Who are the primary users of this product? Describe their role and what they're trying to do." | At least 1 user role required |
| problem | filled-by-user | short_text | "What specific pain or gap does this solve? What happens today without this product?" | At least 1 sentence |
| desired_future | filled-by-user | short_text | "What should feel different after this exists? What does success look like for a user?" | At least 1 sentence |
| core_loop | inferable | - | - | Derive from problem + desired_future; 3 steps min |
| constraints | filled-by-user | list | "What constraints apply? (e.g., language, platform, timeline, compliance, integrations)" | May be empty list if user confirms none |
| success_signals | filled-by-user | list | "How will you know this is working? List 1-3 observable signals." | At least 1 signal |

## Forbidden-to-Infer Fields

These fields MUST NOT be written unless the user provides them. If present in seed, extract verbatim. If absent, send to gate.

| Field | Kind | Question for Gate |
|---|---|---|
| domain_specific_lists | list | Any list of domain items (e.g., metadata fields, API operations, config keys). Question varies by domain -- ask specifically: "What [items] do you need?" |
| performance_targets | list | "What are your performance requirements? (latency, throughput, scale -- or 'none for now')" |
| integration_systems | list | "Which external systems must this integrate with? (or 'none')" |

## Cross-Field Rules

- If `problem` describes a distributed system, multi-node coordination, data replication, or fault tolerance (e.g., "distributed", "cluster", "replication", "consensus", "multi-region", "failover"): add `consistency_model` to forbidden-to-infer list with question "What consistency model do you need? (e.g., eventual consistency, strong consistency, leader-based — or 'not decided yet')"
- If `problem` describes a caching system where entries have structured metadata — including ML inference caches, attention/KV caches, HTTP caches with headers, session caches, or any cache where entries carry typed fields beyond a simple key-value pair (e.g., "KV-cache", "LLM cache", "inference cache", "attention cache", "transformer serving", "cache with metadata"): add `cache_metadata_fields` to forbidden-to-infer list with question "What metadata fields does a cache entry need? List them (e.g., layer, token_range, dtype — or describe the structure)"
- If `constraints` is empty after gate: mark as `[user confirmed: none]`

## Output Annotation Rules

Every section in `work/idea.md` must annotate field sources:

```markdown
## Target User
- {role} [{source}]

## Problem
{description} [{source}]
```

Source values: `[user]`, `[user via gate]`, `[user via clarify-seed]`, `[inferred from {field}]`, `[default]`

## What Replaces "Assumptions"

v1's `## Assumptions` section is removed. In v2:

- Inferred values appear inline with `[inferred from X]` annotation
- Unknown required values are sent to gate before writing
- Remaining uncertainty goes to `## Open Questions` with `blocking` / `non-blocking` type

## Changelog

- v2.0 (2026-04-29): Initial v2 schema. Replaces v1 Assumptions pattern.
