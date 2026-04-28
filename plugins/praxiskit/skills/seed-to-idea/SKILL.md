---
name: seed-to-idea
description: "Turn a raw seed, prompt, rough thought, or vague product/tool/automation concept into work/idea.md. Use when the user has an unshaped idea and needs help clarifying intent before writing a PRD."
---

# Seed to Idea

Turn a rough spark into a short, useful idea brief. This is the first PraxisKit step:

```text
seed -> seed-to-idea -> work/idea.md -> idea-to-prd
```

Do not write a PRD, Kanban board, or implementation plan unless the user explicitly asks to continue to the next step.

## Workflow

1. **Load the seed.** If the user provides a path, read it. If the user pasted text, use it. Do not ask for a seed that was already provided.
2. **Detect product forks.** If the seed explicitly says a high-impact choice is undecided, ask one decision card and stop. High-impact choices include product surface, primary user, first useful version, privacy model, or distribution channel.
3. **Draft before minor questions.** Treat ordinary ambiguity as material for `Assumptions`, not as a reason to block. A rough seed is enough to write a useful `work/idea.md` unless a product fork is present.
4. **Extract the intent.** Identify the user, pain, desired future, core workflow, non-goals, constraints, and why this is worth building.
5. **Mark assumptions.** Do not invent certainty. Put inferred details under `Assumptions` and unresolved items under `Risks & Unknowns`.
6. **Write `work/idea.md`.** Create the file in the target project. If a `work/` directory is missing, create it. Keep small-feature briefs under 120 lines.
7. **Update context index.** Create or update `work/praxiskit-context.md` with source files, canonical constraints, latest decision, and open blockers.
8. **Handoff.** End by stating that `work/idea.md` is ready for the `idea-to-prd` skill if the user wants a formal PRD.

## Clarification UX

Avoid questionnaire dumps. Ask only when there is no seed content, or when one choice would materially change the brief. If the seed says "not sure whether CLI, web, plugin, or repo workflow" or "not sure who the target user is", that is a product fork and should trigger one decision card before writing.

- Ask at most one decision card before writing `work/idea.md`.
- Provide 2-4 concrete options and mark a recommended default.
- Allow "continue" to mean "use the recommended default".
- Put non-blocking gaps into the file instead of asking about them.
- After asking a decision card, stop and wait. Do not write `work/idea.md` in the same turn unless the user already chose an option or explicitly said to continue with defaults.
- If structured choice UI is available, use it. Otherwise use the markdown card format below.

Use this format:

```markdown
One choice would improve the brief:

**Product shape**
- A. Repo workflow that writes `work/idea.md` (Recommended) - lowest friction for PraxisKit.
- B. CLI capture tool - best if this starts outside an editor.
- C. Web app - best if non-developers are primary users.
- D. Other - describe in one phrase.

Say A/B/C/D, or say "continue" and I will assume A.
```

## `work/idea.md` Format

```markdown
# Idea Brief: {name}

## Seed Evidence
- From seed: {direct fact}
- From repo: {direct fact, if any}

## One-Liner
{one sentence}

## Target User
- {primary user}
- {secondary user, if any}

## Problem
{current pain or gap}

## Desired Future
{what should feel different after this exists}

## Core Loop
1. {user action}
2. {system response}
3. {value delivered}

## Early User Stories
- As a {user}, I want {capability}, so that {outcome}.

## Constraints
- {technical, product, time, platform, policy, or integration constraints}

## Assumptions
- {inferred but not confirmed}

## Decisions
| Decision | Choice | Source |
|----------|--------|--------|
| {product fork} | {chosen option} | User / Default / Repo evidence |

## Risks & Unknowns
| Risk / Unknown | Why It Matters | How To Resolve |
|----------------|----------------|----------------|

## Success Signals
- {observable evidence that the idea works}

## PRD Handoff
Recommended next step: invoke the `idea-to-prd` skill.
```

Keep the brief concise. Favor crisp product language over implementation detail.
