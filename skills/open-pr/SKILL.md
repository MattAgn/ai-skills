---
name: open-pr
description: Mandatory skill to use every time before creating any pull request.
---

# Generating Pull Requests

## Process

1. Ensure the developer tested the code by hand — propose the scenarios to test.
2. Push the branch and check for errors from git hooks; fix any by fixing up the relevant commit (or a new commit if fixup doesn't apply).
3. Create the PR as a **draft** — only the user decides when it's ready for review.
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

- **Screenshot/mockup table** — keep only if the change is visual (alters what the user sees). Leave the cells blank for the user to fill. Remove it for non-visual changes (refactor, config, build/CI, pure logic).
- **Experiments / notable approaches** — keep only if something genuinely new or experimental was done. Remove it for routine work.
