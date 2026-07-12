---
name: run-mode
description: Shared run-mode model (attended vs `--unattended`, plus the `--review-commits` / `--auto-merge` toggles) for feat / refactor / fix. Internal helper invoked by those skills to resolve who's in the loop and where the run halts — not meant to be run on its own.
disable-model-invocation: true
---

Governs **who is in the loop** for a build run. The calling skill owns the phases; this contract owns which of them **halt for a human**, and whether the run ends at a draft PR or a merged one.

Two run modes. Parse `$ARGUMENTS`, strip the flags below, the remainder is the ticket ID / description.

## `--unattended` — real hands-off (cloud / CI)

No human at the terminal. **Detailed task in → draft PR out.** Implied when there's no TTY (a CI / cloud run), or set explicitly.

- **Never blocks on input** — underspecified → record an "Assumption made" and proceed; never ask.
- **Automated verification only** — run the full check suite; drive the app where it can be driven headless, but interactive UI checks are flaky, so for UI changes flag "visual verification required" in the PR instead of pretending.
- **Ends at a draft PR** — lead the body with a **⚠️ assumptions banner** (each: "observed behaviour, assumed intended — to confirm") + a **🐞 Suspected bugs** section. **Never merges** — a machine landing on `main` with nobody watching is the one bright line.
- **Attended-only toggles are no-ops** — `--review-commits` needs a human and `--auto-merge` describes a merge; under `--unattended` neither applies. If passed alongside `--unattended`, note "ignored under `--unattended`" up front and proceed.
- Governed throughout by the [execution core](#execution-core).

## Attended (default) — drives to a merged PR through checkpoints

A human is here. The run is autonomous _between_ checkpoints and halts _at_ them. Four, in order:

### 1. Plan — auto-routed (no flag)

Don't ask the human how to plan; **decide** it from the task:

- **Detailed + no material question** (nothing whose answer would change the plan) → state "Plan accepted as specified" and proceed. A clear, complete task earns no interview.
- **Fuzzy scope** → `/grill-me` — pressure-test until unambiguous.
- **Domain / architecture decisions worth recording** → `/grill-with-docs` (ADRs + glossary as you go).
- **Too big for one session** (fog — the route to the destination isn't visible, won't fit one agent session) → stop and hand off to `/wayfinder`; it charts the work as tickets first.

grill is a long loop that does **not** hand control back on its own — when the interview concludes, **return to the calling skill and continue**. Wait for plan approval before building.

### 2. Commits — auto unless `--review-commits`

Default: run the [execution core](#execution-core) and commit each logical chunk directly. `--review-commits`: **confirm each commit before it lands** — the `commit` skill's "Confirmation Before Committing" flow, per commit.

### 3. App works — always, mandatory

Before any PR opens, **validate the running app with the user** — drive it to the affected surface, confirm it functions (for a refactor: confirm _no_ visual change). This checkpoint has no toggle; attended never opens a PR the human hasn't seen work.

### 4. PR — review then merge, unless `--auto-merge`

- Default: open the PR **ready**, the user **reviews it**, then **rebase on `main`** (resolve conflicts, re-run the full check suite) and **merge** per the repo's convention (squash unless the repo differs).
- `--auto-merge`: skip the review — the app was already validated at checkpoint 3 — open ready, rebase, merge directly.

**Stop-and-flag during rebase/merge (both):** if a conflict isn't a clean resolution (it would change behaviour, or you're unsure) or the checks won't go green after the rebase, **do not merge** — stop and flag the reason. Autonomy ends where a silent behaviour change would begin.

## Execution core

Governs every autonomous stretch — the whole `--unattended` run, and attended commits when not using `--review-commits`. Autonomy is bought with verification:

- **Green gate, full check suite** — before the PR, run type-check + lint + format + tests; auto-fix what's auto-fixable. Every autonomous commit still requires green tests first.
- **Mutation-smoke every test you write** — break the code under test, confirm the test goes red, then **revert the mutation**. A test that stays green under mutation asserts nothing; judge by mutations caught, never coverage %.
- **Adversarial review** — run `/code-review` on the diff, fix criticals yourself, note the rest in the PR.
- **Self-repair while it converges** — a review reject or a red check → fix and retry, as long as **each round clears a distinct new failure** (no fixed retry cap). The moment a round **repeats a failure or makes no progress** → **do not open a PR**: flag the ticket owner with the reason (a ticket comment; no ticket → report in the run output), then stop. **Never push red, never open a failing PR, never loop on the same failure.**

## Signpost each phase

The user otherwise can't tell which phase is running. **On entering each phase, print a one-line signpost first** — `▶ Phase N — <short phase name>` — then do the work. One terse line, no preamble. Print the resolved mode + active toggles once, up front, so the user sees what will and won't halt.
