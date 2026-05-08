<div align="center">

# PraxisKit

**A typed transform pipeline for Claude Code and Codex.**
Stop AI agents from inventing requirements. Every artifact field traces to a user source. Every code change waits for explicit authorization. Every iteration archives cleanly.

[![Claude Code Marketplace](https://img.shields.io/badge/Claude%20Code-Marketplace-0F766E?style=flat-square&logo=anthropic)](https://github.com/xmu-csnoob/praxiskit/blob/main/.claude-plugin/marketplace.json)
[![Codex Marketplace](https://img.shields.io/badge/Codex-Marketplace-1F2937?style=flat-square)](https://github.com/xmu-csnoob/praxiskit/blob/main/.agents/plugins/marketplace.json)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](#license)
[![Stars](https://img.shields.io/github/stars/xmu-csnoob/praxiskit?style=flat-square&logo=github)](https://github.com/xmu-csnoob/praxiskit/stargazers)

```text
seed → idea → PRD → task graph → batch → build → review → acceptance
                                   │
                                   └─ explicit user authorization checkpoint
```

</div>

---

## Why PraxisKit

LLM coding agents fail in predictable ways:

- They draft a "plan" that *looks* complete but invents requirements.
- They infer API params, metadata fields, performance targets the user never authorized.
- They claim "tests pass" without showing evidence.
- A 50-line task balloons into 2,000 lines of slop with no rollback.

PraxisKit replaces "one big prompt" with a chain of small typed transforms. Each step has a schema. Each artifact field carries an annotation: `[user]`, `[user via gate]`, or `[inferred from {field}]`. **No drift, no hallucinated requirements, no mystery commits.**

| Free-form prompting | PraxisKit |
|---|---|
| One mega-prompt; agent decides what's reasonable | Each step has a typed schema; missing fields go to a clarification gate |
| Inferred metadata, fabricated specs | Every field annotated with its source — no mystery values |
| "Looks good, ship it" | Pre-flight refuses if blocking questions or acceptance criteria are missing |
| Code merges without explicit approval | `batch-to-build` refuses unless the user authorized **this exact batch** |
| Vague seeds become silent drift | **Sparse-input UX rule**: gate eagerly, never draft-then-infer |
| One huge run, no recovery | Recursive iterations with archive/reset; each loop starts from a clean `work/` |

## The Sparse-Input UX Rule (the killer feature)

Most seeds are 1-2 sentences. That's exactly when drift hurts most. PraxisKit's clarification gate fires **eagerly** on sparse input:

```text
seed: "build a chat app with caching"
                │
                ▼
  cross-field rules trigger:
    cache_metadata_fields  (forbidden-to-infer)
    consistency_model      (forbidden-to-infer)
                │
                ▼
  AskUserQuestion (host-native, up to 4 grouped questions):
    1. Which cache entries need metadata? Which fields?
    2. Consistency model? (eventual / linearizable / per-session)
    3. Auth required for cache reads?
    4. Eviction policy?
                │
                ▼
  draft work/idea.md only after gaps resolved
  every field annotated [user] or [user via gate]
```

`forbidden-to-infer` lists (API params, metadata fields, performance targets) **never** get inferred — gate or block. **Inference is silent drift; gating is one extra question.**

## 30-Second Start

```bash
# Claude Code
claude plugin marketplace add xmu-csnoob/praxiskit
claude plugin install praxiskit@xmu-csnoob-tools

# Codex
codex plugin marketplace add xmu-csnoob/praxiskit
# then /plugins → install praxiskit
```

From the project root, run a recipe end-to-end:

```text
# Standard recipe — most feature work
/praxiskit:seed-to-idea
/praxiskit:idea-to-prd
/praxiskit:prd-to-task-graph
/praxiskit:task-graph-to-batch        # dry-run batch
yes, execute this exact batch         # explicit authorization
/praxiskit:batch-to-build             # subagent-driven execution
/praxiskit:review-and-accept          # review + decision + archive
```

Or let the driver pick the next safe step:

```text
/praxiskit:next-iteration                # one transform step
/praxiskit:next-iteration budget=20      # bounded multi-step loop
```

## Recipes

A recipe is a named chain of transforms. Pick the smallest one that preserves the context you need.

| Recipe | When | Chain |
|---|---|---|
| **Light** | Scripts, prototypes, refactors with direct done criteria | `seed-to-task-graph → batch → build → accept` |
| **Standard** | Most feature work; needs intent, scope, non-goals, traceability | `seed-to-idea → idea-to-prd → prd-to-task-graph → batch → build → accept` |
| **Heavy** | Multiple disjoint write scopes; wall-clock matters | Standard + parallel groups inside batches |
| **From PRD** | A valid PRD already exists | `prd-to-task-graph → batch → build → accept` |

> **Mode = recipe, not a skill flag.** No transform branches on `light`/`standard`/`heavy`. Different needs get different chains.

Full spec: [`plugins/praxiskit/RECIPES.md`](plugins/praxiskit/RECIPES.md).

## Transforms

Each transform is a Claude Code / Codex skill with a typed `## Contract` (Inputs · Output · Preconditions · Stop boundary). Schemas live in [`plugins/praxiskit/schemas/`](plugins/praxiskit/schemas).

| Skill | Input | Output | Stops before |
|---|---|---|---|
| `/praxiskit:seed-to-idea` | seed | `work/idea.md` | PRD |
| `/praxiskit:idea-to-prd` | `work/idea.md` | `work/PRD.md` | task graph |
| `/praxiskit:seed-to-task-graph` | seed | `work/task-graph.md` + `work/SUBAGENT.md` | execution |
| `/praxiskit:prd-to-task-graph` | `work/PRD.md` | `work/task-graph.md` + `work/SUBAGENT.md` | execution |
| `/praxiskit:task-graph-to-batch` | task graph | `work/execution-batch-{n}.md` (dry-run) | source changes |
| `/praxiskit:batch-to-build` | authorized batch | code + `work/build-log-{n}.md` | acceptance |
| `/praxiskit:review-and-accept` | build evidence | `work/review.md` + `work/acceptance.md` + archive | next loop |
| `/praxiskit:next-iteration` | `work/praxiskit-context.md` | one transform step (or bounded loop) | per transform's boundary |

## Authorization Checkpoints

```text
[planning]            [batch-to-build]            [review-and-accept]
   │                       │                            │
   write artifacts         requires explicit            requires user decision
   under work/             batch authorization          (4 options, no
   no source changes       for THIS exact batch         auto-decide, no default)
```

`batch-to-build` refuses without `Mode: execute`. Ambiguous words like "continue", "next", "advance" are **not** authorization. The user explicitly approves through host-native decision UI or one direct yes/no answer for the exact batch.

If a dry-run batch is later authorized, `batch-to-build` re-validates that the task graph fingerprint, dependencies, write scopes, statuses, and baseline are still fresh. Stale batches refuse to upgrade.

## Recursive Iterations

After `review-and-accept` records a decision other than `not_accept_yet`:

1. **Archive** — completed loop moves to `work/archive/iterations/{timestamp}-{decision}/` with a manifest.
2. **Reset** — active `work/` is recreated with the minimal carry-forward set (e.g. task-graph + SUBAGENT for `continue_next_wave`; follow-ups only for `revise`).
3. **Resume** — `work/praxiskit-context.md` records the next entry point.

```text
loop 1: idea → PRD → graph → batch → build → accept_wave        ┐
loop 2: graph (carried) → batch → build → continue_next_wave    ├─ work/archive/iterations/
loop 3: graph (carried) → batch → build → revise + follow-ups   ┘
loop 4: new graph from follow-ups → ...
```

Active context stays small. History stays auditable. `next-iteration` always knows what to read next from `praxiskit-context.md`.

## Design Principles

- **Typed transforms over free-form prompts.** Each step has a schema; the agent refuses instead of inventing.
- **Sources are explicit.** Every artifact field carries an annotation. No mystery values.
- **Mode = recipe, not flag.** No transform branches on `light`/`standard`/`heavy`. Different needs get different chains.
- **Planning vs execution boundary is hard.** Planning skills never modify project source.
- **User checkpoints, not permission slips.** Host-native choice UI when available; never ask the user to type incantation phrases.
- **Sparse input → eager gate.** Most seeds are vague — that's expected. The gate exists precisely for that case.

## Visual Console (optional)

```bash
git submodule update --init console
cd console && npm install && npm run dev
```

Open `http://localhost:5173` → select your project's `work/` directory. The console renders task DAGs with React Flow, tracks wave progress, and provides archive access for historical iterations.

> Submodule: [`praxiskit-console`](https://github.com/xmu-csnoob/praxiskit-console).

## Repository Layout

```text
plugins/praxiskit/
  RECIPES.md                  # named transform chains
  references/                 # clarification gate (with sparse-input UX rules),
                              # shared boundaries, decompose workflow
  schemas/                    # typed artifact schemas (idea, PRD, task graph,
                              # execution batch, review packet, acceptance, seed)
  templates/                  # artifact skeletons
  skills/
    seed-to-idea/
    idea-to-prd/
    seed-to-task-graph/
    prd-to-task-graph/
    task-graph-to-batch/
    batch-to-build/
    review-and-accept/
    next-iteration/
```

There is also a standalone [`prd-to-kanban`](plugins/prd-to-kanban) plugin for users who only need PRD-to-Kanban planning without the full PraxisKit pipeline.

## Install

### Claude Code

```bash
claude plugin marketplace add xmu-csnoob/praxiskit
claude plugin install praxiskit@xmu-csnoob-tools
```

Local dev:

```bash
claude --plugin-dir plugins/praxiskit
```

### Codex

```bash
codex plugin marketplace add xmu-csnoob/praxiskit
# then /plugins → xmu-csnoob Tools → install praxiskit
```

## License

MIT — free for personal and commercial use.

---

<div align="center">

**If PraxisKit kept your AI agent from drifting, leave a ⭐.**

</div>
