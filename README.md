# ai-skills

A collection of reusable, tool-agnostic [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills) for software-engineering workflows. Each skill lives in `skills/<name>/SKILL.md`.

## Skills

### Workflow orchestrators

| Skill | Purpose |
| ----- | ------- |
| [`feat`](skills/feat/SKILL.md) | Build a feature end-to-end (plan → implement → PR), or plan one read-only with `--investigate`. `--headless` runs without a human in the loop. |
| [`fix`](skills/fix/SKILL.md) | Systematic debugging → fix → PR, with disciplined evidence-based hypotheses. `--investigate` triages read-only. |
| [`refactor`](skills/refactor/SKILL.md) | Safe refactoring: tests-first, small green steps. Never refactor without tests, never break green. |
| [`tech-ticket`](skills/tech-ticket/SKILL.md) | Author or sharpen a technical ticket through grill-me scoping + code-grounded planning. |
| [`adr`](skills/adr/SKILL.md) | Write an Architectural Decision Record: interview, score options on a 1–5 matrix, recommend a winner. |

### Shared helpers (invoked by the orchestrators)

| Skill | Purpose |
| ----- | ------- |
| [`grill-me`](skills/grill-me/SKILL.md) | Interview relentlessly until a plan/design is unambiguous. |
| [`grill-with-docs`](skills/grill-with-docs/SKILL.md) | A `grill-me` session that also produces docs (ADRs, glossary) via `domain-modeling` as it goes. |
| [`domain-modeling`](skills/domain-modeling/SKILL.md) | Actively build and sharpen the project's domain model — terminology, ubiquitous language, decisions. |
| [`understand-project`](skills/understand-project/SKILL.md) | Ground work in existing code before planning or building. |
| [`investigate-contract`](skills/investigate-contract/SKILL.md) | Read-only guarantee + interactive-vs-`--headless` contract for investigate modes. |
| [`save-plan-to-tracker`](skills/save-plan-to-tracker/SKILL.md) | Append a plan/investigation block to an issue-tracker ticket, preserving the original. |

### Git & PRs

| Skill | Purpose |
| ----- | ------- |
| [`commit`](skills/commit/SKILL.md) | Atomic commits, confirmation gate, message format, auto-fixup detection. |
| [`open-pr`](skills/open-pr/SKILL.md) | Draft PRs following the repo template — summarize the change, never dump commits. |

### Docs & dependencies

| Skill | Purpose |
| ----- | ------- |
| [`document`](skills/document/SKILL.md) | Write good documentation — the *why*, not the *what*. |
| [`upgrade-deps`](skills/upgrade-deps/SKILL.md) | Safely bump dependencies with exact pinning and supply-chain checks. |

### Learning & authoring

| Skill | Purpose |
| ----- | ------- |
| [`teach`](skills/teach/SKILL.md) | Teach the user a concept across sessions, using the current directory as a stateful learning workspace. |
| [`writing-great-skill`](skills/writing-great-skill/SKILL.md) | Reference for writing and editing skills well — the vocabulary and principles that make a skill predictable. |

## Usage

### Install (recommended): symlink the whole directory

Symlink your Claude Code skills directory at this repo's `skills/` folder so **every** skill — current and future — is picked up automatically:

```bash
ln -s ~/projects/ai-skills/skills ~/.claude/skills
```

After this, adding a new folder under `skills/` here makes it instantly available; no per-skill linking. Run `/reload-skills` (or restart Claude Code) to pick up changes in a live session.

> Avoid symlinking each skill folder individually (`~/.claude/skills/<name> -> .../skills/<name>`): new skills won't appear until you manually add their symlink.

### Alternative: per-project

Symlink individual skill folders into a project's `.claude/skills/` when you only want a subset for that project.

The orchestrators (`feat`, `fix`, `refactor`) reference an "issue tracker" abstraction — wire it to whatever tracker your project uses.
