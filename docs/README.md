# Demo Assets

This directory holds demo recordings and screenshots for the marketplace listing.

## Demo GIF

Record a short screen capture showing the v3 transform workflow:

1. Pick a recipe from `plugins/praxiskit/recipes/`.
2. For a PRD demo, invoke `/praxiskit:prd-to-task-graph`.
3. Show the generated `work/task-graph.md`.
4. Invoke `/praxiskit:task-graph-to-batch` to show dry-run batch selection.

Save as `demo.gif` in this directory. Keep it under 15 seconds and loop-friendly.

**Recommended tools:**
- [Kap](https://getkap.co) - open-source, native macOS, exports GIF/WebM
- [LICEcap](http://www.cockos.com/licecap) - simple, Windows/macOS

## PraxisKit Pipeline Demo

For a longer demo of the full standard recipe:

1. Run `seed-to-idea` with a rough concept.
2. Run `idea-to-prd` to produce the PRD.
3. Run `prd-to-task-graph` to generate the task graph.
4. Run `task-graph-to-batch` to show dry-run scheduling.
5. Explicitly authorize `batch-to-build` only if the demo includes implementation.
6. Run `build-to-review-packet` to show the review packet.
7. Run `review-to-acceptance` to record the user's decision.

Save as `praxiskit-demo.gif` in this directory.

## Screenshots

Marketplace listings benefit from screenshots showing:
- A generated `work/task-graph.md` graph
- A generated `work/execution-batch-{n}.md` dry-run batch
- The Claude Code plugin install UI
- A `work/SUBAGENT.md` context file
