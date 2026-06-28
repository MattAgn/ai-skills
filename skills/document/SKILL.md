---
name: document
description: Best practices to write good documentation — code comments, READMEs, and architecture notes. Use when documenting a change, a module, or a non-obvious decision.
---

# Writing good documentation

Document the **why**, not the **what**. The code already says what it does; good documentation captures the reasoning, constraints, and intent a reader can't recover by reading the code.

## When to document

- **Non-obvious decisions** — why this approach over the obvious alternative, what trade-off was accepted.
- **Hacks and workarounds** — why the ugly code exists, what breaks if you "clean it up", and the upstream issue/condition that would let it be removed.
- **Architecture deviations** — when something intentionally departs from the project's normal patterns.
- **Public surfaces** — modules, APIs, and utilities others will call: their contract, inputs/outputs, and gotchas.

**Skip documentation entirely when the code is self-explanatory.** A comment that restates the code is noise — it rots, drifts out of sync, and trains readers to ignore comments. Good naming and structure beat a comment.

## How to write it

- **Be concise.** One or two sentences usually suffice. Long prose goes unread.
- **Stay close to the code.** A comment next to the tricky line beats a wiki page nobody finds. Reserve separate docs for cross-cutting architecture.
- **Write for a junior developer** who is competent but lacks the context you have right now.
- **Keep it current.** If you change the code a comment describes, update or delete the comment in the same change. A wrong comment is worse than none.
- **Prefer examples** for anything with a non-trivial calling convention — a short usage snippet communicates faster than a paragraph.
