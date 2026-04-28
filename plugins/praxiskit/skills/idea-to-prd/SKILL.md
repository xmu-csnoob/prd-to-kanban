---
name: idea-to-prd
description: "Turn work/idea.md, rough product notes, or a clarified idea into a concise implementation-ready PRD. Use before prd-to-kanban when requirements need goals, scope, acceptance criteria, constraints, and open questions."
---

# Idea to PRD

Convert an idea brief into a PRD that is specific enough for planning, but not yet a task board.

```text
work/idea.md -> idea-to-prd -> work/PRD.md -> prd-to-kanban
```

## Workflow

1. **Load source context.** Prefer `work/idea.md`; otherwise use the user's pasted idea or current conversation.
2. **Inspect the target repo lightly.** Read README/config files only as needed to infer platform, stack, scripts, and existing constraints. If `work/praxiskit-context.md` exists, read it first.
3. **Detect scope forks.** If the source explicitly leaves product surface, primary user, first release scope, privacy model, or distribution channel undecided, ask one decision card and stop.
4. **Draft before minor questions.** Infer a coherent first PRD from the idea brief. Use `Assumptions` and `Open Questions` for gaps instead of blocking on perfect information.
5. **Write `work/PRD.md`.** If a PRD path is supplied, use it. Otherwise use `work/PRD.md`. Keep small-feature PRDs under 180 lines.
6. **Update context index.** Refresh `work/praxiskit-context.md` with PRD path, canonical constraints, open blockers, and known validation gaps.
7. **Stop at product requirements.** Do not decompose into tasks; hand off to the `prd-to-kanban` skill when the user wants planning.

## Clarification UX

Avoid broad question lists. Ask only when the answer would change product scope, target user, or first milestone enough that drafting would be misleading.

- Ask at most one decision card before writing the PRD.
- Provide 2-4 concrete options and mark a recommended default.
- Allow "continue" to mean "use the recommended default".
- Put non-blocking gaps into `Assumptions` or `Open Questions`.
- After asking a decision card, stop and wait. Do not write `work/PRD.md` in the same turn unless the user already chose an option or explicitly said to continue with defaults.
- If structured choice UI is available, use it. Otherwise use the markdown card format below.

Use this format:

```markdown
One decision would change the PRD:

**First release scope**
- A. Personal workflow inside one repo (Recommended) - validates the core loop fastest.
- B. Reusable Codex/Claude plugin - better for distribution early.
- C. Web dashboard - better if non-developers are the first users.
- D. Other - describe in one phrase.

Say A/B/C/D, or say "continue" and I will assume A.
```

## PRD Format

```markdown
# PRD: {project or feature}

## Summary
{2-4 sentences}

## Goals
- {measurable outcome}

## Repo Baseline
| Item | Evidence | Notes |
|------|----------|-------|
| Platform / stack | `{file}` | |
| Test command | `{command}` | Declared / inferred / unknown |
| Build command | `{command}` | Declared / inferred / unknown |
| Known baseline gaps | `{path or command}` | Blocks implementation / release / future |

## Non-Goals
- {explicitly out of scope}

## Users & Use Cases
| User | Need | Success Looks Like |
|------|------|--------------------|

## Functional Requirements
| ID | Requirement | Priority | Validation Layer | Acceptance Criteria |
|----|-------------|----------|------------------|---------------------|
| FR1 | ... | Must | unit / component / integration / manual UI / build gate | Given ..., when ..., then ... |

## Non-Functional Requirements
- Performance:
- Reliability:
- Security / Privacy:
- Accessibility:

## UX / Workflow
1. {step}
2. {step}

## Data, Contracts, and Integrations
- `{path or external system}` - {expected role or contract}

## Edge Cases
- {case and expected behavior}

## Milestones
| ID | Title | Outcome |
|----|-------|---------|

## Assumptions
- {inferred detail}

## Open Questions
| Question | Owner | Blocks | Type |
|----------|-------|--------|------|
| ... | ... | implementation / release / future | blocking / non-blocking |

## Kanban Handoff
Recommended next step: invoke the `prd-to-kanban` skill.
```

Acceptance criteria must be observable by an implementation agent. Avoid vague criteria such as "works well" or "feels nice"; translate them into concrete behavior.
