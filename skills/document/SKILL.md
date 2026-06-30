---
name: document
description: Best practices to write good documentation — code comments, READMEs, and architecture notes. Use when documenting a change, a module, or a non-obvious decision.
disable-model-invocation: true
---

# Writing good documentation

Document the **why**, not the **what**. Code already says what it does; capture the reasoning, constraints, and intent a reader can't recover from the code.

## When to document

- **Non-obvious decisions** — why this approach over the obvious one, what trade-off was accepted.
- **Hacks and workarounds** — why the ugly code exists, what breaks if "cleaned up", and the upstream condition that would let it go.
- **Architecture deviations** — when something intentionally departs from the project's normal patterns.
- **Public surfaces** — modules/APIs/utilities others call: their contract, inputs/outputs, and gotchas.

**Skip it when the code is self-explanatory.** A comment that restates the code is noise — it rots, drifts out of sync, and trains readers to ignore comments. Good naming and structure beat a comment.

## How to write it

- **Be concise.** One or two sentences usually suffice; long prose goes unread.
- **Stay close to the code.** A comment by the tricky line beats a wiki page nobody finds; reserve separate docs for cross-cutting architecture.
- **Write for a competent junior** who lacks the context you have right now.
- **Keep it current.** Update or delete a comment in the same change as the code it describes — a wrong comment is worse than none.
- **Prefer examples** for any non-trivial calling convention; a short usage snippet beats a paragraph.
