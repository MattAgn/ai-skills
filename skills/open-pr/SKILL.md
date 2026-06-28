---
name: open-pr
description: MANDATORY skill for ALL pull requests. Must be used EVERY TIME before creating any pull request. No exceptions.
---

# Generating Pull Requests

## Mandatory Process

0. **ALWAYS** ensure the developer tested the code thoroughly by hand → propose the scenarios to test
1. **ALWAYS** push the branch and check for any errors returned by git hooks
2. **ALWAYS** fix the errors if any — fixup into the relevant commits, or create a new commit if fixup doesn't apply
3. **ALWAYS** generate the PR as a **draft**, following the repo's PR template (use the bug template if the repo has one and this is a bug fix; otherwise the default template)
   1. Read the relevant template file first and use its structure in the PR body
   2. **ALWAYS** create the PR as a **draft** — only the user decides when a PR is ready for review
   3. The title should start with a type prefix matching the repo's convention (e.g. `[Feat]` / `[Fix]` / `[Refactor]` / `[Chore]` / `[Doc]`)
   4. Add the appropriate label (e.g. a `bug` label for a PR fixing a bug) if the repo uses them
   5. **NEVER** add an issue-tracker ticket id in the PR title

> Note: if your PR tool requires the body inline rather than via a template flag, read the template file and pass its filled-in content as the body — don't rely on the two being combinable.

## Writing the description

The description is for a human reviewer who needs to grasp *what this PR does* at a glance. Write the kind of summary you'd write by hand.

- Summarize the **main changes only** — the meaningful, functional changes a reviewer needs to know about. A few clear bullet points or short sentences is enough.
- **NEVER dump commit details** — do not paste commit messages, do not write a commit-by-commit breakdown. The git history already holds that; repeating it just adds noise.
- **Skip non-important changes** — small refactors, formatting, renames, lint fixes. They dilute the signal; leave them out.
- If the description reads like a changelog of every diff, it's wrong. Clear, concise, high-level — that's the bar.

## Filling the template

Blockquote (`>`) lines in a template are **instructions to the author**, not content. Strip every one of them out — they must never appear in the final PR body.

Conditional sections should be included only when they actually apply; otherwise remove them entirely so the PR stays clean. Common examples:

### Screenshot / mockup table

Keep a screenshot/mockup table **only if the change is visual** — a feature or fix that alters what the user sees on screen. In that case leave the cells blank; the user adds the images himself. If the change is **not visual** (refactor, technical change, config, build/CI, pure logic), **remove the table** — there's nothing to screenshot.

### "Experiments / notable approaches" section

Keep a section like this **only if something genuinely new or experimental was done** — a new approach tried, a tool or workflow experimented with. If the work was just routine, **remove the section entirely** — routine work is expected, not an experiment, and doesn't need reporting.
