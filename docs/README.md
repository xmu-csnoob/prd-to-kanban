# Demo Assets

This directory holds demo recordings and screenshots for the marketplace listing.

## Demo GIF

Record a short screen capture showing the core workflow:

1. Paste a PRD (or use `examples/sample-prd.md`)
2. Invoke `/prd-to-kanban`
3. Show the generated `work/kanban.md`

Save as `demo.gif` in this directory. Keep it under 15 seconds and loop-friendly.

**Recommended tools:**
- [Kap](https://getkap.co) — open-source, native macOS, exports GIF/WebM
- [LICEcap](http://www.cockos.com/licecap) — simple, Windows/macOS

## PraxisKit Pipeline Demo

For a longer demo of the full PraxisKit chain:

1. Run `seed-to-idea` with a rough concept
2. Run `idea-to-prd` to produce the PRD
3. Run `prd-to-kanban` to generate the Kanban board
4. Run `kanban-to-agents` to show dry-run scheduling
5. Run `build-to-review` to show the acceptance review format

Save as `praxiskit-demo.gif` in this directory.

## Screenshots

Marketplace listings benefit from screenshots showing:
- A generated `work/kanban.md` board (use `examples/sample-kanban.md` as reference)
- The Claude Code plugin install UI
- A `work/SUBAGENT.md` context file
