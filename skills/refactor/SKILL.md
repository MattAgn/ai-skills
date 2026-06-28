---
name: refactor
description: Refactor safely end-to-end (issue-tracker ticket or free-text → tests-first → small green steps → PR), OR plan one read-only with `--investigate`. Add `--auto` to run unattended (never asks; build stops at a draft PR, still tests-first). Core principle: never refactor without tests, never break green.
argument-hint: [area-or-ticketId] [--investigate] [--auto]
disable-model-invocation: true
---

Safe refactoring orchestrator. Core principle: **never refactor without tests, never break green**. One flow of phases; the mode only changes how a few phases behave (tagged inline).

Inspired by **Martin Fowler** (behaviour-preserving refactoring catalog), **Kent Beck** ("make the change easy, then make the easy change"; red-green-refactor), **Michael Feathers** (characterization tests for legacy code).

**Behaviour preservation is sacred.** Refactoring is _not_ a chance to fix bugs or add features. Default invariant: "same observable behaviour" — if the ticket implies a behaviour change, flag it explicitly.

## Mode selection

1. **Flags**: scan `$ARGUMENTS` for `--investigate` and `--auto`; strip them out — the rest is the ticket ID / area.
2. **Resolve the mode** — the two flags are independent:

   | Flags                  | Mode                        | Behaviour                                                                                          |
   | ---------------------- | --------------------------- | -------------------------------------------------------------------------------------------------- |
   | _(none)_               | **interactive build**       | full refactor, human in the loop (default)                                                         |
   | `--auto`               | **autonomous build**        | full refactor, unattended → stops at a draft PR (see [Autonomous build](#autonomous-build---auto)) |
   | `--investigate`        | **interactive investigate** | read-only plan, human in the loop                                                                  |
   | `--investigate --auto` | **autonomous investigate**  | read-only plan, unattended                                                                         |

3. **Which phases run** (decided by `--investigate` alone; `--auto` only changes _how_ phases behave, never which run):
   - **BUILD** (no `--investigate`): all phases 0 → 9, in strict order: safety-net tests (Phase 3) → plan + grill (Phase 4) → code (Phase 6). Never skip earlier phases. `--auto` keeps this order — it removes approval gates, never the tests-first sequence.
   - **INVESTIGATE** (`--investigate`): read-only subset, Phases 1 → 5. Skip Phase 0 (never branches) and Phases 6-9 (never refactors). First use the `investigate-contract` skill (read-only guarantee + interactive-vs-`--auto` behaviour).

## Autonomous build (`--auto`)

`--auto` without `--investigate` runs the full refactor unattended for batch/background use. Every phase still runs in strict order; human gates are lifted and the **draft PR is the terminal deliverable**. Guiding rule: **never block on input** — when scope is underspecified, record an "Assumption made" and proceed.

Lifted vs interactive build (the safety net is **not** lifted):

- **No questions** — skip `grill-me` and every "ask the user" step.
- **No approval gates** — branch, the Phase 3 test commit, Phase 6 step commits, and the PR happen without confirmation.
- **Safety net still FIRST** — Phase 3 still writes characterization tests and proves them green before any refactor; it just commits them directly (still test-only, before production code).
- **Headless verification only** — automated tests only (interactive UI checks are flaky unattended). For UI-touching refactors, add "visual verification required" to the PR's verification steps.

Verification & failure (the unattended safety core):

- **Green gate, full check suite** — before opening the PR, run the project's full checks (type-check + lint + format + tests); auto-fix what's auto-fixable. Each step commit still requires green tests first.
- **Mutation-smoke the char tests** — before trusting the net, break the code under test; tests must go red, then **revert the mutation**. A green char test that asserts nothing is no safety net — and unattended, nobody catches it. Judge by mutations caught, never coverage %.
- **Adversarial review** — run a code-review pass, fix criticals (esp. behaviour drift) yourself, note the rest in the PR.
- **Self-repair while it converges** — review rejects or checks won't go green → fix and retry, as long as **each round clears a distinct new failure** (no fixed retry cap). Stop the moment a round **repeats a failure or makes no progress** → **do not open a PR**: flag the ticket's owner with the reason (no ticket → report in the run output), then stop. **Never push red, never open a failing PR, never loop on the same failure.**
- **Stop at the draft PR** — open a draft; lead the body with a **⚠️ banner** of scope assumptions taken and any review-flagged behaviour-drift, plus a **🐞 Suspected bugs** section. Never mark it ready or merge.

> **Commit exception (build):** characterization tests (Phase 3) get their **own commit BEFORE refactoring begins** — overriding the commit skill's "tests with change" rule. They capture existing behaviour and are the deliverable of Phase 3, not paired with a code change.

## Progress signposting

The user can't otherwise tell which phases ran or were skipped. **As you enter each phase, print a one-line signpost first** — `▶ Phase N — <short phase name>` — then do the work. One terse line. Don't signpost phases the active mode skips.

## Phase 1: Understand scope

1. If `$ARGUMENTS` has a ticket ID, fetch it from the tracker. _Investigate_: save the full existing description verbatim (you append to it in Phase 5, never overwrite).
2. _Build_: without a ticket, treat `$ARGUMENTS` as an area/description; if empty, ask (_auto: empty → stop and report, nothing to refactor — never ask_). _Investigate_: a ticket ID is **required** (stop and report if missing).
3. **Parse** title (module/area hint from brackets) and description (goal, motivation, constraints). If the goal is implicit, infer it and state the inference as an "Assumption made".
4. **Classify the goal**: readability, performance, structure/architecture alignment, deduplication, testability, dependency removal, naming, extracting concerns, etc. Multiple goals is fine — list them. Propose your own refactor ideas in the touched area.
5. **Classify per Fowler** when applicable: Extract Function/Component, Inline, Move, Rename, Replace Conditional with Polymorphism, Simplify Conditional, Encapsulate, etc.
6. **Resolve ambiguity** (see `investigate-contract` for the interactive-vs-`--auto` rule): ask only the question(s) that _materially_ change the output (intended behaviour change vs pure refactor, scope boundaries, target architecture when multiple are plausible); record minor uncertainties as "Assumptions made" and proceed.
   - _Build, interactive_: resolve scope questions here; the plan itself is pressure-tested with `grill-me` in Phase 4.
   - _Build, auto_: skip scope questions and grill-me; record assumptions and proceed.

## Phase 2: Ground in the codebase

Use the `understand-project` skill to ground the work in existing code.
When genuinely unsure whether an area applies, read it — but skip one the refactor plainly never touches.

Output: the target pattern to align with (point at a file that already follows it — "align with `<module>/`"), the full touch surface (every file to edit/move/delete, tracing data flow and callers), and any third-party API to check against version-matched docs.

## Phase 3: Safety net — tests FIRST

**Refactoring without a safety net is reckless.** Check existing coverage **first** (run the relevant tests), then write only the tests you actually need. Then:

- **Build** — guarantee a green safety net **before any code change**:
  1. If coverage is sufficient, confirm it and move on.
  2. If insufficient, write **characterization tests** (Feathers) — capture _current_ behaviour, not ideal behaviour. Read existing tests first and follow their patterns; implement one at a time, verifying each passes. All must pass.
  3. **STOP before committing — even though these are "just tests."** Mandatory, never skip. List the test files, summarize the behaviour they pin down, and say: "Characterization tests ready. Please review and confirm when ready to commit." Do NOT commit without explicit approval. Same gate as Phase 6 — and this is exactly where it tends to get skipped, so hold it. _Auto: skip the approval wait, but first **mutation-smoke** the char tests (break code → must go red → revert); once they bite and are green, commit them directly (still test-only, before any refactor)._
  4. Once the user confirms (interactive) → **commit the characterization tests**, before any refactoring, via the `commit` skill with a `test:` prefix. ONLY test files — zero production code (the commit exception above).
     - **Exception — new pure-function extraction**: when the plan extracts a NEW pure function, its stub (signature + minimal implementation copied from existing logic) may ride in the test commit so tests can import it. Direct copy of existing logic, not new behaviour.
- **Investigate** — no code written; **assess** the net instead. Classify each affected area as **Sufficient** (behaviour to preserve is covered → plan goes straight to refactor steps), **Partial** (gaps on critical branches → plan lists char tests to fill them), or **None** (untested → plan lists char tests to write first). This drives the "Safety net" section of the posted plan.

### Mutation-test the safety net (both modes, before trusting it)

A green test suite proves nothing until you've seen it go red. For the critical behaviours the refactor must preserve, insert a credible bug (invert a condition, change a threshold, drop a branch), confirm the relevant test fails, then **revert the mutation**. Repeat for 3-4 key behaviours. Report as a table: mutation | caught? | action taken. A test that stays green under mutation is not protecting you.

### Testability through pure-function extraction (both modes)

When logic is coupled to the UI framework (hooks, components), extract the core into a **pure function** taking parameters and returning values; the hook becomes a thin wrapper wiring state to it. This makes every branch testable with plain assertions — no rendering harness, no mocks. Often the right _first_ step (build) or highest-leverage suggestion (investigate).

```ts
// ❌ hard to test — logic buried in hook, needs render harness + mocks
export const useRedirectTarget = () => {
  const hasNewItems = useHasNewItems();
  const shouldShowAnnouncement = useShouldShowAnnouncement();
  if (hasNewItems) return "/new";
  if (shouldShowAnnouncement) return "/announcement";
  return null;
};

// ✅ pure function — all branches testable with plain assertions
export const computeRedirectTarget = (
  hasNewItems: boolean,
  shouldShowAnnouncement: boolean,
): string | null => {
  if (hasNewItems) return "/new";
  if (shouldShowAnnouncement) return "/announcement";
  return null;
};

// hook = thin wrapper, just wires state
export const useRedirectTarget = () =>
  computeRedirectTarget(useHasNewItems(), useShouldShowAnnouncement());
```

## Phase 4: Build the plan

Compose the plan from Phases 1-3. Pick the simplest, cleanest solution — fewest files moved, smallest behaviour surface, reuse existing patterns.

- **Investigate** — mid-depth plan, no commit breakdown, no alternatives. Five sections, which become the posted block in Phase 5:
  - **Approach** — 3-6 bullets: Fowler classification(s), target pattern reference (a file that already follows it), invariant preserved (no behaviour change unless the ticket asks).
  - **Safety net** — the Phase 3 verdict (Sufficient / Partial / None) and, if Partial/None, the char tests to write first; mention pure-function extraction when relevant.
  - **Impacted files** — table of every file to create/edit/move/delete with a one-liner.
  - **Steps** — small atomic steps (Beck); each keeps tests green. If char tests are needed, they are step 1 in a dedicated commit.
  - **Test strategy** — table per layer (unit / integration / E2E); drives char tests for None/Partial, lists regression checks for Sufficient.
- **Build** — break the refactor into **small atomic steps**, each keeping tests green. Present an ordered step table; note a rollback strategy for each medium/high-risk step:

  | Step | Refactoring              | Files | Risk   |
  | ---- | ------------------------ | ----- | ------ |
  | 1    | Extract component X      | path  | Low    |
  | 2    | Move to /modules/domain/ | paths | Medium |

  Then use the `grill-me` skill to pressure-test the **refactoring plan and its scope** — resolve each branch before committing to steps. grill-me is a long loop that does **not** hand control back on its own — when the interview concludes, **return to this skill and continue**; do NOT jump straight to code. If you reached planning without a green, committed characterization-test commit, stop and complete Phase 3 first. **Wait for user approval before proceeding.** _Auto: skip grill-me and the approval wait; record open calls as assumptions and proceed (the Phase 3 safety-net precondition still holds)._

## Phase 5: Post plan to the tracker — _investigate only_

The plan block is the investigate deliverable; post it via the `save-plan-to-tracker` skill. **Interactive: present it in chat, fold in the user's edits, post once they approve** (`investigate-contract` → "Review before posting"). **`--auto`: post directly, no prompt.** _Build never posts — the PR carries the plan._

Template for the block (on top of `save-plan-to-tracker` mechanics):

```
## 🎯 Automatic plan — Refactor

### 📋 Context
[Goal in 2-3 sentences. Refactor category(ies): readability / perf / structure / dedup / testability / etc.]

[If assumptions were made for lack of ticket detail, list under "Assumptions made:" — one line each.]

### 🛠️ Recommended approach

[3-6 bullets.
- Fowler classification (e.g. "Extract Function + Move Module")
- Target pattern: point to an existing file that already follows it (e.g. "Align with <module>/")
- Preserved invariant: no behaviour change (unless the ticket explicitly says otherwise)
- Prefer the simplest solution — fewer files moved, less surface, reuse existing patterns.]

### 🛡️ Safety net — test coverage

[Current coverage state: Sufficient / Partial / None]

[If Partial or None: list the characterization tests to write BEFORE the refactor — they capture current behaviour, not ideal (Feathers). Mention if a pure-function extraction is a relevant first step to gain testability.]

### 📂 Impacted files

| File | Action | Description |
| ---- | ------ | ----------- |
| path/to/file | Edit / Move / Create / Delete | Short description |

### 📝 Steps

1. [Atomic step — small, keeps tests green]
2. [Next step]

[If characterization tests are required: they are step 1, in a dedicated commit before any refactor.]

### 🧪 Test strategy

| Layer | Scenario | File |
| ----- | -------- | ---- |
| Unit | ... | ... |
| Integration | ... | ... |
| E2E | (if relevant) | ... |

---
*Automatic plan — human validation required*
```

**Investigate stops here.** The remaining phases are build only.

## Phase 6: Implement — small steps, always green — _build only_

**Precondition — verify the safety net before touching production code.** The core invariant is _never refactor without tests_. Confirm Phase 3 is done: characterization tests exist, pass, and are in a **test-only commit**. The long grill-me interview in Phase 4 is the most common place this gets dropped — if you can't point to that green test commit, go back and complete Phase 3 now. No green safety net → do not implement.

Core loop (repeat for each step from the Phase 4 plan):

1. Apply ONE refactoring step.
2. Run the test suite — must stay green. Snapshot failures caused ONLY by expected structural changes (no behaviour change) → update them, verify the diff is structural, then proceed. Tests breaking for other reasons → revert the step, analyze, adjust. Include updated snapshots in the same commit as the step that caused them.
3. **STOP before committing — even for a one-file change.** Mandatory, never skip. List changed files, summarize, say: "Step N done. Please review in your editor and confirm when ready to commit." Do NOT commit without explicit approval.
4. Once the user confirms → commit via the `commit` skill. Refactoring commits contain production code + snapshot updates caused by it; NOT new test files (those belong to the Phase 3 commit).

_Auto: skip steps 3-4 — once the suite is green, commit directly and move on (a step that breaks tests for a non-structural reason is reverted at step 2, never committed red)._

### Checklist

- [ ] Follow the project's architecture/structure conventions
- [ ] Follow the project's styling and import conventions
- [ ] No behaviour changes — refactoring only (if a behaviour change is needed, flag it to the user)
- [ ] Use version-matched library docs when needed

## Phase 7: Verify — _build only_

Run the full suite. If the refactor touched UI, drive the app to the affected screen and confirm there is **no** visual change. Check for regressions in related modules. Propose manual verification steps for the user. _Auto: gate on the full check suite instead; skip interactive UI checks (automated tests only) — flag "visual verification required" in the PR._

## Phase 8: Document — _build only_

Delegate to the `document` skill. Only for architecture deviations, non-obvious decisions, or a needed migration guide. **Skip if self-explanatory** — good refactoring speaks for itself.

## Phase 9: Review & PR — _build only_

1. **Review (pre-PR)** — run a code-review pass on the current diff (a subagent works well). Surface its findings; **address criticals** (especially behaviour drift) before the PR; note the rest for the user. Keep it lightweight — a gate, not a second refactor loop. _Auto: fix criticals yourself; keep repairing while each round clears a new failure — when a round stops making progress → flag the ticket owner, no PR (see Autonomous build)._
2. **Open PR** — ensure all tests pass, propose manual verification steps for the reviewer, **wait for user confirmation**, then use the `open-pr` skill with the `[Refactor]` prefix. _Auto: gate on the full check suite (not just tests), skip the wait, then `open-pr` (draft) with the ⚠️ assumptions banner + 🐞 Suspected bugs, and stop._
