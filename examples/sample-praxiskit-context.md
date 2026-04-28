# PraxisKit Context: Sparkboard Quick Capture

## Source Files
- idea: `work/idea.md`
- prd: `work/PRD.md`
- kanban: `work/kanban.md`
- subagent: `work/SUBAGENT.md`
- latest run: `work/agent-runs/20260428-1513.md`

## Current Milestone
M0 - Contract foundation (partial)

## Canonical Constraints
- `src/types.ts` - defines `CaptureInput`, `CaptureReflection`, and idea status values
- `src/reflection.ts` - generates the first lightweight reflection from raw input
- `src/captureState.ts` - validates raw thought and computes next status
- Keep capture lightweight; avoid heavy project-management forms
- Local-first web app; no cloud sync in Phase 1

## Open Blockers
- T0.0: Vite build baseline is missing `index.html`
- T1.1: Quick Capture UI is not implemented
- T2.2: persistence behavior needs confirmation before release

## Latest Validation
- `rtk npm test` -> pass (2026-04-28)
- `rtk npm run build` -> fail: missing Vite entrypoint (2026-04-28)

## Last Updated By
kanban-to-agents on 2026-04-28
