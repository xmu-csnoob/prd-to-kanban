# Kanban Board: Auth Service Phase 1
  > Source: sample-prd.md · Agents edit this file · 📊 = computed
  > Last updated: 2026-04-28 by prd-to-kanban

## Overview
| Metric | Value |
|--------|-------|
| Phase | 1 |
| Epics | 4 |
| Tasks | 10 |
| Done | 0 |
| Progress | 0% |
| Agents | 3 |
| Scale | M |
| Milestone | M0 — Schema freeze |

## Milestones
| ID | Title | Status |
|----|-------|--------|
| M0 | Schema freeze | [ ] |
| M1 | Auth endpoints live | [ ] |
| M2 | Hardened auth | [ ] |

## Epics

### E0: Contracts [Architect · W0 · M0] 0/2
- [ ] ⚠ T0.1 | Define user schema (email, passwordHash, createdAt) → Types compile; schema rejects missing email, non-string password; duplicate-email uniqueness enforced at DB level
- [ ] T0.2 | Define JWT contract (payload shape, expiry, signing key env var) [T0.1] → Contract doc merged; auth middleware compiles against it

### E1: Auth Endpoints [Backend · W1 · M1] 0/3
- [ ] ⚠ T1.1 | POST /auth/register [T0.1] → 201 + userId on valid input; 409 on duplicate email; 400 on missing fields
- [ ] T1.2 | POST /auth/login [T0.2] → 200 + JWT on valid creds; 401 on wrong password; 404 on unknown email
- [ ] T1.3 | POST /auth/logout — invalidate token server-side [T1.2] → Token rejected after logout; 200 on valid request

### E2: Middleware [Backend · W1 · M1] 0/2
- [ ] ⚠ T2.1 | Auth middleware validates JWT [T0.2] → Valid token passes; expired token returns 401; missing header returns 401
- [ ] T2.2 | Wire middleware to protected routes [T2.1,T1.1] → Integration test: unauthenticated request to protected route returns 401

### E3: Hardening [Security · W2 · M2] 0/3
- [ ] ⚠ T3.1 | Rate-limit auth endpoints [T1.1,T1.2] → Register and login return 429 after 10 req/min per IP
- [ ] T3.2 | Emit audit log on login and logout [T1.2,T1.3] → Structured log entry written for each event; fields: userId, ip, timestamp, outcome
- [ ] T3.3 | Add refresh-token rotation [T0.2,T1.2] → Refresh grants new access token; old refresh token invalidated; replay returns 401

## Dependencies & Parallelism 📊

### Dependency Layers (topological order)
| Layer | Tasks | Depends On | Wave |
|-------|-------|------------|------|
| L0 | T0.1 | — | W0 |
| L1 | T0.2 | L0 | W0 |
| L2 | T1.1, T2.1 | L0..L1 | W1 |
| L3 | T1.2 | L2 | W1 |
| L4 | T1.3, T2.2 | L0..L3 | W1 |
| L5 | T3.1, T3.2, T3.3 | L0..L4 | W2 |

### Executable Batches
| Batch | Tasks | Can Run In Parallel | Why |
|-------|-------|---------------------|-----|
| W0a | T0.1 | No | Root schema — nothing can start without it |
| W0b | T0.2 | No | Depends on T0.1 |
| W1a | T1.1, T2.1 | Yes | Same layer; disjoint write scopes (routes/ vs middleware/) |
| W1b | T1.2 | No | Depends on T1.1 |
| W1c | T1.3, T2.2 | Yes | Same layer; disjoint write scopes |
| W2a | T3.1, T3.2, T3.3 | Yes | Same layer; disjoint write scopes (ratelimit/ vs audit/ vs tokens/) |

### Critical Path
`T0.1 → T0.2 → T1.1 → T1.2 → T1.3 → T3.2`

## Risks & Gates
| Risk | Mitigation | Gate | Status |
|------|------------|------|--------|
| JWT secret rotation breaks live tokens | Document key rotation procedure | M1 | [ ] |
| Refresh token replay after network retry | Implement idempotency window (5 s) | M2 | [ ] |

## Preflight
| Check | Command / Evidence | Baseline Status | Action |
|-------|--------------------|-----------------|--------|
| Tests | `npm test` | unknown | Run before W1 |
| Build | `npm run build` | unknown | Run before W1 |

## Agent Roles
| Role | Model | Strength | Best For |
|------|-------|----------|----------|
| Architect | opus | Schema design, contracts | E0 |
| Backend | sonnet | CRUD, middleware | E1, E2 |
| Security | sonnet | Rate limiting, audit, tokens | E3 |
