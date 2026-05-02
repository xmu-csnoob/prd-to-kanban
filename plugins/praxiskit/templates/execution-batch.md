# Execution Batch {n}

## Source
- Task graph: `work/task-graph.md`
- Task graph fingerprint: `{mtime/hash}`
- Generated: {date}

## Baseline
- Test command: `{cmd}` -> {result}
- Build command: `{cmd}` -> {result}
- Status: pass | fail | unavailable
- Baseline repair: true | false

## Selected Tasks
| ID | Title | Acceptance Criteria | Write Scope | Dependencies | Status At Batch | Parallel Group |
|----|-------|---------------------|-------------|--------------|-----------------|----------------|
| T1.1 | ... | Given..., when..., then... | `path/` | [T0.1] | [ ] | A |

## Parallel Groups
- Group A: [T1.1, T1.2] (disjoint write scopes)
- Sequential: [T1.3]

## Execution Mode
- Mode: subagent-driven | orchestrator-single-task | sequential
- Dispatch expectation: one subagent per task in every 2+ task parallel group; orchestrator owns bookkeeping only

## Authorization
- Mode: dry-run | execute
- Approved by user: yes / no
- Authorization source: decision-ui | chat-confirmation | none
- Approval timestamp: {date or "not approved"}
- Upgraded by batch-to-build: yes / no

## Handoff
Next: `batch-to-build`. It executes only when `Mode: execute`, or when the user explicitly authorizes upgrading this dry-run batch and freshness checks pass.
