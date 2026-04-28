# PRD to Kanban

[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code%20Marketplace-Published-green?style=flat-square&logo=anthropic)](https://github.com/xmu-csnoob/prd-to-kanban/blob/main/.claude-plugin/marketplace.json)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](#license)

`prd-to-kanban` turns a PRD, design doc, or feature requirements into two planning artifacts for multi-agent implementation:

- `work/kanban.md` - a living Kanban board with epics, tasks, dependencies, layers, waves, critical path, and progress.
- `work/SUBAGENT.md` - compact shared context that every implementation subagent can read before starting.

This skill is planning-only. It decomposes work and prepares handoff; it does not implement the tasks it creates.

The Claude Code plugin is published in the `xmu-csnoob-tools` marketplace as `prd-to-kanban@xmu-csnoob-tools`.

## Workflow

```text
PRD -> prd-to-kanban -> work/kanban.md + work/SUBAGENT.md -> subagent-driven-development
```

Use it when a feature needs dependency-aware task breakdown, multiple owners, or parallel agent execution. For small single-file bugs or already-sequenced task lists, handle the work directly instead.

## PraxisKit Preview

This repository also contains an experimental `PraxisKit` plugin in `plugins/praxiskit/`. It packages a broader intent-to-acceptance chain:

```text
seed-to-idea -> idea-to-prd -> prd-to-kanban -> kanban-to-agents -> build-to-review
```

The intent is to preserve user intent as it becomes progressively more concrete: idea, PRD, executable Kanban plan, coordinated implementation, and final acceptance review.

| Skill | User Intent | Primary Output | Stops Before |
|-------|-------------|----------------|--------------|
| `seed-to-idea` | "I have a rough seed; help me shape it." | `work/idea.md` | Requirements writing |
| `idea-to-prd` | "Turn this idea into requirements." | `work/PRD.md` | Task decomposition |
| `prd-to-kanban` | "Break this PRD into agent-ready work." | `work/kanban.md`, `work/SUBAGENT.md` | Implementation |
| `kanban-to-agents` | "Execute or coordinate this Kanban plan." | Code changes, updated Kanban, run logs | User acceptance |
| `build-to-review` | "Show me what was built so I can judge it." | `work/review.md` acceptance packet | New feature scope |

`build-to-review` is the final step: it presents the implemented result against the original idea, PRD, and Kanban criteria so the user can accept, revise, or continue. Both the standalone `prd-to-kanban` plugin and the full `praxiskit` plugin are listed in the marketplace catalog — install one or the other, not both (PraxisKit bundles all pipeline steps including prd-to-kanban).

## What It Generates

`work/kanban.md` is the dynamic board. It uses plain markdown checkboxes, task IDs, dependency references, computed dependency layers, parallelism windows, and a critical path.

```markdown
# Kanban Board: Checkout Phase 1
  > Source: checkout-prd.md · Agents edit this file · 📊 = computed
  > Last updated: 2026-04-28 by prd-to-kanban

## Overview
| Metric | Value |
|--------|-------|
| Phase | 1 |
| Epics | 3 |
| Tasks | 8 |
| Done | 0 |
| Progress | 0% |
| Milestone | M0 — Contract freeze |

## Milestones
| ID | Title | Status |
|----|-------|--------|
| M0 | Contract freeze | [ ] |
| M1 | Checkout flow complete | [ ] |

## Epics
### E0: Contracts [Architect · W0 · M0] 0/2
- [ ] ⚠ T0.1 | Define order schema → Types compile; schema accepts valid payloads and rejects invalid payloads
- [ ] T0.2 | Define payment API contract [T0.1] → Contract tests cover success, auth failure, and error shape

### E1: Checkout API [Backend · W1 · M1] 0/3
- [ ] ⚠ T1.1 | Create checkout session endpoint [T0.1,T0.2] → POST returns expected session shape and validation errors
- [ ] T1.2 | Persist checkout state [T0.1] → State transitions are covered by unit tests

## Dependencies & Parallelism 📊

### Dependency Layers (topological order)
| Layer | Tasks | Depends On | Wave |
|-------|-------|------------|------|
| L0 | T0.1 | — | W0 |
| L1 | T0.2 | L0 | W0 |
| L2 | T1.1, T1.2 | L0..L1 | W1 |

### Critical Path
`T0.1 → T0.2 → T1.1`
```

`work/SUBAGENT.md` is static context. It summarizes the project, stack, frozen contracts, write scopes, and reporting convention. The orchestrator passes this path to each subagent so they all start from the same constraints.

## Install

> **PraxisKit vs prd-to-kanban:** PraxisKit bundles all pipeline steps (including prd-to-kanban). Install PraxisKit if you want the full intent-to-acceptance chain; install the standalone prd-to-kanban if you only need the planning step.

### PraxisKit — Claude Code plugin

For local development from this repository:

```bash
claude --plugin-dir plugins/praxiskit
```

For published marketplace installation:

```bash
claude plugin marketplace add xmu-csnoob/prd-to-kanban
claude plugin install praxiskit@xmu-csnoob-tools
```

### PraxisKit — Codex plugin

For local testing from this repository:

```bash
codex plugin marketplace add .
```

For marketplace-style installation after publishing this repository:

```bash
codex plugin marketplace add xmu-csnoob/prd-to-kanban
```

Then open `/plugins`, choose `xmu-csnoob Tools`, and install `praxiskit`.

---

### prd-to-kanban — Claude Code standalone skill

```bash
git clone https://github.com/xmu-csnoob/prd-to-kanban.git ~/.claude/skills/prd-to-kanban
```

Then run once in Claude Code if you want automatic trigger rules:

```text
/prd-to-kanban install
```

Use `/prd-to-kanban install --project` to write trigger rules to the current project's `CLAUDE.md` instead of the global Claude config.

### Claude Code plugin

For local development from this repository:

```bash
claude --plugin-dir plugins/prd-to-kanban
```

For published marketplace installation:

```bash
claude plugin marketplace add xmu-csnoob/prd-to-kanban
claude plugin install prd-to-kanban@xmu-csnoob-tools
```

### Codex standalone skill

```bash
git clone https://github.com/xmu-csnoob/prd-to-kanban.git ~/.codex/skills/prd-to-kanban
```

Restart Codex so the skill list is refreshed. Codex discovers the skill from metadata; it does not need the Claude-only `install` sub-command.

### Codex plugin

For local testing from this repository:

```bash
codex plugin marketplace add .
```

For marketplace-style installation after publishing this repository:

```bash
codex plugin marketplace add xmu-csnoob/prd-to-kanban
```

Then open `/plugins`, choose `xmu-csnoob Tools`, and install `prd-to-kanban`.

## Usage

```text
/prd-to-kanban                         # Claude standalone skill
/prd-to-kanban install                 # Write Claude trigger rules to ~/.claude/CLAUDE.md
/prd-to-kanban install --project       # Write Claude trigger rules to ./CLAUDE.md
/prd-to-kanban:prd-to-kanban           # Claude plugin skill
Use $prd-to-kanban to plan this PRD.   # Codex skill/plugin invocation style
```

Accepted inputs:

- A path to a PRD or design doc.
- Pasted requirements.
- A reference to an existing planning document.
- Conversation context with enough detail to infer tasks and dependencies.

## Repository Layout

```text
SKILL.md                                           # Standalone Claude/Codex skill entry (thin wrapper)
plugins/prd-to-kanban/SKILL.md                    # Shared skill source of truth
plugins/prd-to-kanban/skills/prd-to-kanban/       # Plugin-packaged skill entry (thin wrapper)
plugins/prd-to-kanban/.claude-plugin/             # Claude Code plugin manifest
plugins/prd-to-kanban/.codex-plugin/              # Codex plugin manifest
plugins/praxiskit/skills/seed-to-idea/            # PraxisKit: raw seed → work/idea.md
plugins/praxiskit/skills/idea-to-prd/             # PraxisKit: idea → work/PRD.md
plugins/praxiskit/skills/prd-to-kanban/           # PraxisKit: PRD → Kanban (thin wrapper)
plugins/praxiskit/skills/kanban-to-agents/        # PraxisKit: Kanban → coordinated agent execution
plugins/praxiskit/skills/build-to-review/         # PraxisKit: build → acceptance review
plugins/praxiskit/.claude-plugin/                 # PraxisKit Claude Code plugin manifest
plugins/praxiskit/.codex-plugin/                  # PraxisKit Codex plugin manifest
.claude-plugin/marketplace.json                   # Claude Code marketplace catalog
.agents/plugins/marketplace.json                  # Codex marketplace catalog
examples/sample-prd.md                            # Example PRD input
examples/sample-kanban.md                         # Example kanban.md output
docs/README.md                                    # Demo GIF recording notes
```

When changing prd-to-kanban behavior, edit `plugins/prd-to-kanban/SKILL.md`. The root `SKILL.md` and plugin-packaged `SKILL.md` files are thin entries that point to the shared source.

## License

MIT
