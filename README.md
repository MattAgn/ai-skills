# ai-skills

A collection of reusable, tool-agnostic [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills) for software-engineering workflows. Each skill lives in `skills/<name>/SKILL.md`.

## Skills

### Workflow orchestrators

| Skill | Purpose |
| ----- | ------- |
| [`feat`](skills/feat/SKILL.md) | Build a feature end-to-end (plan → implement → PR), or plan one read-only with `--investigate`. `--auto` runs unattended. |
| [`fix`](skills/fix/SKILL.md) | Systematic debugging → fix → PR, with disciplined evidence-based hypotheses. `--investigate` triages read-only. |
| [`refactor`](skills/refactor/SKILL.md) | Safe refactoring: tests-first, small green steps. Never refactor without tests, never break green. |
| [`tech-ticket`](skills/tech-ticket/SKILL.md) | Author or sharpen a technical ticket through grill-me scoping + code-grounded planning. |
| [`adr`](skills/adr/SKILL.md) | Write an Architectural Decision Record: interview, score options on a 1–5 matrix, recommend a winner. |

### Shared helpers (invoked by the orchestrators)

| Skill | Purpose |
| ----- | ------- |
| [`grill-me`](skills/grill-me/SKILL.md) | Interview relentlessly until a plan/design is unambiguous. |
| [`understand-project`](skills/understand-project/SKILL.md) | Ground work in existing code before planning or building. |
| [`investigate-contract`](skills/investigate-contract/SKILL.md) | Read-only guarantee + interactive-vs-`--auto` contract for investigate modes. |
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

## Usage

Point your Claude Code skills directory at this repo, or symlink individual skill folders into a project's `.claude/skills/`. The orchestrators (`feat`, `fix`, `refactor`) reference an "issue tracker" abstraction — wire it to whatever tracker your project uses.
