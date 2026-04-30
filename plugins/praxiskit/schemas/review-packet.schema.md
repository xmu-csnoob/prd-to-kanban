---
schema_name: review-packet
schema_version: "3.0"
used_by: build-to-review-packet, review-to-acceptance
input_files: build, plus any of idea/PRD/task-graph
output_file: work/review.md
---

# Review Packet Schema v3.0

A `review_packet` is the inspectable artifact a user reads to decide acceptance. It maps delivered behavior to upstream acceptance criteria with concrete evidence.

## Required Sections

| Section | State | Notes |
|---|---|---|
| review_scope | filled-by-user (recipe-derived) | Wave / Partial Build / Full Build / Regression |
| original_promise | inferable | 1-3 sentences from idea/PRD |
| ready_to_inspect | inferable | Artifacts, URLs, commands, or files the user can examine |
| cannot_inspect_because | inferable | Missing UI, failed build, incomplete wave, or N/A |
| demo_path | inferable | Step-by-step user actions and expected results |
| acceptance_match | filled-by-user (sources) / inferable (mapping) | Table mapping PRD/task-graph criteria to Pass/Gap/Unknown |
| evidence | inferable | Validation commands, changed files, run log paths |
| gaps_and_risks | filled-by-user (where listed in source) / inferable (mapping) | Table linking gaps to follow-up tasks |
| decision_options | constant | Always: accept wave / revise / continue next wave / not yet |

## Required Fields per Acceptance Match Row

```
| Source | Expectation | Result | Status | Evidence |
|--------|-------------|--------|--------|----------|
```

- **Source**: e.g., `PRD FR4`, `task-graph T1.2`, `idea Open Question 3`
- **Expectation**: copy verbatim from upstream artifact
- **Result**: what the build actually does
- **Status**: Pass / Gap / Unknown (must be one of these three)
- **Evidence**: validation command output, file path, screenshot, or "manual inspection: {note}"

## What This Schema Does Not Do

- Does NOT permit fabricated evidence. Every "Pass" must have a concrete pointer to where the evidence lives.
- Does NOT permit "Pass with caveats" — caveats become a Gap row.
- Does NOT make the acceptance decision; that lives in `schemas/acceptance-decision.schema.md`.

## Output Format (work/review.md)

```markdown
# {Wave Review | Partial Build Review | Full Build Review | Regression Review}: {project}

## Review Scope
- Type: {one of the four}
- Task graph completion: {N/M}
- User-inspectable artifact: {yes / no}
- Full-product acceptance allowed: {yes / no}

## Original Promise
{1-3 sentences from idea/PRD}

## What Is Ready To Inspect
- {artifact, URL, command, screenshot, file, or behavior}

## Cannot Inspect Because
- {reason or "N/A"}

## Demo Path
1. {step}
2. {expected result}
3. {value moment}

## Acceptance Match
| Source | Expectation | Result | Status | Evidence |
|--------|-------------|--------|--------|----------|

## Evidence
- Validation: `{command}` -> {result}
- Changed areas: `{path}`, `{path}`
- Run logs: `{path}`

## Gaps & Risks
| Gap / Risk | Impact | Task Graph Link | Recommended Next Step |
|------------|--------|-----------------|-----------------------|

## Decision Hand-Off
Next: `review-to-acceptance` for the formal accept/revise/continue decision.
```

## Changelog

- v3.0 (2026-04-30): Formalized from v2-M build-to-review's free-form output.
