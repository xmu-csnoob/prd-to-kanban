# Kanban Board: Sparkboard Quick Capture Phase 1
  > Source: sparkboard-prd.md - Agents edit this file - computed sections marked with [computed]
  > Last updated: 2026-04-28 by prd-to-kanban

## Overview
| Metric | Value |
|--------|-------|
| Phase | 1 |
| Epics | 5 |
| Tasks | 14 |
| Done | 0 |
| Progress | 0% |
| Agents | 4 |
| Scale | M |
| Milestone | M1 - Quick capture usable |

## Milestones
| ID | Title | Status |
|----|-------|--------|
| M0 | Contract foundation | [ ] |
| M1 | Quick capture usable | [ ] |
| M2 | Acceptance-ready build | [ ] |

## Epics

### E0: Preflight & Contracts [Architect - W0 - M0] 0/4
- [ ] T0.0 | Restore Vite build baseline -> `npm run build` succeeds from a clean checkout
- [ ] ⚠ T0.1 | Define capture types -> Types compile; invalid status cannot be represented
- [ ] T0.2 | Define reflection helper [T0.1] -> Empty input policy, truncation policy, and confidence labels are tested
- [ ] T0.3 | Define capture status helper [T0.1] -> follow-up answered/skipped states cannot conflict

### E1: Capture UI [Frontend - W1 - M1] 0/3
- [ ] ⚠ T1.1 | Add quick capture composer [T0.1] -> User can enter a messy thought and submit with keyboard or button
- [ ] T1.2 | Render reflection result [T0.2,T1.1] -> System displays summary, suggested follow-up, and confidence label
- [ ] T1.3 | Add lightweight validation states [T1.1] -> Empty thought shows inline error without losing input

### E2: Local State [Frontend - W1 - M1] 0/3
- [ ] T2.1 | Store captured thoughts in app state [T0.1] -> New capture appears in the idea list with `clarifying` status
- [ ] T2.2 | Persist captures locally [T2.1] -> Refresh keeps ideas; storage errors surface non-destructively
- [ ] T2.3 | Promote idea to PRD-ready [T2.1,T0.3] -> Status moves to `planned` only after reflection is accepted

### E3: Review Surface [Frontend - W2 - M2] 0/2
- [ ] T3.1 | Add detail view for captured idea [T1.2,T2.1] -> Detail shows raw seed, reflection, decision, and next step
- [ ] T3.2 | Link to PRD handoff artifact [T3.1,T2.3] -> User can copy a markdown handoff without extra formatting

### E4: Validation & Accessibility [QA - W2 - M2] 0/2
- [ ] ⚠ T4.1 | Add component and state tests [T1.1,T2.1] -> Tests cover submit, error, reflection, and state update paths
- [ ] T4.2 | Run acceptance gates [T0.0,T4.1,T3.2] -> `npm test` and `npm run build` pass; manual keyboard path documented

## Dependencies & Parallelism [computed]

### Dependency Layers (topological order)
| Layer | Tasks | Depends On | Wave |
|-------|-------|------------|------|
| L0 | T0.0, T0.1 | - | W0 |
| L1 | T0.2, T0.3 | L0 | W0 |
| L2 | T1.1, T2.1 | L0..L1 | W1 |
| L3 | T1.2, T1.3, T2.2 | L2 | W1 |
| L4 | T2.3, T3.1, T4.1 | L2..L3 | W1..W2 |
| L5 | T3.2 | L4 | W2 |
| L6 | T4.2 | L0..L5 | W2 |

### Executable Batches
| Batch | Tasks | Can Run In Parallel | Why |
|-------|-------|---------------------|-----|
| W0a | T0.0, T0.1 | Yes | Build baseline and type contract touch disjoint files |
| W0b | T0.2, T0.3 | Yes | Both depend on T0.1 and write disjoint helpers |
| W1a | T1.1, T2.1 | Yes | UI composer and state helper can use frozen contracts independently |
| W1b | T1.2, T1.3, T2.2 | Yes | Same layer with UI display, validation, and persistence split by scope |
| W1c | T2.3 | No | Promotion depends on state and status policy |
| W2a | T3.1, T4.1 | Yes | Detail view and tests can advance in parallel after W1 core behavior |
| W2b | T3.2 | No | Handoff link depends on detail view |
| W2c | T4.2 | No | Final gate depends on all implementation and tests |

### Critical Path
`T0.1 -> T0.2 -> T1.1 -> T1.2 -> T3.1 -> T3.2 -> T4.2`

## Risks & Gates
| Risk | Mitigation | Gate | Status |
|------|------------|------|--------|
| Existing build baseline is broken | Add T0.0 before UI work proceeds | M0 | [ ] |
| Capture flow becomes too form-like | Limit first release to one optional follow-up | M1 | [ ] |
| Local persistence behavior is unclear | Treat localStorage as Phase 1 assumption and document failure behavior | M2 | [ ] |

## Preflight
| Check | Command / Evidence | Baseline Status | Action |
|-------|--------------------|-----------------|--------|
| Tests | `npm test` | unknown | Run before W0 |
| Build | `npm run build` | fail | T0.0 restores missing Vite entrypoint |

## Agent Roles
| Role | Model | Strength | Best For |
|------|-------|----------|----------|
| Architect | gpt-5.5 | Contracts and dependency ordering | E0 |
| Frontend | gpt-5.4 | React UI and state wiring | E1, E2, E3 |
| QA | gpt-5.4-mini | Focused tests and acceptance gates | E4 |
