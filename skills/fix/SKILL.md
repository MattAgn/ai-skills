---
name: fix
description: Debug and fix any non-trivial issue end-to-end — ticket or free-text → loop-proven root cause → fix → merged PR. Diagnosis runs loop-first via `/diagnosing-bugs`. Attended by default (you confirm the root cause, validate the running fix, then review + merge). `--unattended` for cloud/CI (task → draft PR — only when a red loop can validate the fix). `--review-commits` to gate each commit; `--auto-merge` to skip the PR review.
argument-hint: [ticketId-or-description] [--unattended] [--review-commits] [--auto-merge]
disable-model-invocation: true
---

Systematic debugging orchestrator. Resolve the run mode via the `run-mode` skill (attended by default, `--unattended` for cloud/CI), then apply this skill's checkpoint map. Signpost each phase and print the resolved mode + toggles up front (`run-mode` → "Signpost each phase").

## Mode & checkpoints

This skill's [`run-mode`](../run-mode/SKILL.md) checkpoints map to:

| Checkpoint      | Where                                     | Halts when                              |
| --------------- | ----------------------------------------- | --------------------------------------- |
| **Plan** (root cause) | Phase 3 confirm the root cause      | attended: you confirm — see carve-out   |
| **Commits**     | Phase 6 per-commit                        | `--review-commits`                      |
| **App works**   | Phase 6 before the PR                     | always (attended)                       |
| **PR**          | Phase 7 open + merge                      | default; skipped by `--auto-merge`      |

**Root-cause carve-out (fix's Plan checkpoint is special).** A fix must never be built on a root cause "confirmed" by static inference alone — that self-delusion ships the wrong fix. The safe validator is a **tight red loop** (`/diagnosing-bugs`): a fast, deterministic, agent-runnable command that goes **red** on this exact bug and **green** once fixed. So the Plan checkpoint isn't grill-me/wayfinder — it's root-cause confirmation:

- **Attended** → you confirm the root cause (the loop red + the evidence) before any fix is written.
- **`--unattended`** → allowed **only because the red loop validates the fix in your place**. If no red loop can be built (`/diagnosing-bugs` "genuinely cannot build a loop"), an unattended fix is unsafe → **stop and report**; never self-validate from inference.

Autonomous stretches follow the [execution core](../run-mode/SKILL.md#execution-core).

## Phase 1: Get the bug

- If `$ARGUMENTS` has a ticket ID, fetch it immediately — title, description, priority, labels, attachments. Scan attachments for links to error-monitoring data.
- Without a ticket, collect bug info from the user (_unattended: the task is the input; empty → stop and report_).

## Phase 2: Understand the bug

1. **Parse title** — module hint from brackets (e.g. `[Auth]` → auth module).
2. **Parse description** — reproduction steps, environment, error messages, affected users.
3. **Error-monitoring data** — if a link was found, pull error message, stacktrace, breadcrumbs, affected releases, device/OS distribution. Look for patterns (specific devices? OS versions? recent release only?).
4. **No monitoring link?** — proactively search your error-monitoring tool with a natural-language query from the title / module / error message. Try 2-3 variants. If a relevant issue is found, pull its full stacktrace; else note it and proceed.
5. **Triage** — keep only info pertinent to solving the bug: stacktrace source, breadcrumbs explaining the user flow, discriminating tags. Skip error name (already in title), status, release (unless a regression), OS/device (unless platform-specific).
6. Summarize: what is the bug, which module, what error, what context exists.

## Phase 3: Diagnose the root cause — the Plan checkpoint

Run **`/diagnosing-bugs`** to reach a root cause a **tight red loop** proves (it owns the full loop / reproduce / minimise / instrument discipline). Fix carries two things back:

- The **minimised repro** — it becomes the Phase 6 regression test.
- The **loop**, red on this bug — the validator behind the Plan checkpoint.

Settle it per the [root-cause carve-out](#mode--checkpoints):

- _Attended_: present the evidence (loop red + the code) and **wait for the user to confirm or disprove** before writing any fix.
- _Unattended_: proceed on the red loop alone. **No loop buildable → stop and report.**

When a ticket exists, **offer** (attended) to post the diagnosis as a living log on it (`save-plan-to-tracker`) and keep it updated as the fix lands.

## Phase 4: Bug analysis

Walk the bug-PR template (if the repo has one):

1. **Code analysis** — the "before" snippet; what's wrong and why it causes the bug.
2. **Spread check** — grep for the same pattern; list every instance. Spread instances join the fix scope.
3. **Prevention plan** — concrete actions: `[test]` / `[lint]` / `[arch]` / `[doc]`. Track each; resolve in Phase 6.

## Phase 5: Fix planning

Don't jump to the first fix — propose multiple approaches:

| Fix approach    | Type       | Pros                                  | Cons                      | Effort   | Fixes spread?  | Enables prevention? |
| --------------- | ---------- | ------------------------------------- | ------------------------- | -------- | -------------- | ------------------- |
| [Quick patch]   | Patch      | Fast, low risk                        | Doesn't fix root cause    | Low      | Yes/No/Partial | Which items         |
| [Refactor/arch] | Structural | Fixes root cause, prevents recurrence | More changes, higher risk | Med-High | Yes/No/Partial | Which items         |

Types: **Patch** (guard clause, null check), **Structural** (fix the pattern/architecture), **Upstream** (dependency PR/update/workaround), **Configuration**. Always propose ≥2 approaches when the root cause is architectural; assess whether each fixes the spread; present trade-offs. _Attended: let the user decide. Unattended: pick the simplest approach that fixes root cause + spread, record the call as an assumption._

## Phase 6: Implement & verify

- Apply the chosen fix; fix **all** spread instances; implement the prevention items.
- **Regression test** — turn the Phase 3 minimised repro into a failing test at a correct seam, watch it fail, apply the fix, watch it pass (`/tdd`). If no correct seam exists, that itself is a finding — note it (the architecture is blocking lockdown). Then re-run the loop against the original, un-minimised scenario.
- **Cleanup** (from `/diagnosing-bugs`): remove all `[DEBUG-…]` instrumentation (`grep` the prefix), delete throwaway harnesses, and state the winning hypothesis in the commit/PR message so the next debugger learns.
- **Commits checkpoint** — _`--review-commits`: confirm before committing via the `commit` skill's confirmation flow._ _default / unattended: commit directly per the [execution core](../run-mode/SKILL.md#execution-core)._
- **App works checkpoint** — before the PR, confirm the bug is gone in the running app (`run-mode` → "App works"): _attended: drive the app / have the user confirm — always._ _unattended: the loop green is the proof; flag "visual verification required" for any UI aspect it couldn't drive._

## Phase 7: Review & PR

1. **Review (pre-PR)** — run `/code-review` on the current diff. Surface findings; **address criticals** (real bugs, regressions in the fix) before the PR; note the rest for the user. _unattended: fix criticals yourself, then self-repair per the [execution core](../run-mode/SKILL.md#execution-core)._
2. **PR** — assemble from Phase 4 (fill the "after" snippet; "before" was captured there). Commit fix + tests + spread fixes. Use the `open-pr` skill with the `[Fix]` prefix and the `bug` label. _attended: open **ready**; the user **reviews the PR**, then rebase on `main` and **merge** (`run-mode` → "PR"). `--auto-merge`: skip the review, open ready → rebase → merge._ _unattended: open a **draft** with the ⚠️ assumptions banner + 🐞 Suspected bugs, and **stop** — never merge._
