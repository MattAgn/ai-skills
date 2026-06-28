---
name: understand-project
description: Ground a feature or refactor in existing code before planning or building — read the relevant conventions, find a similar/target pattern, identify reusable assets, map the touch surface. Internal helper invoked by feat / refactor — not meant to be run on its own.
disable-model-invocation: true
---

Ground every plan and implementation in existing code. A plan that points at "the same pattern as `<module>/`" beats one describing an abstraction abstractly — the executor gets a working reference. This is the _method_; the calling skill lists which conventions to check.

## Steps

1. **Read the relevant conventions.** The calling skill provides a **routing table** mapping each topic (architecture, state, data fetching, error handling, styling, tests, …) to the condition that warrants reading it. Read an area only when its condition matches the change — don't bulk-read; each doc costs context and most tasks touch a slice. If unsure an area applies, read it; if plainly untouched, skip it.
2. **Find a similar implementation / target pattern.** Glob/Grep for a module, component, or refactor solving a comparable problem or already in the target shape. Read 1-2 end-to-end so you can cite them by path.
3. **Identify reusable assets** — existing components, hooks, utilities, stores, generated clients. Reuse by name instead of inventing new ones.
4. **Map the touch surface** — every file, module, route, constant that must change. Trace data flow and callers so nothing is missed.
5. **Check third-party libraries** — when one is involved, fetch version-matched docs before assuming an API.

## Bias

Pick the simplest solution: reuse existing patterns/helpers, fewest files touched, smallest new surface. Given equal outcomes, choose the boring approach over the clever one.
