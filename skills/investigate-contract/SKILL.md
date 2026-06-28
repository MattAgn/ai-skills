---
name: investigate-contract
description: Read-only guarantee + interactive-vs-autonomous (--auto) behavior for investigate modes. Internal helper invoked by the investigate mode of feat / refactor / fix — not meant to be run on its own.
disable-model-invocation: true
---

Applies whenever a skill runs with `--investigate`. The domain-specific output (a plan or a bug triage) and any labeling live in the calling skill; this is the shared contract.

## Read-only (enforce yourself)

Investigate mode is **strictly read-only**: never edit code, create a branch, or commit. Tools may not block you, so hold the line yourself. Your only writes are to the issue tracker (append the plan/investigation; the calling skill may add labels in autonomous mode). If your environment hard-blocks a tool call (no shell exec, egress, or writes elsewhere), don't work around it.

## Interactive vs autonomous

`--investigate` is interactive by default; `--investigate --auto` is autonomous. The analysis is identical — only the human-in-the-loop differs:

|           | **Interactive** (`--investigate`)                                           | **Autonomous** (`--investigate --auto`)           |
| --------- | --------------------------------------------------------------------------- | ------------------------------------------------- |
| Questions | May ask 1-2 targeted questions when a material ambiguity changes the output | **Never** ask — record the assumption and proceed |
| Use case  | While the user is at their machine                                          | Unattended batch (e.g. nightly)                   |

## Record assumptions instead of blocking

When the ticket is underspecified, don't stall. Interactive: ask only the question(s) that _materially_ change the output; state every minor assumption explicitly in the posted block (label it "Assumptions made") and proceed. Autonomous: never ask — always record and continue. Visible assumptions beat a half-done investigation waiting on input.

## Review before posting (interactive only)

The posted block is the deliverable, but in interactive mode the user is right there — let them shape it first. Draft the plan, **present it in chat, fold in their edits, and write to the tracker only once they approve.** Posting unreviewed and asking for validation after it's on the ticket is backwards when a human can catch a wrong assumption first.

**Present it as clean, rendered Markdown — never the raw tracker block.** Tracker-flavored syntax (collapsible fences, leading separators, footers) renders as literal noise in chat. Translate the block into normal Markdown: real headings instead of collapsible toggles (every section expanded), keep tables/bullets as-is, drop the syntax artifacts and footer. Assemble the tracker-flavored block only when you write it, after approval.

Autonomous (`--auto`) is the exception: no human to ask, so post directly — the tracker write is the deliverable and the only write. Never block a nightly run on approval.
