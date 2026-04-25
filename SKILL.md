---
name: prd-to-kanban
description: "Convert a PRD into a living Kanban board (markdown). PRD to kanban.md via subagent-driven-development execution. Sub-commands: install (auto-configure trigger rules in CLAUDE.md)."
---

# PRD to Kanban Skill

Convert product requirements into a single markdown Kanban board — readable by humans, editable by agents.

**Position in workflow:** PRD → **prd-to-kanban** → `kanban.md` → **subagent-driven-development** → implemented code.

This skill converts a static PRD into a living, agent-editable Kanban board. **You plan — you do not execute.** After generating `kanban.md`, hand off to `subagent-driven-development` immediately. Agents that attempt to implement tasks directly instead of delegating to subagents are not following this workflow.

## Install Sub-Command (`install`)

Invoke with `/prd-to-kanban install [--project]` to auto-configure trigger rules so this skill fires automatically.

**What it does:**
1. Reads the target CLAUDE.md — `~/.claude/CLAUDE.md` (default), or `./CLAUDE.md` (with `--project`)
2. Creates the file if it doesn't exist
3. Checks for an existing `## Auto Mode — Required Skill Triggers` section
4. If the section exists: adds the `/prd-to-kanban` trigger row if missing
5. If the section is missing: appends the full Auto Mode block with both triggers
6. Reports what was added

**Trigger block inserted (standard form):**
```markdown
## Auto Mode — Required Skill Triggers

When you detect any of the following, **invoke the corresponding skill immediately** — do not implement manually, do not ask first:

| Trigger | Skill |
|---------|-------|
| PRD, design doc, feature requirements, or "plan this" / "break this down" / "create a kanban" | `/prd-to-kanban` |

This is **not optional**. If you implement a planning task manually instead of invoking the skill, you are not following this workflow.
```

**Merging behavior:**
- If a `## Auto Mode` section already exists with other triggers, the `/prd-to-kanban` row is inserted — existing rows are preserved
- If the `/prd-to-kanban` trigger row already exists, no change is made (idempotent)
- The `## Auto Mode` heading is normalized to exactly `## Auto Mode — Required Skill Triggers`

**Edge cases:**
- If permissions don't allow writing to the target file, report the error and suggest the user run `! echo '...' >> ~/.claude/CLAUDE.md` manually
- If CLAUDE.md already references `@RTK.md` or another include, preserve it — append after existing content

## Trigger

Use this skill when:
- User provides a PRD, design doc, or feature requirements
- User says "plan this" or "break this down" or "create a kanban"
- Multi-agent or multi-phase development is anticipated
- The task touches more than 3 files or spans multiple modules

**Do NOT use for:**
- Single-file tasks or bug fixes (even with "plan this" language)
- Vague ideas without enough detail to decompose into tasks
- To-do lists with already-well-defined sequential tasks
- Non-implementation work (pure research, pure design without implementation intent)

If input is borderline, ask: "Does this need decomposition into independent tasks with owners and dependencies?" If no, decline politely.

## Input

Accept these forms:
1. Path to a PRD file (`.md`)
2. Raw requirements text pasted by user
3. Reference to an existing planning doc
4. Conversation context alone — infer tasks, epics, and dependencies from the current discussion when no file is available

If input is vague, ask:
- "What is the target repository structure?" (for write-scope inference)
- "Are multiple agents/models involved?" (for role assignment)
- "Is there a frozen contract or schema that must not change?" (for dependency ordering)

## Output

Produce **one file** in the project's work directory:

| File | Purpose | Path |
|------|---------|------|
| **Kanban Board** | Human-readable + agent-editable | `<project>/work/kanban.md` |

**Rule:** This is a single source of truth. Agents edit task statuses. Humans read it in any markdown viewer.

After writing, open the file:
- macOS: `open <path>`
- Linux: `xdg-open <path>`
- Windows: `start <path>`

---

## Output Format (`kanban.md`)

A single markdown file using native markdown syntax. No custom pipe-delimited formats, no HTML.

### Structure

```
# Kanban Board: {project} Phase {phase}
  > Source: {prd-file} · Agents edit this file · 📊 = computed
  > Last updated: {date} by {agent}

## Overview
| Metric | Value |
|--------|-------|
| Phase | {n} |
| Epics | {n}  |
| Tasks | {n}  |
| Done  | {n}  |
| Progress | {n}% |
| Agents | {n}  |
| Milestone | {id} — {title} |

## Milestones
| ID | Title | Status |
|----|-------|--------|
| M0 | ... | [ ] |   ← [x] done, [/] wip, [!] blocked, [ ] pending
| M1 | ... | [/] |   ← Milestones and tasks share the same status system

## Epics
### E0: {title} [owner · {wave} · {milestone}] {done}/{total}
- [{status}] T0.1 | {title} → {ac}          ← status: [x]=done, [ ]=pending, [/]=wip, [!]=blocked
- [{status}] T0.6 | {title} [T0.1,T0.2] → {ac}   ← [deps] after title
- [{status}] ⚠ T3.8 | {title} → {ac}        ← ⚠ = critical path task

### E1: ...
...

## Dependencies & Parallelism 📊

**Two distinct concepts:**
- **Wave (W0, W1, ...)** — Scheduling phases. Tasks in later waves cannot start until all earlier waves complete. Waves are time-based.
- **Layer (L0, L1, ...)** — Topological position. Tasks in the same layer have no transitive dependencies on each other. Layers are structure-based.

Use waves for "when can this run?" and layers for "what can run in parallel?"

### Dependency Layers (topological order)
| Layer | Tasks | Depends On | Wave |
|-------|-------|------------|------|
| L0 | T0.1, T0.2, ... | — | W0 |
| L1 | T1.1, T2.1 | L0 | W0..W1 |
| L2 | T1.3, T2.2 | L0..L1 | W1 |
| L3 | T1.5, T2.3, T3.4 | L0..L2 | W1 |

Tasks in the same layer have no transitive dependencies on each other.
Same layer + different owners + disjoint write scopes → **safe to parallelize**.

### Parallelism Windows (waves)
| Window | Wave | Tasks | Agents | Strategy |
|--------|------|-------|--------|----------|
| 1 | W0 | T0.* | Architect | sequential |
| 2 | W1 | T1.*, T2.*, T3.* | Architect + Implementer | parallel (disjoint scopes) |
| 3 | W2 | T4.* | Implementer | sequential |
| 4 | W3 | T5.* | Validator | sequential |

Tasks in the same wave run concurrently; tasks across waves run sequentially.

### Critical Path
`T0.2 → T3.1 → T3.2 → T3.4 → T5.1 → T5.5 → T5.6`

Longest dependency chain (computed). Any delay on a critical task directly delays
the project. Tasks on this chain are marked with ⚠ in Epics.

## Risks & Gates
| Risk | Mitigation | Gate | Status |
|------|------------|------|--------|
| ...  | ...        | M0   | [ ]    |

## Agent Roles
| Role | Model | Strength | Best For |
|------|-------|----------|----------|
| Architect | ... | ... | ... |
```

### Syntax Rules

- **Status symbols**: `[x]` = done, `[ ]` = pending, `[/]` = in-progress, `[!]` = blocked
- **Blocked tasks** (`[!]`): waiting on external dependency, human decision, or failed attempt. Distinguish from `[ ]` — blocked tasks cannot be started even if resources are available.
- **Critical tasks**: Prefix title with `⚠` (e.g., `⚠ Wire integration`)
- **Inferred tasks**: Prefix title with `~` (e.g., `~ T0.7 | Add error handling`). Inferred tasks are not in the original PRD — add to epic tail, flag in Open Questions. Agents may not remove them without author approval.
- **Dependencies**: In task line after title: `[T0.1,T1.2]`
- **Acceptance criteria**: After `→` at end of task line
- **Owner column**: Short role name — Architect, Implementer, Validator
- **Epic ownership** is a **default** — tasks inherit the epic's owner unless explicitly overridden on a per-task basis. Only override when a specific task needs a different owner
- **Computed fields** marked with 📊 — refresh on regeneration, never edit by hand

### Sections Guide

| Section | Content | Editable? |
|---------|---------|-----------|
| Overview | Phase/epic/task counts, milestone | 📊 computed |
| Milestones | ID, title, status | ✅ Yes |
| Epics | Per-epic task list with deps & AC | ✅ Yes (task status) |
| Dependencies & Parallelism | Layers, windows, critical path | 📊 computed |
| Risks & Gates | Risk, mitigation, gate, status | ✅ Yes |
| Agent Roles | Role, model, strength | ✅ Yes |

### Regenerating Computed Fields

📊 fields (Overview metrics, dependency layers, parallelism windows, critical path) are **computed from task statuses**, not entered manually.

**When to regenerate:**
- After any task status changes (`[ ]` → `[/]` → `[x]`)
- Before starting a new wave of work
- When asked by human or coordinating agent

**How to regenerate:**
1. Read current `kanban.md`
2. Count `[x]` tasks per epic → update `done/total` and Progress %
3. Recalculate critical path (longest chain through dependency graph)
4. Layer assignments are **static** (topology doesn't change with completion status)
5. Write updated Overview and Dependencies & Parallelism sections

**Who regenerates:** In subagent workflows, the orchestrator (main agent) owns all kanban updates — subagents implement and report back, they do not edit the board. The orchestrator regenerates 📊 fields after each batch completes.

---

## Update Protocol (Multi-Agent)

When multiple agents modify `kanban.md`:

1. **Orchestrator owns updates.** The main agent coordinating execution updates kanban — not subagents. Subagents implement and report back; only the orchestrator edits task statuses. This prevents conflicts from concurrent writes.
2. **Never remove or reorder task IDs** — add new tasks at the end of their epic
3. **Computed fields are 📊** — don't edit Overview, layers, or critical path
4. **Locking critical tasks:** change `⚠` to `🔒` when another agent is currently working on a critical task. Decision rule:
   - If you see `🔒` on a task you need: **skip it**, pick a different task, or wait
   - If you see `🔒` on a task you own: you must un-lock it before abandoning work
   - If no tasks are available: **flag to human** — do not force through a locked critical path task

5. **Unblocking `[!]` tasks:** Only the blocking condition's owner (human or external system) clears `[!]` back to `[ ]`. Do not clear another agent's `[!]` flag.

## Task Granularity

A task should be:
- **Completable in one agent session** (1-3 hours of work max)
- **Self-contained** with a clear output (file created, function implemented, test added)
- **Max ~50 lines of code change** across files it touches

If a task would span multiple days or touch 5+ files, **break it down** into smaller tasks. If a task is a single checkbox (e.g., "fix bug X in Y"), it's too small for this board — handle it inline.

**Size heuristics by task type:**

| Task Type | Size Target | Decompose If |
|-----------|-----------|--------------|
| Schema / contract / type | 1-3 files, new types defined | 5+ new types without grouping |
| API / endpoint | 1-3 files per endpoint | Endpoint references 3+ other services |
| UI / component | 1 file per component | Component references 3+ unrelated areas |
| Feature / logic | 2-4 files | Implementation touches 5+ modules |
| Test | 1 test file per feature | Tests require setup in 3+ separate areas |
| Integration / wiring | 1 file | Depends on 5+ other tasks to be ready |
| Docs | 1 doc file per feature area | Doc references 5+ distinct features |

## Definition of Done (DoD)

Acceptance criteria must be **verifiable by an agent** (not subjective). Use this table by task type:

| Task Type | Good AC Pattern | Bad AC Pattern |
|-----------|---------------|----------------|
| Schema / type | "TypeScript types compile; schema validation passes with valid + invalid inputs" | "types are clean" |
| API / endpoint | "GET /foo returns 200 with expected shape; returns 401 unauthenticated" | "endpoint works" |
| UI / component | "Component renders with props X, Y, Z; passes snapshot test; keyboard-navigable" | "looks good" |
| Feature / logic | "Given X, when Y, then Z (unit test covers path)" | "feature works" |
| Test | "Test file runs; N/M assertions cover key paths; no false positives" | "tests added" |
| Integration | "End-to-end flow completes; no unhandled errors in logs" | "integrated" |
| Config / infra | "Service starts; config loaded from env; validation errors surface clearly" | "configured" |

If AC cannot be made concrete, mark task as `Needs Human Approval` and use `~` as AC placeholder. Decision rule: **pause generation and ask human** to either (a) provide concrete AC and re-trigger, or (b) confirm the task is informational and does not need implementation. **In multi-agent runs with no human in the loop:** mark the task `[!]` blocked and continue generation; add the blocked question to an **## Open Questions** section at the bottom of the file.

---

## Execution Rules

1. **Parse the PRD** to extract all tasks, epics, dependencies, and milestones
2. **Apply task granularity checks** — verify each task is completable in one session, <~50 lines change. Break down anything too large; skip anything too small
3. **Write acceptance criteria** using the DoD table — for each task type, use the "Good AC Pattern" from the DoD table. Flag unresolvable ACs as `Needs Human Approval` with `~`
4. **Compute topological layers** from dependency graph — same layer = no transitive deps
5. **Identify the critical path** — longest chain through dependency graph
6. **Assign owners** — epic header owner is a default; individual tasks inherit it unless overridden. Integration/wiring tasks always have a single explicit owner
7. **Determine parallelism opportunities** — same layer + different owners + disjoint write scopes = parallel. Build dependency layers table and parallelism windows table
8. **Generate `kanban.md`** and report the file path. If work is already in progress, set initial statuses to reflect actual state — e.g., a completed task starts as `[x]`, not `[ ]`. Do not reset in-progress work to pending.
9. **Handoff to execution.** After generating `kanban.md`, invoke **`subagent-driven-development`** to dispatch parallel implementers. You are a planning agent — do not implement tasks yourself. Offload execution to subagents immediately after the board is ready.

---

## Design Principles

1. **Frozen interfaces first.** Any task that defines a type/schema/contract consumed by others is Wave 0. No exceptions.
2. **Write scope determines parallelism.** Two tasks can only be parallel if their write scopes are disjoint. Same directory = sequential.
3. **Same layer = parallel opportunity.** Topological layers reveal what can run concurrently. Layer table + parallelism windows make this explicit.
4. **Single owner for integration.** The final "wire everything together" task should have one owner, even if earlier tasks were parallel.
5. **Acceptance criteria must be verifiable.** Prefer "schema validation passes" over "code looks good".
6. **Dependencies flow from contracts, not intuition.** If Task B reads data shaped by Task A, B depends on A — even if they touch different files.
7. **One file, one source of truth.** Everything in `kanban.md`. No HTML. No split formats.
8. **Same file → sequential.** Two agents editing the same file need sequential ordering or coordination.

---

## Execution Notes

- If PRD has **pre-existing dependency info**, preserve it in task-level `[deps]` — do not create a separate dependency section.
- Always suggest **worktree or branch isolation** when parallel agents are involved.

## Open Questions

| Task | Blocked By | Question |
|------|------------|----------|
| — | — | — |

**Resolution:** When a question is answered, move the row to **## Resolved Questions** with the answer inline, or delete the row if no answer is needed. Resolved rows remain in the file as a record.
