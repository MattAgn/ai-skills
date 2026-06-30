---
name: fix
description: Debug and fix any non-trivial issue end-to-end, OR triage a bug ticket read-only. Default (`/fix [ticketId]`) = systematic debugging → fix → PR. `--investigate` = read-only bug triage that posts a structured investigation to the ticket (interactive). `--investigate --headless` = the same, fully headless (no questions) — used by a nightly batch.
argument-hint: [ticketId-or-description] [--investigate] [--headless]
disable-model-invocation: true
---

Systematic debugging assistant. One flow of phases; the mode only changes how a few behave (tagged inline).

## Mode selection

1. **Flags**: scan `$ARGUMENTS` for `--investigate` and `--headless`; strip them — the remainder is the ticket ID / description.
2. **Validate**: `--headless` is only valid with `--investigate`. If alone, **stop and report**: "`--headless` only applies to `--investigate` (headless triage). A headless fix isn't supported — drop `--headless`."
3. **Which phases run**:
   - **BUILD** (no `--investigate`): all phases 0 → 11.
   - **INVESTIGATE** (`--investigate`): read-only subset, Phases 1 → 8. Skip Phase 0 (never branches) and 9-11 (never fixes). First use the `investigate-contract` skill (read-only guarantee + interactive-vs-`--headless` behaviour).

## Progress signposting

The user can't tell which phases ran or were skipped. **Entering each phase, first print a one-line signpost** — `▶ Phase N — <short name>` — then do its work. One terse line. Don't signpost skipped phases.

## Phase 1: Get the bug

- If `$ARGUMENTS` has a ticket ID, fetch it immediately — title, description, priority, labels, attachments. Scan attachments for links to error-monitoring data.
- _Build_: without a ticket, collect bug info from the user (or ask whether they'll provide a ticket ID).
- _Investigate_: a ticket ID is **required** (stop and report if missing). If the issue has an "investigated" label — in `--headless` skip it and report "already investigated"; interactive, mention it and proceed only if a fresh pass is wanted.

## Phase 2: Understand the bug

1. **Parse title** — module hint from brackets (e.g. `[Auth]` → auth module).
2. **Parse description** — reproduction steps, environment, error messages, affected users.
3. **Error-monitoring data** — if a link was found, pull error message, stacktrace, breadcrumbs, affected releases, device/OS distribution. Look for patterns (specific devices? OS versions? recent release only?).
4. **No monitoring link?** — proactively search your error-monitoring tool with a natural-language query from the title / module / error message. Try 2-3 variants (module name, error type, screen name). If a relevant issue is found, pull its full stacktrace; else note it and proceed.
5. **Triage** — keep only info pertinent to solving the bug: stacktrace source, breadcrumbs explaining the user flow, discriminating tags. Skip error name (already in title), status, release (unless a regression), OS/device (unless platform-specific).
6. Summarize: what is the bug, which module, what error, what context exists.

## Phase 3: Understand the system

Trace the code path. Starting points (use whichever apply):

- Module name from Phase 2 → read that module (hooks, components, utils, state)
- Stacktrace → read the exact files and lines
- Error message → grep the codebase for the string
- API-related → check the relevant client/endpoint hooks

For each relevant file: read it, trace the data flow (entry → processing → error), identify all components/hooks/utils involved, note suspicious patterns (missing error handling, race conditions, implicit assumptions). **Keep a running list of every file analyzed** — it becomes the "code path".

## Phase 4: Form hypotheses

Form 3-5 testable hypotheses (race conditions, null/undefined, stale state, API contract mismatch, environment-specific, recent regression, library bug/misuse). Apply **Evidence discipline** as you write them.

### Evidence discipline (read before writing any hypothesis)

The failure mode this kills: you read code, spot a line that _looks_ like the culprit, and present that hunch as fact. A suspicious line is a **clue**, not proof — sounding certain on a clue sends the reader (the user, or a dev acting cold on the ticket) chasing the wrong thing.

Tag every claim as exactly one, never blurred:

- **Proven** — you read the exact code and can quote it (`file:line` + snippet). For a data-flow claim ("`undefined` reaches `X`"), proven means you traced _every hop_, not that the endpoints look connected. Can't paste the code? **Not** proven.
- **Inferred** — a reasonable deduction you have NOT verified. Inference generates leads — but say so ("I suspect…", "unverified", "haven't traced this"). Never let an inference wear the costume of a finding.

Highest confidence is reserved for hypotheses whose mechanism is backed by quoted code, never for how plausible the story feels. Gut-check before typing: _"Can I paste the code that proves this, or am I pattern-matching?"_

Example:

❌ clue-as-fact: "**H1 (High)** — the crash comes from the player not handling `failed`." (nothing quoted, path never traced — "High" unearned.)
✅ disciplined: "**H1 (Medium)** — the player may not handle `.failed`. **Proof** `Player.x:142`: handler only has `case .ready`. **⚠️ Critical link**: that `.failed` is actually emitted by the SDK on load error — if it never is, H1 collapses → read the SDK source first."

### Hypothesis format (always maintain)

Bullet points, not a table. Status emoji (⏳ to validate / ✅ confirmed / ❌ disproven) before `Hx`. Maintain in the ticket when one exists.

```
**⏳ H1 — [short hypothesis title]**
- **Hypothesis**: [the proposed mechanism — what would cause the bug]
- **Proof in code**: `path/file:42` + quoted snippet. If nothing to quote, write "none — deduction at this stage".
- **⚠️ Critical link**: THE single unproven assumption that, if false, collapses the hypothesis — and how to prove/refute it. Dig this FIRST (Phase 5). "none" if everything is proven.
- **Unverified**: *secondary* assumptions (repro, timing, runtime value) that don't threaten the hypothesis.
- **Validation**: how to verify at runtime — concrete action, log, test.
- **Probability**: High / Medium / Low — High only if the mechanism AND its critical link are backed by quoted code.
```

`⚠️ Critical link` is its own line, not buried in `Unverified`: a flat caveat list hides which one is load-bearing. Isolating it spotlights the exact spot you're most likely to be confidently wrong.

## Phase 5: Dig the critical link

Principle from grill-me: _a question you can answer by reading code, you answer by reading code_ — don't park the critical link as "unverified" and wait. For each hypothesis, prove or refute its **⚠️ Critical link** statically _before_ settling confidence. Dig nearest to farthest, no stopping at the first layer:

1. **Trace the code path** — every hop, assume no intermediate step.
2. **Read dependency source** — a library's behaviour is readable, not a guess. Use version-matched docs.
3. **Search for guards** — null checks, error boundaries, try-catch that would prevent the bug.
4. **Grep for the pattern** — does the same pattern work elsewhere?
5. **Check git history** — `git log --oneline -20 -- <file>` and `git blame` on suspicious lines.
6. **Compare with working code** — if a similar feature works, what's different?
7. **Search the web (library/dep hypotheses only)** — exact error + library + installed version. Fold findings inline into that hypothesis's evidence (URL + one-line takeaway). Skip for pure business-logic bugs.

**This is NOT self-validation.** Proving a mechanism _possible and correct_ by reading code ≠ confirming it _actually happened_ here. Do the first exhaustively yourself; settle the second in Phase 6. Escalate to runtime only for what is genuinely **undecidable statically**: real runtime values, timing/races, device/environment-specific behaviour.

## Phase 6: Settle the root cause

- **Build** — validate with the user at runtime. **NEVER self-validate**: only the user decides confirmed/disproven. For each hypothesis: (1) present evidence split into **proven** (quoted `file:line`, stacktrace, logs you saw) vs **inferred**; (2) propose concrete validation methods — a log line + reproduce, check monitoring breadcrumbs/tags, a try-catch to isolate the call site, `git bisect`, local repro, comment-out by elimination, or driving the app to inspect a visual bug; (3) **wait for the user to confirm or disprove** before updating status. **No fix is written before a hypothesis is user-confirmed (✅).**
- **Investigate** — no runtime, no user (especially `--headless`). Rate each hypothesis statically: **High** (mechanism AND critical link proven by quoted code, nothing contradicting), **Medium** (mechanism partly code-backed, critical link needs runtime confirmation), **Low** (code contradicts it, or guards already exist). Be honest about limits and always state the runtime test that would close the remaining critical link.

## Phase 7: Bug analysis

Walk the bug-PR template (if the repo has one):

1. **Code analysis** — the "before" snippet; what's wrong and why it causes the bug.
2. **Spread check** — grep for the same pattern; list every instance.
3. **Prevention plan** — concrete actions: `[test]` / `[lint]` / `[arch]` / `[doc]`.

- _Build_: track each prevention item and spread instance; resolve in Phase 10. Spread instances join the fix scope.
- _Investigate_: post these as **suggestions** only — don't implement.

## Phase 8: Post to the tracker

Post the investigation (hypotheses + code analysis + prevention) to the ticket using the `save-plan-to-tracker` skill, then add labels. Available in **both** modes:

- _Investigate_: the block — the [`investigation-template`](investigation-template.md) — is the deliverable. **Interactive: present the draft in chat, fold in the user's edits, post only once they approve** (`investigate-contract` → "Review before posting"). **Headless: post directly, no prompt.**
- _Build_: when a ticket exists, **offer** it — "Post/update the hypotheses on the ticket?" — and keep it updated as statuses change (a living diagnostic log).

**Label (every mode, every post)**: add an "investigated" label. ⚠️ If the tracker **overwrites** the label set rather than merging, passing only the new label silently drops every existing one (e.g. `bug`). Pass the **union**: reuse the labels from the Phase 1/2 fetch, append the new one, de-duplicated.

**Investigate stops here.** The remaining phases are build only.

## Phase 9: Fix planning — _build only_

Don't jump to the first fix — propose multiple approaches, let the user choose.

| Fix approach    | Type       | Pros                                  | Cons                      | Effort   | Fixes spread?  | Enables prevention? |
| --------------- | ---------- | ------------------------------------- | ------------------------- | -------- | -------------- | ------------------- |
| [Quick patch]   | Patch      | Fast, low risk                        | Doesn't fix root cause    | Low      | Yes/No/Partial | Which items         |
| [Refactor/arch] | Structural | Fixes root cause, prevents recurrence | More changes, higher risk | Med-High | Yes/No/Partial | Which items         |

Types: **Patch** (guard clause, null check), **Structural** (fix the pattern/architecture), **Upstream** (dependency PR/update/workaround), **Configuration**. Always propose ≥2 approaches when the root cause is architectural; assess whether each fixes the spread; present trade-offs and let the user decide.

## Phase 10: Implement & verify — _build only_

- Apply the chosen fix; fix **all** spread instances; implement the prevention items.
- Verify the bug is resolved and tests pass.
- **STOP before committing — even for a one-file change.** Mandatory. List changed files, summarize, say: "Fix ready. Please review in your editor and confirm when ready to commit." Do NOT commit without explicit approval. Never skip this.
- Once the user confirms → commit via the `commit` skill.

## Phase 11: Review & PR — _build only_

1. **Review (pre-PR)** — run a code-review pass on the current diff (a subagent works well). Surface findings; **address criticals** (real bugs, regressions in the fix) before the PR; note the rest for the user. Keep it lightweight — a gate, not a second debugging loop.
2. **Open PR** — assemble from Phase 7 (fill the "after" snippet; "before" was captured there). Commit fix + tests + spread fixes. Use the `open-pr` skill with the `[Fix]` prefix and the `bug` label.
