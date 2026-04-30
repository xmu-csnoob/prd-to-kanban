# PraxisKit

**Turn rough intent into shipped code through typed A-to-B transforms and recipes.**

PraxisKit is a Claude Code / Codex plugin for software-development workflows. v3 replaces large integrated skills with small typed transforms. Recipes are named chains of those transforms.

```text
seed -> idea -> PRD -> task graph -> execution batch -> build -> review packet -> acceptance
```

> Modes are recipes, not flags inside a skill. Light, standard, heavy, and from-prd flows reuse the same transform contracts.

[![Claude Code Marketplace](https://img.shields.io/badge/Claude%20Code%20Marketplace-Published-green?style=flat-square&logo=anthropic)](https://github.com/xmu-csnoob/praxiskit/blob/main/.claude-plugin/marketplace.json)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](#license)
[![Stars](https://img.shields.io/github/stars/xmu-csnoob/praxiskit?style=flat-square&logo=github)](https://github.com/xmu-csnoob/praxiskit/stargazers)

## Recipes

| Recipe | Chain | Use when... |
|---|---|---|
| Light | `seed-to-task-graph -> task-graph-to-batch -> batch-to-build -> build-to-review-packet -> review-to-acceptance` | Toy projects, scripts, prototypes, and direct acceptance criteria |
| Standard | `seed-to-idea -> idea-to-prd -> prd-to-task-graph -> task-graph-to-batch -> batch-to-build -> build-to-review-packet -> review-to-acceptance` | Most feature work |
| Heavy | Standard chain with repeated batches and parallel groups | Large projects with disjoint write scopes |
| From PRD | `prd-to-task-graph -> task-graph-to-batch -> batch-to-build -> build-to-review-packet -> review-to-acceptance` | You already have a valid PRD |

Read `plugins/praxiskit/recipes/README.md` and the specific recipe file before starting.

## Transforms

| Transform | Input | Output | Stops before |
|---|---|---|---|
| `/praxiskit:seed-to-idea` | seed | `work/idea.md` | PRD |
| `/praxiskit:idea-to-prd` | `work/idea.md` | `work/PRD.md` | task graph |
| `/praxiskit:seed-to-task-graph` | seed | `work/task-graph.md` + `work/SUBAGENT.md` | execution |
| `/praxiskit:prd-to-task-graph` | `work/PRD.md` | `work/task-graph.md` + `work/SUBAGENT.md` | execution |
| `/praxiskit:task-graph-to-batch` | `work/task-graph.md` | `work/execution-batch-{n}.md` | source changes |
| `/praxiskit:batch-to-build` | authorized batch | code + `work/build-log-{n}.md` | acceptance decision |
| `/praxiskit:build-to-review-packet` | build + task graph/PRD | `work/review.md` | acceptance decision |
| `/praxiskit:review-to-acceptance` | `work/review.md` | `work/acceptance.md` | follow-up execution |

## Quick Start

```bash
# 1. Add marketplace
claude plugin marketplace add xmu-csnoob/praxiskit

# 2. Install the plugin
claude plugin install praxiskit@xmu-csnoob-tools

# 3. Pick a recipe and invoke transforms in order
/praxiskit:seed-to-task-graph   # light recipe start
/praxiskit:task-graph-to-batch  # produces a dry-run batch by default
```

To execute the dry-run batch, explicitly say: `execute this batch`.

For Codex: `codex plugin marketplace add xmu-csnoob/praxiskit`, then open `/plugins` and install `praxiskit`.

## Authorization

Planning transforms may write artifacts under `work/`, but they do not modify project source.

`batch-to-build` is the execution boundary. It refuses unless:

- the batch artifact says `authorization: execute`
- the user has explicitly confirmed execution in the current session

`review-to-acceptance` also requires user input. It never auto-decides.

## What Gets Produced

### `work/task-graph.md`

A dependency-aware graph of tasks with acceptance criteria, write scopes, dependency edges, waves, and parallelism windows.

```markdown
| ID | Title | Status | Acceptance Criteria | Write Scope | Dependencies |
|----|-------|--------|---------------------|-------------|--------------|
| T0.1 | ... | [ ] | Given..., when..., then... | `path/` | - |
```

### `work/execution-batch-{n}.md`

A selected set of unblocked tasks. It includes baseline evidence, parallel groups, sequential tasks, and authorization state.

### `work/SUBAGENT.md`

Shared execution context for agents: project summary, stack, frozen contracts, write scopes, and reporting convention.

### `work/review.md` and `work/acceptance.md`

`build-to-review-packet` writes inspectable evidence. `review-to-acceptance` records the user's formal decision.

## Install

### Claude Code plugin

```bash
claude plugin marketplace add xmu-csnoob/praxiskit
claude plugin install praxiskit@xmu-csnoob-tools
```

### Claude Code local dev

```bash
claude --plugin-dir plugins/praxiskit
```

### Codex plugin

```bash
codex plugin marketplace add xmu-csnoob/praxiskit
# then open /plugins -> xmu-csnoob Tools -> install praxiskit
```

## Repository Layout

```text
plugins/praxiskit/
  recipes/              # named transform chains
  references/           # clarification gate and field-state semantics
  schemas/              # typed artifact schemas
  skills/
    seed-to-idea/
    idea-to-prd/
    seed-to-task-graph/
    prd-to-task-graph/
    task-graph-to-batch/
    batch-to-build/
    build-to-review-packet/
    review-to-acceptance/
```

## License

MIT
