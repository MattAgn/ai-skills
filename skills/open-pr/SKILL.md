---
name: open-pr
description: Mandatory skill to use every time before creating any pull request.
---

# Generating Pull Requests

## Process

1. Ensure the developer tested the code by hand — propose the scenarios to test.
2. Push the branch and check for errors from git hooks; fix any by fixing up the relevant commit (or a new commit if fixup doesn't apply).
3. Create the PR as a **draft** — only the user decides when it's ready for review.
   - **Exception — attended callers reaching merge** (see `run-mode`): when the caller has validated the running app and is opening the PR to land it (default PR review, or `--auto-merge`), open it **ready**; let the user review unless `--auto-merge`, then rebase on `main` (resolve conflicts, re-run checks) and **merge** per the repo's convention. Stop and flag instead if a conflict isn't a clean resolution or checks won't go green. An `--unattended` run or a standalone invocation stays draft-and-stop — never merges.
   - Follow the repo's PR template: read the template file first and use its structure in the body (use the bug template for a bug fix if one exists, otherwise the default).
   - If your tool wants the body inline rather than via a template flag, pass the filled-in template content as the body.
   - Title starts with a type prefix per the repo's convention (e.g. `[Feat]` / `[Fix]` / `[Refactor]` / `[Chore]` / `[Doc]`). Never include an issue-tracker ticket id.
   - Add the appropriate label (e.g. a `bug` label) if the repo uses them.

## Writing the description

For a human reviewer who needs to grasp *what this PR does* at a glance — the kind of summary you'd write by hand.

- Summarize the **main, meaningful changes only**, in a few clear bullets or short sentences.
- Never dump commit details or write a commit-by-commit breakdown — the git history already holds that.
- Skip minor changes (refactors, formatting, renames, lint fixes); they dilute the signal.

## Filling the template

Blockquote (`>`) lines are instructions to the author, not content — strip them all from the final body.

Include conditional sections only when they apply; otherwise remove them entirely. Common cases:

- **Screenshot/mockup table** — keep only if the change is visual (alters what the user sees). For a web-app UI change, capture the affected screen(s) — reuse the verification shots, or drive the browser now (a short series when the change spans a multi-step flow). Save each to a file and display it in the chat (read the image back) so the user can grab it, then leave the table cells holding the file paths plus a one-line note: drag these into the PR to embed them (GitHub can't upload via CLI). Remove the section entirely for non-visual changes (refactor, config, build/CI, pure logic).
- **Experiments / notable approaches** — keep only if something genuinely new or experimental was done. Remove it for routine work.
