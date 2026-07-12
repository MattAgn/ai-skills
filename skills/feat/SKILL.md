---
name: feat
description: Build a feature end-to-end — issue-tracker ticket or free-text → plan → implement → merged PR. Attended by default (agent auto-decides whether to challenge the plan; you validate the running app, then review + merge). `--unattended` for cloud/CI (task → draft PR, no human). `--review-commits` to gate each commit; `--auto-merge` to skip the PR review.
argument-hint: [ticketId-or-description] [--unattended] [--review-commits] [--auto-merge]
disable-model-invocation: true
---

Full feature lifecycle orchestrator. Resolve the run mode via the `run-mode` skill (attended by default, `--unattended` for cloud/CI), then apply this skill's checkpoint map. Signpost each phase and print the resolved mode + toggles up front (`run-mode` → "Signpost each phase").

## Mode & checkpoints

This skill's [`run-mode`](../run-mode/SKILL.md) checkpoints map to:

| Checkpoint         | Where                                          | Halts when                          |
| ------------------ | ---------------------------------------------- | ----------------------------------- |
| **Plan** (auto)    | Phase 1 scope + Phase 3 plan                   | agent routes: accept / grill / wayfinder |
| **Commits**        | Phase 5 per-commit                             | `--review-commits`                  |
| **App works**      | Phase 7 before the PR                          | always (attended)                   |
| **PR**             | Phase 7 open + merge                           | default; skipped by `--auto-merge`  |

`--unattended` lifts every human checkpoint: task → draft PR, never asks, never merges (`run-mode` → "`--unattended`"). Autonomous stretches follow the [execution core](../run-mode/SKILL.md#execution-core).

## Phase 1: Understand requirements

1. If `$ARGUMENTS` has a ticket ID, fetch it immediately — title, description, priority, labels, attachments.
2. Without a ticket, treat `$ARGUMENTS` as free-text; if empty, ask for a ticket ID or description (_unattended: empty → stop and report, never ask_).
3. **Parse** the title (module hint from brackets, e.g. `[Auth]` → auth module) and description (user-facing goal, acceptance criteria, edge cases, mockups in attachments).
4. **Identify feature type**: new screen, module, extension of an existing feature, component, settings toggle, etc.
5. **Resolve ambiguity**: ask only question(s) that _materially_ change the output; record minor uncertainties as "Assumptions made". This is the start of the **plan auto-route** (`run-mode` → "Plan"): a fuzzy scope heads toward grilling in Phase 3, a fully-specified one toward accept-as-specified, and a job too big for one session hands off to `/wayfinder` now. _unattended: skip questions; record assumptions and proceed._

## Phase 2: Ground in the codebase

Ground the work in existing code. Don't bulk-read every convention — most features touch only a slice. Read the project's **architecture/structure** conventions by default (they decide module/screen placement).

When genuinely unsure whether an area applies, read it — grounding is the point and a wrong plan costs more than one doc. But skip an area the feature plainly never touches.

Output: a similar feature to point at by path, the reusable assets (components/hooks/utils/stores/generated clients) to reuse by name, and the full touch surface (files, modules, routes, constants).

## Phase 3: Build the plan

Compose the plan from Phases 1-2. Pick the simplest, cleanest solution — reuse existing patterns, fewest files touched, smallest new surface.

Present the plan **commit by commit** with key implementation details, tests in the same commit as the code they cover:

| File | Action      | Description  |
| ---- | ----------- | ------------ |
| path | Create/Edit | What changes |

Propose refactors in the touched area only if the feature needs them. Organise into ordered atomic commits. Then hit the **Plan checkpoint** — route it (`run-mode` → "Plan"): accept-as-specified, `/grill-me`, or `/grill-with-docs`; when grilling, **wait for approval before building**, and on return from the interview continue here — do NOT jump to code. _unattended: skip the route — record open calls as assumptions and proceed._

## Phase 4: Test plan

Define the testing strategy before implementing. Check for missing tests on the touched feature and propose to write them. Present a test-plan table (Unit / Integration / E2E — scenario + file); user confirms. Tests are written in Phase 5 alongside the code, test-first at the agreed seams via `/tdd`. _commits auto / unattended: define the plan and proceed without confirmation._

## Phase 5: Implement & verify

**Precondition**: an approved plan exists (or, unattended, a composed plan) — if you can't point to one, finish Phase 3 first.

Core loop, repeated per commit from the Phase 3 plan:

1. Write code + tests for ONE logical chunk. Read existing tests first and follow their patterns; propose scenarios and implement one at a time (`/tdd`).
2. Run the test suite — must stay green. Snapshot failures from ONLY expected structural changes → update them, verify the diff makes sense, proceed. Tests breaking for other reasons → fix first. Include updated snapshots in the same commit as the code that caused them.
3. **Commits checkpoint** — _`--review-commits`: confirm before committing — even a one-file change — via the `commit` skill's confirmation flow._ _default / unattended: commit directly per the [execution core](../run-mode/SKILL.md#execution-core)._

### Implementation checklist

- [ ] Follow the project's architecture/structure conventions (domain logic, screen/component placement)
- [ ] Reuse existing components, hooks, utilities, and generated clients rather than reinventing
- [ ] Register new screens/routes where the project expects them
- [ ] Validate route params / inputs; handle the not-found / invalid path
- [ ] Follow the project's styling and import conventions

For UI changes: drive the app to the affected screen, observe/screenshot it, and verify visually. Fetch version-matched library docs when needed. _unattended: automated tests only during the loop; the running app is checked at Phase 7._

## Phase 6: Document

Delegate to the `document` skill. Only for hacks, WHY reasoning, architecture deviations, non-trivial decisions. **Skip entirely** if the code is self-explanatory.

## Phase 7: Review, validate & PR

1. **Review (pre-PR)** — run `/code-review` on the current diff. Surface findings; **address criticals** before the PR; note the rest for the user. _unattended: fix criticals yourself, then self-repair per the [execution core](../run-mode/SKILL.md#execution-core)._
2. **App works checkpoint** — validate the running feature (`run-mode` → "App works"): _attended: drive the app / have the user confirm the UI functions — always, before the PR._ _unattended: automated checks only, flag "visual verification required" in the PR._
3. **PR** — ensure the full check suite passes, then use the `open-pr` skill with the `[Feat]` prefix. _attended: open **ready**; the user **reviews the PR**, then rebase on `main` and **merge** (`run-mode` → "PR"). `--auto-merge`: skip the review, open ready → rebase → merge._ _unattended: open a **draft** with the ⚠️ assumptions banner + 🐞 Suspected bugs, and **stop** — never merge._
