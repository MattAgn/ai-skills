---
name: commit
description: MANDATORY skill for ALL commits. Must be used EVERY TIME before creating any git commit. No exceptions.
---

# Generating Commit Messages

## Mandatory Process

**BEFORE ANY git commit COMMAND:**

1. **ALWAYS** run `git diff --staged` first to see changes
2. **ALWAYS** analyze the staged changes thoroughly
3. **ALWAYS** split the code changes into atomic commits, one per coherent / cohesive change. Tests for a change should be in the same commit as the change itself. A single feature spanning multiple layers (utility + hook + component + test) is ONE cohesive change — do not split a feature by layer. Only split when changes are truly unrelated (e.g. a bug fix + a new feature + a docs update)
4. **ALWAYS** run the project's checks (tests for the relevant files, linter, formatter, type-checker). Stage any formatter changes before committing.
5. **ALWAYS** generate a commit message following the format below
6. **NEVER** commit automatically as a side effect of making code changes. Only commit when the user explicitly invokes the commit skill or says "commit".

## Confirmation Before Committing

User trust requires seeing the plan before execution. Always present the full commit plan and wait for explicit approval before running any `git commit` command.

**For each commit (regular or fixup), present:**

- The commit message (summary + description if applicable)
- The list of files included
- If splitting into multiple commits: the full split plan (which files go in which commit, in what order)
- If fixup: which commit SHA it targets and why

**Then ask the user to confirm.** Do not proceed until they approve. If they request changes to the message or grouping, adjust and re-present.

This applies equally to regular commits, fixups, and any commits triggered during the open-pr workflow.

## Auto-Fixup Detection

Before creating a new commit, check whether the staged changes should be fixup'd into a recent commit on the current branch.

**Process:**

1. Run `git log main..HEAD --oneline` to list all commits on the branch since diverging from the base branch
2. For each staged file, check `git log main..HEAD -- <file>` to see if it was modified in a recent branch commit
3. If a staged change clearly amends or extends code from a previous commit (same file, nearby lines, related logic — e.g. fixing a typo introduced in a prior commit, adding a missing import for a recently added module), suggest fixup'ing into that commit
4. Present the suggestion: "This change to `<file>` looks like it should be fixup'd into `<sha> <message>`. Want me to fixup instead of creating a new commit?"

**When fixup is confirmed:**

1. Run `git commit --fixup=<sha>` (with user confirmation)
2. Then run `GIT_SEQUENCE_EDITOR=true git rebase --interactive --autosquash main` to squash immediately (with user confirmation before the rebase)

If the change doesn't clearly relate to a previous commit, proceed with a normal new commit.

## Required Commit Message Format

The format for commits **you create on a branch** is fixed and described below. Do not derive it from `main`'s history — see "Do not copy main's format" below for why.

**Shape:** `<verb>: <lowercase description>`

- `<verb>` is one of: `chore` `feat` `fix` `refactor` `docs` `tests` (lowercase)
- A colon and single space follow the verb
- `<description>` starts lowercase and uses present-tense imperative ("add caching", not "added caching" or "Add caching")

**Example:** `chore: add error monitoring`

1. **Summary line** (STRICTLY under 72 characters — count them before committing)

   1. Follow the `<verb>: <lowercase description>` shape above
   2. Be specific and descriptive
   3. If your message exceeds 72 chars, shorten it. Abbreviate the scope, drop filler words

### Do not copy main's format

The history on `main` may be formatted differently (e.g. `[Chore] Add X` — bracketed, capitalized) because squash-merge commits often take their message from the PR title, which follows a different convention. **Do not imitate that.** Individual commits on a working branch always use the lowercase `chore: add x` shape above, regardless of what the history on `main` looks like.

2. **Detailed description**
   1. ONLY add one if not obvious from a junior developer's perspective
   2. Explain WHAT was changed only if the commit concerns more than 3 files
   3. Explain WHY it was changed
   4. Include any important context

## Co-Authored-By

Only add a `Co-Authored-By` trailer when the assistant actually wrote the code being committed. If the user wrote the changes themselves (and the assistant is just committing), do not add it.
