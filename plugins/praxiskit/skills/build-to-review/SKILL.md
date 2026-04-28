---
name: build-to-review
description: "Present an implemented build for user acceptance review. Use after kanban-to-agents or implementation work when the user wants to inspect what was built, compare it against the original idea/PRD/Kanban, and decide whether to accept, revise, or continue."
---

# Build to Review

Close the PraxisKit loop by showing the implemented outcome in a form the user can judge.

```text
idea -> PRD -> kanban -> agents -> build-to-review -> acceptance decision
```

This is not an early ideation or prototype skill. Use it after implementation, integration, or a completed Kanban wave. The goal is to make the final result inspectable against the user's original intent.

## Inputs

Load the best available evidence:

- `work/idea.md` - original intent
- `work/PRD.md` - requirements and acceptance criteria
- `work/kanban.md` - planned tasks and completion state
- `work/SUBAGENT.md` - shared execution constraints
- `work/praxiskit-context.md` - compact milestone, blockers, and validation index
- `work/agent-runs/` - execution logs, if present
- Git diff, test output, screenshots, local app URL, or generated artifacts

If the implementation cannot be run or inspected, state that clearly and produce a review packet from static evidence.

## Workflow

1. **Recover the promise.** Summarize what the user originally wanted and what "done" should mean.
2. **Determine review type.** Use Kanban completion and gates to choose `Wave Review`, `Partial Build Review`, `Full Build Review`, or `Regression Review`.
3. **Inspect the result.** Review changed files, completed Kanban tasks, validation logs, and any runnable UI or CLI behavior.
4. **Show the outcome.** Provide the shortest useful artifact: demo script, screenshots/URL summary, command transcript summary, or acceptance packet.
5. **Compare against intent.** Map delivered behavior to PRD goals and Kanban acceptance criteria.
6. **Surface gaps.** List mismatches, incomplete tasks, risks, and follow-up decisions. Map each gap to the next Kanban task when possible.
7. **Ask for acceptance.** End with a concrete scoped decision frame.

## Output Format

Write `work/review.md` unless the user asks only for an inline summary.

```markdown
# {Wave Review | Partial Build Review | Full Build Review | Regression Review}: {project or feature}

## Review Scope
- Type: Wave Review / Partial Build Review / Full Build Review / Regression Review
- Kanban completion:
- User-inspectable artifact:
- Full-product acceptance allowed: Yes / No

## Original Promise
{1-3 sentences from idea/PRD}

## What Is Ready To Inspect
- {artifact, URL, command, screenshot, file, or behavior}

## Cannot Inspect Because
- {missing UI, failed build, missing URL, incomplete wave, or not applicable}

## Demo Path
1. {step the user can take}
2. {expected visible result}
3. {value moment}

## Acceptance Match
| Source | Expectation | Result | Status |
|--------|-------------|--------|--------|
| PRD / Kanban | ... | ... | Pass / Gap / Unknown |

## Evidence
- Validation: `{command}` -> {result}
- Changed areas: `{path}`, `{path}`
- Run logs: `{path}`

## Gaps & Risks
| Gap / Risk | Impact | Kanban Link | Recommended Next Step |
|------------|--------|-------------|-----------------------|

## Decision
- Accept this wave only:
- Revise completed work:
- Continue next Kanban wave:
- Do not accept full product yet:
```

Keep the output user-facing. Do not bury the user in implementation detail; link to files and summarize the evidence that supports acceptance. If Kanban completion is below 100% or release gates are failing, do not present the result as a fully accepted build.
