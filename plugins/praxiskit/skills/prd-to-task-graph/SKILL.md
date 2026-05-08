---
name: prd-to-task-graph
description: "Create work/task-graph.md and work/SUBAGENT.md from work/PRD.md. Pre-flight-validated; planning only."
---

# PRD to Task Graph

```text
work/PRD.md -> prd-to-task-graph -> work/task-graph.md + work/SUBAGENT.md -> task-graph-to-batch
```

## Contract

**Inputs:** `work/PRD.md` (must follow `schemas/prd.schema.md`)
**Output:** `work/task-graph.md` + `work/SUBAGENT.md` per `schemas/task-graph.schema.md` (v3.0); does NOT fire clarification gate (pre-flight-validated)
**Preconditions:** `work/PRD.md` exists; all blocking Open Questions resolved; every FR has acceptance criteria; ≥ 1 milestone defined
**Stop boundary:** Does NOT execute tasks, spawn agents, or modify project source code. Hands off to `task-graph-to-batch`.

## Bundled Resources

`schemas/`, `templates/`, and `references/` paths refer to plugin-bundled files. Do not copy them into the project unless the user explicitly asks. Project writes from this transform: `work/PRD.md` (only when adopting an existing PRD), `work/task-graph.md`, `work/SUBAGENT.md`, `work/praxiskit-context.md`.

## PRD Adoption

If `work/PRD.md` is missing but exactly one obvious project PRD exists (e.g., root `PRD.md`), copy it to `work/PRD.md`, record `PRD source: root PRD.md` in `work/praxiskit-context.md`, and continue normal pre-flight. This is planning artifact adoption, not a source change.

If multiple candidate PRDs exist, ask with host-native choice input. If no candidate exists, stop and route to `seed-to-idea`, `idea-to-prd`, or the `from-prd` recipe setup.

## Pre-flight: Blocking Check

Load bundled `schemas/task-graph.schema.md` and run the **PRD-sourced pre-flight checks** table. For each failing check, output the condition name, what you found, and the schema's remediation steps. Do not write `work/task-graph.md` until all checks pass.

## Main Workflow

After pre-flight passes:

1. **Load `schemas/task-graph.schema.md`** for task field rules.
2. **Decompose and write artifacts.** Follow `references/decompose-task-graph.md` (steps 1-10). Wave 0 carries frozen contracts and schema definitions.
