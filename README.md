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

Use PraxisKit from the root of the project you want to change. PraxisKit writes planning artifacts under `work/` in that project, then stops at explicit checkpoints before touching source code.

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

## Usage Guide

### 1. Pick the Right Recipe

Start by choosing the smallest recipe that still preserves the context you need.

| If you have... | Use | Start with |
|---|---|---|
| A short prompt with clear done criteria | Light | `/praxiskit:seed-to-task-graph` |
| A rough idea that needs shaping | Standard | `/praxiskit:seed-to-idea` |
| A complete PRD already written | From PRD | `/praxiskit:prd-to-task-graph` |
| A large project that should run in waves | Heavy | `/praxiskit:seed-to-idea` |

PraxisKit does not hide mode switches inside a skill. A recipe is just an ordered chain of small transforms, so you can see which artifact is produced at every step.

### 2. Light Recipe: Seed to Dry-Run Batch

Use this for scripts, prototypes, focused bug fixes, and small local tools.

Example request:

```text
Use PraxisKit light recipe. Build a local CLI that turns unchecked Markdown checklist items into GitHub issue draft Markdown. It should ignore checked items, not call the GitHub API, and include a minimal parser/output test.
```

Then invoke:

```text
/praxiskit:seed-to-task-graph
/praxiskit:task-graph-to-batch
```

The first transform creates:

- `work/task-graph.md`: tasks, acceptance criteria, dependencies, write scopes
- `work/SUBAGENT.md`: execution context and worker write-scope boundaries

The second transform creates:

- `work/execution-batch-{n}.md`: the next unblocked set of tasks
- `authorization: dry-run` by default

At this point no source files should be modified. Review the batch. If it looks right, explicitly authorize execution:

```text
execute this batch
```

Then invoke:

```text
/praxiskit:batch-to-build
```

### 3. Standard Recipe: Idea to Acceptance

Use this for most feature work where intent, scope, non-goals, and acceptance criteria matter.

```text
/praxiskit:seed-to-idea
/praxiskit:idea-to-prd
/praxiskit:prd-to-task-graph
/praxiskit:task-graph-to-batch
```

Review `work/task-graph.md` and `work/execution-batch-{n}.md`. To run the batch, say one of the exact execution phrases, for example:

```text
execute this batch
```

Then continue:

```text
/praxiskit:batch-to-build
/praxiskit:build-to-review-packet
/praxiskit:review-to-acceptance
```

`review-to-acceptance` records your decision. It does not infer accept/revise/continue by itself.

### 4. From-PRD Recipe

If `work/PRD.md` already exists and is implementation-ready, skip seed and idea shaping:

```text
/praxiskit:prd-to-task-graph
/praxiskit:task-graph-to-batch
```

Then review the dry-run batch and authorize execution only when ready.

### 5. Heavy Recipe

Heavy uses the same transforms as standard, but repeats batch/build/review cycles. Use it when the task graph has clear waves or disjoint write scopes:

```text
/praxiskit:task-graph-to-batch
execute this batch
/praxiskit:batch-to-build
/praxiskit:build-to-review-packet
```

After review, run `task-graph-to-batch` again for the next unblocked wave. Keep repeating until `review-to-acceptance` closes the work.

## Authorization

Planning transforms may write artifacts under `work/`, but they do not modify project source.

`batch-to-build` is the execution boundary. It refuses unless:

- the batch artifact says `authorization: execute`
- the user has explicitly confirmed execution in the current session

Valid execution phrases are intentionally narrow:

```text
execute this batch
implement this wave
modify code for this batch
run agents for this batch
delegate implementation for this batch
```

Words like `continue`, `next`, `advance`, or `proceed` are not execution authorization.

If a dry-run batch is later authorized, `batch-to-build` first checks that the batch is still fresh: selected tasks, dependencies, write scopes, status, and baseline must still match.

`review-to-acceptance` also requires user input. It never auto-decides.

## Baseline Behavior

`task-graph-to-batch` records the narrowest practical build/test command before producing a batch.

| Baseline status | Meaning | Execution behavior |
|---|---|---|
| `pass` | Existing project baseline is healthy | Normal execution may proceed after authorization |
| `fail` | Existing project baseline is broken | Only a baseline-repair batch may execute |
| `unavailable` | No baseline command was found | Dry-run only; add/select a baseline command before execution |

This keeps PraxisKit from mixing pre-existing failures with new implementation work.

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
