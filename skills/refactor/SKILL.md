---
name: refactor
description: Refactor safely end-to-end — issue-tracker ticket or free-text → tests-first → small green steps → merged PR. Attended by default (agent auto-decides whether to challenge the plan; you validate no visual change, then review + merge). `--unattended` for cloud/CI (task → draft PR, no human). `--review-commits` to gate each commit; `--auto-merge` to skip the PR review. Core principle: never refactor without tests, never break green.
argument-hint: [area-or-ticketId] [--unattended] [--review-commits] [--auto-merge]
disable-model-invocation: true
---

Safe refactoring orchestrator. Core principle: **never refactor without tests, never break green**. Resolve the run mode via the `run-mode` skill (attended by default, `--unattended` for cloud/CI), then apply this skill's checkpoint map. Signpost each phase and print the resolved mode + toggles up front (`run-mode` → "Signpost each phase").

Inspired by **Martin Fowler** (behaviour-preserving refactoring catalog), **Kent Beck** ("make the change easy, then make the easy change"; red-green-refactor), **Michael Feathers** (characterization tests for legacy code).

**Behaviour preservation is sacred.** Refactoring is _not_ a chance to fix bugs or add features. Default invariant: "same observable behaviour" — if the ticket implies a behaviour change, flag it explicitly.

Phases run in strict order: safety-net tests (Phase 3) → plan (Phase 4) → code (Phase 5). Never skip earlier phases.

## Mode & checkpoints

This skill's [`run-mode`](../run-mode/SKILL.md) checkpoints map to:

| Checkpoint         | Where                                                | Halts when                          |
| ------------------ | ---------------------------------------------------- | ----------------------------------- |
| **Plan** (auto)    | Phase 4 plan                                         | agent routes: accept / grill / wayfinder |
| **Commits**        | Phase 3 char-test commit + Phase 5 step commits      | `--review-commits`                  |
| **App works**      | Phase 6 (confirm no visual change)                   | always (attended)                   |
| **PR**             | Phase 8 open + merge                                 | default; skipped by `--auto-merge`  |

**Safety-net carve-out (never lifted, any mode):** Phase 3 writes characterization tests and proves them green (mutation-smoke) **before any production code**, in strict order. `--unattended` and auto-commits remove the *approval wait* on the test commit — never the tests-first sequence itself. Autonomous stretches follow the [execution core](../run-mode/SKILL.md#execution-core).

> **Commit exception:** characterization tests (Phase 3) get their **own commit BEFORE refactoring begins** — overriding the commit skill's "tests with change" rule. They capture existing behaviour and are the deliverable of Phase 3, not paired with a code change.

## Phase 1: Understand scope

1. If `$ARGUMENTS` has a ticket ID, fetch it from the tracker.
2. Without a ticket, treat `$ARGUMENTS` as an area/description; if empty, ask (_unattended: empty → stop and report, nothing to refactor — never ask_).
3. **Parse** title (module/area hint from brackets) and description (goal, motivation, constraints). If the goal is implicit, infer it and state the inference as an "Assumption made".
4. **Classify the goal**: readability, performance, structure/architecture alignment, deduplication, testability, dependency removal, naming, extracting concerns, etc. Multiple goals is fine — list them. Propose your own refactor ideas in the touched area.
5. **Classify per Fowler** when applicable: Extract Function/Component, Inline, Move, Rename, Replace Conditional with Polymorphism, Simplify Conditional, Encapsulate, etc.
6. **Resolve ambiguity**: ask only the question(s) that _materially_ change the output (intended behaviour change vs pure refactor, scope boundaries, target architecture when multiple are plausible); record minor uncertainties as "Assumptions made". This feeds the **plan auto-route** in Phase 4 — and a job too big for one session (a sprawling architecture migration) hands off to `/wayfinder` now. _unattended: skip scope questions; record assumptions and proceed._

## Phase 2: Ground in the codebase

Ground the work in existing code — read a convention area only when the refactor touches it.
When genuinely unsure whether an area applies, read it — but skip one the refactor plainly never touches.

Output: the target pattern to align with (point at a file that already follows it — "align with `<module>/`"), the full touch surface (every file to edit/move/delete, tracing data flow and callers), and any third-party API to check against version-matched docs.

## Phase 3: Safety net — tests FIRST

**Refactoring without a safety net is reckless.** Check existing coverage **first** (run the relevant tests), then write only the tests you actually need:

1. If coverage is sufficient, confirm it and move on.
2. If insufficient, write **characterization tests** (Feathers) via `/tdd` — capture _current_ behaviour, not ideal behaviour. Read existing tests first and follow their patterns; implement one at a time, verifying each passes. All must pass.
3. **Commits checkpoint** — _`--review-commits`: confirm before committing — even though these are "just tests" (exactly where the gate gets skipped) — via the `commit` skill's confirmation flow._ _default / unattended: commit the char tests directly once they're mutation-smoked green (see below)._
4. **Commit the characterization tests before any refactoring** via the `commit` skill with a `test:` prefix. ONLY test files — zero production code (the commit exception above).
   - **Exception — new pure-function extraction**: when the plan extracts a NEW pure function, its stub (signature + minimal implementation copied from existing logic) may ride in the test commit so tests can import it. Direct copy of existing logic, not new behaviour.

### Mutation-smoke the safety net (before trusting it)

A green test suite proves nothing until you've seen it go red. For the critical behaviours the refactor must preserve, insert a credible bug (invert a condition, change a threshold, drop a branch), confirm the relevant test fails, then **revert the mutation**. Repeat for 3-4 key behaviours. Report as a table: mutation | caught? | action taken. A test that stays green under mutation is not protecting you.

### Testability through pure-function extraction

When logic is coupled to the UI framework (hooks, components), extract the core into a **pure function** taking parameters and returning values; the hook becomes a thin wrapper wiring state to it. Every branch then tests with plain assertions — no rendering harness, no mocks. Often the right _first_ step.

## Phase 4: Build the plan

Compose the plan from Phases 1-3. Pick the simplest, cleanest solution — fewest files moved, smallest behaviour surface, reuse existing patterns.

Break the refactor into **small atomic steps**, each keeping tests green. Present an ordered step table; note a rollback strategy for each medium/high-risk step:

| Step | Refactoring              | Files | Risk   |
| ---- | ------------------------ | ----- | ------ |
| 1    | Extract component X      | path  | Low    |
| 2    | Move to /modules/domain/ | paths | Medium |

Then hit the **Plan checkpoint** — route it (`run-mode` → "Plan"): accept-as-specified, `/grill-me`, or `/grill-with-docs` for architecture decisions worth recording. If you reached planning without a green, committed characterization-test commit, stop and complete Phase 3 first. When grilling, **wait for approval before building**; on return from the interview continue here — do NOT jump to code. _unattended: skip the route — record open calls as assumptions and proceed (the Phase 3 safety-net precondition still holds)._

## Phase 5: Implement — small steps, always green

**Precondition — verify the safety net before touching production code.** The core invariant is _never refactor without tests_. Confirm Phase 3 is done: characterization tests exist, pass, and are in a **test-only commit**. If you can't point to that green test commit, go back and complete Phase 3 now. No green safety net → do not implement.

Core loop (repeat for each step from the Phase 4 plan):

1. Apply ONE refactoring step.
2. Run the test suite — must stay green. Snapshot failures caused ONLY by expected structural changes (no behaviour change) → update them, verify the diff is structural, then proceed. Tests breaking for other reasons → revert the step, analyze, adjust. Include updated snapshots in the same commit as the step that caused them.
3. **Commits checkpoint** — _`--review-commits`: confirm before committing via the `commit` skill's confirmation flow (the commit holds production code + snapshot updates it caused; NOT new test files — those belong to the Phase 3 commit)._ _default / unattended: once the suite is green, commit directly and move on (a step that breaks tests for a non-structural reason is reverted at step 2, never committed red)._

### Checklist

- [ ] Follow the project's architecture/structure conventions
- [ ] Follow the project's styling and import conventions
- [ ] No behaviour changes — refactoring only (if a behaviour change is needed, flag it to the user)
- [ ] Use version-matched library docs when needed

## Phase 6: Verify

Run the full suite. **App works checkpoint** — if the refactor touched UI, drive the app to the affected screen and confirm there is **no** visual change (`run-mode` → "App works"): _attended: always, with the user, before the PR._ _unattended: automated checks only, flag "visual verification required" in the PR._ Check for regressions in related modules.

## Phase 7: Document

Delegate to the `document` skill. Only for architecture deviations, non-obvious decisions, or a needed migration guide. **Skip if self-explanatory** — good refactoring speaks for itself.

## Phase 8: Review & PR

1. **Review (pre-PR)** — run `/code-review` on the current diff. Surface its findings; **address criticals** (especially behaviour drift) before the PR; note the rest for the user. _unattended: fix criticals yourself, then self-repair per the [execution core](../run-mode/SKILL.md#execution-core)._
2. **PR** — ensure the full check suite passes, then use the `open-pr` skill with the `[Refactor]` prefix. _attended: open **ready**; the user **reviews the PR**, then rebase on `main` and **merge** (`run-mode` → "PR"). `--auto-merge`: skip the review, open ready → rebase → merge._ _unattended: open a **draft** with the ⚠️ assumptions banner + 🐞 Suspected bugs, and **stop** — never merge._
