# PraxisKit Context: Auth Service

## Source Files
- idea: `work/idea.md`
- prd: `work/PRD.md`
- kanban: `work/kanban.md`
- subagent: `work/SUBAGENT.md`

## Current Milestone
M0 — Schema freeze (pending)

## Canonical Constraints
- `src/types/user.ts` — user schema (email, passwordHash, createdAt); frozen after T0.1
- `src/types/jwt.ts` — JWT payload shape, expiry, signing key env var; frozen after T0.2

## Open Blockers
- T0.1: not started — all W1 tasks blocked until schema merged

## Latest Validation
- `npm test` → 47 passed, 0 failed (2026-04-28)

## Last Updated By
prd-to-kanban on 2026-04-28
