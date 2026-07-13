---
name: commit
description: Mandatory skill for all commits. Use every time before creating any git commit.
---

# Generating Commit Messages

## Mandatory Process

Before any `git commit`:

1. Run `git diff --staged` and analyze the staged changes thoroughly.
2. Split into atomic commits — one per coherent change, with its tests. A single feature spanning multiple layers (utility + hook + component + test) is ONE change; do not split by layer. Split only when changes are truly unrelated (e.g. a bug fix + a new feature + a docs update).
3. Run the project's checks (tests for the relevant files, linter, formatter, type-checker), and stage any formatter changes.
4. Write a message in the format below.
5. Never commit as a side effect of making changes — only when the user explicitly invokes this skill or says "commit".

## Confirmation Before Committing

Always present the full plan and wait for explicit approval before running `git commit`. For each commit (regular or fixup), show:

- The commit message (summary, plus description if applicable)
- The files included
- For a split: which files go in which commit, in what order
- For a fixup: the target commit SHA and why

Wait for approval; if the user requests changes, adjust and re-present. This applies to regular commits, fixups, and commits triggered during the open-pr workflow.

**Exception — autonomous callers.** When invoked from a skill that has established there is no per-commit review (an autonomous run — `--unattended`, or an attended run without `--review-commits` — see `run-mode`), skip the confirmation and commit directly. The message format and atomic-split rules below still apply.

## Auto-Fixup Detection

**Skip under `--auto-merge`.** Fixups exist to keep branch history readable for PR review; `--auto-merge` skips that review (see `run-mode`), so the tidying earns nothing — commit normally.

Before creating a new commit, check whether the staged changes belong in a recent commit on the current branch:

1. List branch commits: `git log main..HEAD --oneline`.
2. For each staged file, check `git log main..HEAD -- <file>` for recent modifications.
3. If a change clearly amends or extends a previous commit (same file, nearby lines, related logic — e.g. fixing a typo or adding a missing import from a prior commit), suggest a fixup: "This change to `<file>` looks like it should be fixup'd into `<sha> <message>`. Want me to fixup instead?"

When fixup is confirmed (each step needs user confirmation):

1. `git commit --fixup=<sha>`
2. `GIT_SEQUENCE_EDITOR=true git rebase --interactive --autosquash main` to squash immediately.

Otherwise, proceed with a normal commit.

## Required Commit Message Format

For commits you create on a branch, use this fixed format. Do not derive it from `main`'s history (see below).

**Shape:** `<verb>: <lowercase description>`

- `<verb>` (lowercase): one of `chore` `feat` `fix` `refactor` `docs` `tests`
- Colon and single space after the verb
- `<description>` starts lowercase, present-tense imperative ("add caching", not "added" or "Add")

**Example:** `chore: add error monitoring`

**Summary line:** must be under 72 characters (count before committing); be specific. If too long, shorten — abbreviate scope, drop filler words.

**Detailed description:** add only when not obvious to a junior developer. Explain WHAT only if the commit touches more than 3 files; always explain WHY and any important context.

## Co-Authored-By

Add a `Co-Authored-By` trailer only when the assistant wrote the code being committed. If the user wrote the changes and the assistant is just committing, omit it.
