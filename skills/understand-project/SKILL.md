---
name: understand-project
description: Ground a feature or refactor in existing code before planning or building — read the relevant conventions, find a similar/target pattern, identify reusable assets, map the touch surface. Internal helper invoked by feat / refactor — not meant to be run on its own.
disable-model-invocation: true
---

Ground every plan and implementation in code that already exists. A plan that says "follows the same pattern as `<module>/`" beats one describing an abstraction in the abstract — the executor gets low cognitive load and a working reference. This is the _method_; the calling skill lists which conventions to check.

## Steps

1. **Read the project conventions** relevant to the change (architecture/structure, state management, data fetching, error handling, styling, test conventions, …). The calling skill gives a **routing table** mapping each topic to the condition that warrants reading it — read about an area only when its condition matches the change at hand; don't bulk-read everything. Each doc costs context, and most tasks touch only a slice. When genuinely unsure an area applies, read it (grounding is the goal); when its area is plainly untouched, skip it.
2. **Find a similar existing implementation / target pattern.** Glob/Grep for a screen, module, component, or refactor that solves a comparable problem or already follows the target shape. Read 1-2 end-to-end so you can point at them by path.
3. **Identify reusable assets** — existing components, hooks, utilities, stores, generated clients. Reuse these by name rather than inventing new ones.
4. **Map the touch surface** — every file, module, route, constant that needs to change. Trace data flow and callers so nothing is missed.
5. **Check third-party libraries** — when a library is involved, fetch version-matched docs before assuming an API.

## Bias

Pick the simplest, cleanest solution: reuse existing patterns/components/helpers, fewest files touched, smallest new surface. If a clever approach and a boring approach reach the same outcome, choose the boring one.
