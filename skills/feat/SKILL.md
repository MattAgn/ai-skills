---
name: feat
description: Build a feature end-to-end (issue-tracker ticket or free-text → plan → implement → PR), OR plan one read-only with `--investigate`. Add `--headless` to run headless (never asks; build stops at a draft PR). Use to build a feature, or to plan one ahead without writing code.
argument-hint: [ticketId-or-description] [--investigate] [--headless]
disable-model-invocation: true
---

Full feature lifecycle orchestrator. One flow of phases; the mode only changes how a few phases behave (tagged inline).

## Mode selection

1. **Flags**: scan `$ARGUMENTS` for `--investigate` and `--headless`; strip them — the rest is the ticket ID / description.
2. **Resolve the mode** — the flags are independent, giving four combinations:

   | Flags                  | Mode                        | Behaviour                                                                                       |
   | ---------------------- | --------------------------- | ----------------------------------------------------------------------------------------------- |
   | _(none)_               | **interactive build**       | full build, human in the loop (default)                                                         |
   | `--headless`               | **headless build**          | full build, headless → stops at a draft PR (see [Headless build](#headless-build---headless))       |
   | `--investigate`        | **interactive investigate** | read-only plan, human in the loop                                                               |
   | `--investigate --headless` | **headless investigate**    | read-only plan, headless                                                                        |

3. **Which phases run** (decided by `--investigate` alone; `--headless` only changes _how_ a phase behaves, never which run):
   - **BUILD** (no `--investigate`): all phases, 0 → 8.
   - **INVESTIGATE** (`--investigate`): read-only subset, Phases 1 → 4. Skip Phase 0 (never branches) and Phases 5-8 (never implements). First use the `investigate-contract` skill (read-only guarantee + interactive-vs-`--headless` behaviour).

## Headless build (`--headless`)

`--headless` without `--investigate` runs the full build **headless** — no human at the terminal. Every phase runs, human gates lifted, the **draft PR the terminal deliverable**. The prior the word recruits: a headless run has no one to ask, so it **never blocks on input** — when underspecified, record an "Assumption made" and proceed.

Lifted vs interactive build:

- **No questions** — skip `grill-me` and every "ask the user" step.
- **No approval gates** — branch, commits, and PR happen without confirmation.
- **Automated checks only** — no interactive UI/manual checks (flaky headless, the #1 false "it works"). For UI changes, add "visual verification required" to the PR's manual-test scenarios.

Verification & failure (the headless safety core):

- **Green gate, full check suite** — before the PR, run the project's full checks (type-check + lint + format + tests); auto-fix what's auto-fixable. Loop commits still require green tests first.
- **Mutation-smoke every test you write** — break the code under test; the test must go red, then **revert the mutation** (committing mutated code ships a broken build). A test asserting nothing fakes the safety net; judge by mutations caught, never coverage %.
- **Adversarial review** — run a code-review pass, fix criticals yourself, note the rest in the PR.
- **Self-repair while it converges** — review rejects or checks won't go green → fix and retry, as long as **each round clears a distinct new failure** (no fixed retry cap). The moment a round **repeats a failure or makes no progress** → **do not open a PR**: flag the ticket's owner with the reason (a ticket comment; no ticket → report in the run output), then stop. **Never push red, never open a failing PR, never loop on the same failure.**
- **Stop at the draft PR** — open a draft; lead the body with a **⚠️ banner** listing each recorded assumption ("observed behaviour, assumed intended — to confirm") and a **🐞 Suspected bugs** section. Never mark ready or merge.

## Progress signposting

The user otherwise can't tell which phases ran or were skipped. **On entering each phase, print a one-line signpost first** — `▶ Phase N — <short phase name>` — then do the work. One terse line, no preamble or recap. Don't signpost phases the active mode skips.

## Phase 1: Understand requirements

1. If `$ARGUMENTS` has a ticket ID, fetch it immediately — title, description, priority, labels, attachments. _Investigate_: save the full existing description verbatim (you append in Phase 4, never overwrite).
2. _Build_: without a ticket, treat `$ARGUMENTS` as free-text; if empty, ask for a ticket ID or description (_headless: empty → stop and report, never ask_). _Investigate_: a ticket ID is **required** (stop and report if missing).
3. **Parse** the title (module hint from brackets, e.g. `[Auth]` → auth module) and description (user-facing goal, acceptance criteria, edge cases, mockups in attachments).
4. **Identify feature type**: new screen, module, extension of an existing feature, component, settings toggle, etc.
5. **Resolve ambiguity** (see `investigate-contract` for the interactive-vs-`--headless` rule): ask only question(s) that _materially_ change the output; record minor uncertainties as "Assumptions made" and proceed.
   - _Build, interactive_: use `grill-me` to pressure-test the **scope** until unambiguous. grill-me is a long loop that does **not** hand control back on its own — when the interview concludes, **return to this skill and continue**; do NOT jump to planning or code.
   - _Build, headless_: skip grill-me; record assumptions and proceed.

## Phase 2: Ground in the codebase

Use `understand-project` to ground the work in existing code. Don't bulk-read every convention — most features touch only a slice. Read the project's **architecture/structure** conventions by default (they decide module/screen placement)

When genuinely unsure whether an area applies, read it — grounding is the point and a wrong plan costs more than one doc. But skip an area the feature plainly never touches.

Output: a similar feature to point at by path, the reusable assets (components/hooks/utils/stores/generated clients) to reuse by name, and the full touch surface (files, modules, routes, constants).

## Phase 3: Build the plan

Compose the plan from Phases 1-2. Pick the simplest, cleanest solution — reuse existing patterns, fewest files touched, smallest new surface.

- **Investigate** — mid-depth plan: no commit breakdown, no alternatives, no 1:1 commit mapping. Compose the four-section block (Approach / Impacted files / Steps / Test strategy) in the [`investigate-plan-template`](investigate-plan-template.md) format — it becomes the Phase 4 deliverable.
- **Build** — present the plan **commit by commit** with key implementation details, tests in the same commit as the code they cover:

  | File | Action      | Description  |
  | ---- | ----------- | ------------ |
  | path | Create/Edit | What changes |

  Propose refactors in the touched area only if the feature needs them. Organise into ordered atomic commits. Then use `grill-me` to pressure-test the **plan and scope** — same return-guard as Phase 1. **Wait for user approval before proceeding.** _Headless: skip grill-me and the approval wait — record open calls as assumptions and proceed._

## Phase 4: Post plan to the tracker — _investigate only_

The plan block is the investigate deliverable; post it via `save-plan-to-tracker`. **Interactive: present in chat, fold in the user's edits, post once they approve** (`investigate-contract` → "Review before posting"). **Headless: post directly, no prompt.** _Build never posts — the PR carries the plan._

Post the [`investigate-plan-template`](investigate-plan-template.md) block (on top of the `save-plan-to-tracker` mechanics).

**Investigate stops here.** The remaining phases are build only.

## Phase 5: Test plan — _build only_

Define the testing strategy before implementing. Check for missing tests on the touched feature and propose to write them. Present a test-plan table (Unit / Integration / E2E — scenario + file); user confirms. Tests are written in Phase 6 alongside the code. _Headless: define the plan and proceed without confirmation._

## Phase 6: Implement & verify — _build only_

**Precondition**: a plan exists (auto needs only this); interactive additionally requires it grilled and user-approved — the long grill-me interview is the most common place this gets dropped, so if you can't point to an approved plan, finish Phase 3 first.

Core loop (repeat per commit from the Phase 3 plan):

1. Write code + tests for ONE logical chunk. Read existing tests first and follow their patterns; propose scenarios and implement one at a time.
2. Run the test suite — must stay green. Snapshot failures from ONLY expected structural changes → update them, verify the diff makes sense, proceed. Tests breaking for other reasons → fix first. Include updated snapshots in the same commit as the code that caused them.
3. **STOP before committing — even for a one-file change.** Mandatory, never skip. List changed files, summarize, say: "Step N done. Please review in your editor and confirm when ready to commit." Do NOT commit without explicit approval. _Headless: skip steps 3-4 — mutation-smoke any test written (break code → red → revert), then once green commit directly and continue._
4. Once the user confirms → commit via the `commit` skill.

### Implementation checklist

- [ ] Follow the project's architecture/structure conventions (domain logic, screen/component placement)
- [ ] Reuse existing components, hooks, utilities, and generated clients rather than reinventing
- [ ] Register new screens/routes where the project expects them
- [ ] Validate route params / inputs; handle the not-found / invalid path
- [ ] Follow the project's styling and import conventions

For UI changes: drive the app to the affected screen, observe/screenshot it, and verify visually before asking the user to review. Fetch version-matched library docs when needed. _Headless: skip interactive UI checks (automated tests only) — flag "visual verification required" in the PR instead._

## Phase 7: Document — _build only_

Delegate to the `document` skill. Only for hacks, WHY reasoning, architecture deviations, non-trivial decisions. **Skip entirely** if the code is self-explanatory.

## Phase 8: Review & PR — _build only_

1. **Review (pre-PR)** — run a code-review pass on the current diff (a subagent works well). Surface findings; **address criticals** before the PR; note the rest for the user. Keep it lightweight — a gate, not a second build loop. _Headless: fix criticals yourself, then self-repair per [Headless build](#headless-build---headless)._
2. **Open PR** — ensure all tests pass, propose manual test scenarios for the reviewer, **wait for user confirmation**, then use the `open-pr` skill with the `[Feat]` prefix. _Headless: gate on the full check suite (not just tests), skip the wait, then `open-pr` (draft) with the ⚠️ assumptions banner + 🐞 Suspected bugs, and stop._
