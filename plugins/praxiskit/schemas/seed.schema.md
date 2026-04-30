---
schema_name: seed
schema_version: "3.0"
used_by: seed-to-idea, seed-to-task-graph
input_file: work/seed.md (or pasted text)
output_file: none — this schema is informational; describes what makes a seed "ready" to consume
---

# Seed Schema v3.0

A `seed` is the raw user input that initiates a recipe. Unlike downstream artifacts, a seed has no required structure — users provide whatever they have. This schema describes what an *ideal* seed contains, used by `seed-to-idea` and `seed-to-task-graph` to triage gaps.

## Triage Fields

These are the dimensions a producing transform looks for in a seed. Missing dimensions become gate questions.

| Dimension | State | Rationale |
|---|---|---|
| goal_statement | filled-by-user | "What are you trying to do?" — without this, no transform can proceed |
| target_user | filled-by-user | Who benefits — required for `seed-to-idea`; less critical for `seed-to-task-graph` |
| problem_or_pain | filled-by-user | What's broken without this — required for `seed-to-idea`; optional for `seed-to-task-graph` |
| acceptance_signal | filled-by-user | "How will you know this is done?" — required for `seed-to-task-graph` only when the seed does not already include observable done criteria |
| constraints | filled-by-user | Language, platform, timing, integrations |
| domain_specifics | forbidden-to-infer | Any domain list (cache fields, API endpoints, schema columns) — never infer |
| recipe_choice | filled-by-user | Light / standard / heavy — if user did not pick, default to standard |

## Gate Routing

When a seed is consumed by `seed-to-idea`, missing dimensions go to the `idea.schema.md` gate.

When a seed is consumed by `seed-to-task-graph`, missing dimensions go to the `task-graph.schema.md` gate (which also imports `idea.schema.md` cross-field rules for domain-specific lists).

## Cross-Field Rules

Inherited from `idea.schema.md` cross-field rules:

- Distributed system mentions → fire `consistency_model` gate
- Caching system with structured metadata → fire `cache_metadata_fields` gate

These rules apply regardless of which transform consumes the seed.

## What This Schema Does NOT Do

- Does NOT enforce a seed file format. Seeds may be pasted text, file paths, GitHub issues, Linear tickets, or hand-typed prompts.
- Does NOT block a transform from running on a sparse seed. A sparse seed just means more gate fields will fire.

## Changelog

- v3.0 (2026-04-30): Initial schema. Introduced as part of v3's typed-artifact model.
