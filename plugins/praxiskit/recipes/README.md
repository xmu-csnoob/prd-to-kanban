# PraxisKit Recipes

A recipe is a named ordered chain of transforms. Recipes are the user-facing entry point; transforms are the implementation primitive.

## How to Invoke a Recipe

PraxisKit v3.0 uses **manual chaining** (Decision D1 in the v3 design). The user (or a thin agent driver) invokes each transform's skill in order.

For each recipe, the procedure is:

1. Read this directory's recipe file for the recipe you want (e.g., `recipes/standard.md`).
2. Note the chain of transforms.
3. Invoke the first transform's skill. Wait for its output artifact and any clarification gate.
4. Once the first transform's stop boundary is reached, invoke the next transform.
5. Continue until the recipe completes (typically with `review-to-acceptance`).

## Authorization Boundaries

Every recipe has natural pauses where the user must explicitly authorize the next step:

- **Planning checkpoint** — after `task_graph` is produced. No code is touched yet.
- **Build checkpoint** — after an explicit execution phrase in the current session. The batch may already have `authorization: execute`, or `batch-to-build` may upgrade a current dry-run batch after validating that it is not stale.
- **Acceptance checkpoint** — `review-to-acceptance` REQUIRES user input; no auto-decide.

Recipe documents list these checkpoints explicitly.

## Available Recipes

| Recipe | When to Use |
|---|---|
| [light](./light.md) | Toy projects, scripts, prototypes; skip product/engineering specs |
| [standard](./standard.md) | Most features; full traceability from intent to delivery |
| [heavy](./heavy.md) | Large projects with parallelism opportunities |
| [from-prd](./from-prd.md) | Existing PRD documents, no need to re-derive intent |

## Mode = Recipe, Not Skill Flag

A core v3 principle: NO transform branches on a `light`/`standard`/`heavy` flag. If you find yourself wanting to add a mode flag to a transform, the right answer is to write a new recipe (and possibly a new transform) instead.

## Adding a New Recipe

1. Identify the chain of existing transforms that fits your use case
2. If a needed transform doesn't exist, write a new transform skill following the contract template (`schemas/<artifact>.schema.md` + `skills/<name>/SKILL.md` with a `## Contract` block)
3. Write a `recipes/<name>.md` file using the template below
4. Update this README with a link

## Recipe File Template

```markdown
---
recipe_name: {name}
recipe_version: "3.0"
description: "{one-line summary}"
---

# Recipe: {name}

## Chain

1. `transform-name-1`
2. `transform-name-2`
...

## When to Use

{Description of which scenarios fit this recipe.}

## Authorization Boundaries

- Pre-`{first execution transform}`: planning only
- `{execution transform}`: requires explicit user authorization
- `{decision transform}`: requires user input

## Notes

{Recipe-specific guidance.}
```
