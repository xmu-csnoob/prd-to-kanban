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

## Recursive Iteration

PraxisKit treats `work/` as the active working set for the current loop. When `review-to-acceptance` records a decision other than `not_accept_yet`, it archives the completed loop under `work/archive/iterations/{timestamp}-{decision}/`, resets active `work/` to the minimal carry-forward files, and rewrites `work/praxiskit-context.md` with the next entry point.

Use the driver skills when you want the loop advanced for you:

| Driver | What it does | Stops at |
|---|---|---|
| `/praxiskit:next-iteration` | Reads `work/praxiskit-context.md`, chooses the next transform, and runs exactly one step | That transform's normal boundary |
| `/praxiskit:auto-iterate` | Repeats `next-iteration` across multiple waves within a budget | Execution authorization, acceptance input, blocker, failed validation, completion, or budget |

The drivers are orchestration only. They do not auto-authorize source changes, auto-accept reviews, or bypass subagent rules.

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

## Drivers

| Driver | Input | Output |
|---|---|---|
| `/praxiskit:next-iteration` | `work/praxiskit-context.md` or active `work/` artifacts | one selected transform step |
| `/praxiskit:auto-iterate` | current PraxisKit state plus optional budget | repeated safe steps until checkpoint |

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

To execute a dry-run batch, approve the exact batch through the host decision UI or answer a direct yes/no question for that batch. After a loop closes, use `/praxiskit:next-iteration` to continue from `work/praxiskit-context.md`.

For Codex: `codex plugin marketplace add xmu-csnoob/praxiskit`, then open `/plugins` and install `praxiskit`.

## Visual Console

PraxisKit ships with an interactive visual workbench for browsing iterations, task graphs, and review packets.

```bash
git submodule update --init console
cd console && npm install && npm run dev
```

Then open `http://localhost:5173` and select your project's `work/` directory. The console renders task DAGs with React Flow, tracks wave progress, and provides archive access for historical iterations.

> The console is a Git submodule at `console/` → `https://github.com/xmu-csnoob/praxiskit-console`.

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
- `Mode: dry-run` by default

At this point no source files should be modified. Review the batch. If it looks right, explicitly authorize execution:

```text
yes, execute this exact batch
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

Review `work/task-graph.md` and `work/execution-batch-{n}.md`. To run the batch, approve the exact batch through the host decision UI or answer the direct yes/no execution question.

```text
yes, execute this exact batch
```

Then continue:

```text
/praxiskit:batch-to-build
/praxiskit:build-to-review-packet
/praxiskit:review-to-acceptance
```

`review-to-acceptance` records your decision. It does not infer accept/revise/continue by itself.

After acceptance, the loop is archived and active `work/` is reset. Run `/praxiskit:next-iteration` to continue one more wave, or `/praxiskit:auto-iterate` to keep advancing until the next checkpoint.

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
yes, execute this exact batch
/praxiskit:batch-to-build
/praxiskit:build-to-review-packet
```

After review, run `task-graph-to-batch` again for the next unblocked wave. Keep repeating until `review-to-acceptance` closes the work.

You can replace the manual repeat with:

```text
/praxiskit:next-iteration   # one safe step
/praxiskit:auto-iterate     # bounded multi-step loop
```

## Authorization

Planning transforms may write artifacts under `work/`, but they do not modify project source.

`batch-to-build` is the execution boundary. It refuses unless:

- the batch artifact says `authorization: execute`
- the user has explicitly confirmed execution for that exact batch in the current session

Use host-native decision/input when available. Otherwise, answer one direct yes/no question for the exact batch. Words like `continue`, `next`, `advance`, or `proceed` are not execution authorization by themselves.

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

`build-to-review-packet` writes inspectable evidence. `review-to-acceptance` records the user's formal decision, then archives the loop and resets active `work/` unless the decision is `not_accept_yet`.

### `work/archive/iterations/` and `work/praxiskit-context.md`

Completed loops are durable archives. Active `work/` stays small, and `work/praxiskit-context.md` tells `next-iteration` or `auto-iterate` which files to read next.

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
    next-iteration/
    auto-iterate/
```

## License

MIT
