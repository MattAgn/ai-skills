---
name: investigate-contract
description: Read-only guarantee + interactive-vs-autonomous (--auto) behavior for investigate modes. Internal helper invoked by the investigate mode of feat / refactor / fix — not meant to be run on its own.
disable-model-invocation: true
---

Applies whenever a skill runs with `--investigate`. The domain-specific output (a plan, or a bug triage) and any labeling live in the calling skill; this is the shared contract.

## Read-only (enforce yourself)

Investigate mode is **strictly read-only**: never edit code, never create a branch, never commit. Nothing at the tool level necessarily blocks you, so hold the line yourself. The only writes you make are to the issue tracker (append the plan/investigation; the calling skill may also add labels in autonomous mode). If your environment enforces hard limits (no shell exec, no egress, no writes to other services) and a tool call is blocked, do not work around it.

## Interactive vs autonomous

`--investigate` is interactive by default; `--investigate --auto` is autonomous. The analysis is identical — only the human-in-the-loop differs:

|           | **Interactive** (`--investigate`)                                           | **Autonomous** (`--investigate --auto`)           |
| --------- | --------------------------------------------------------------------------- | ------------------------------------------------- |
| Questions | May ask 1-2 targeted questions when a material ambiguity changes the output | **Never** ask — record the assumption and proceed |
| Use case  | While the user is at their machine                                          | Unattended batch (e.g. nightly)                   |

## Record assumptions instead of blocking

When the ticket is underspecified, don't stall. Interactive: ask only the question(s) that _materially_ change the output; for everything minor, state the assumption explicitly in the posted block (label it "Assumptions made") and proceed. Autonomous: never ask — always record the assumption and continue. A reader seeing your assumptions is far more useful than a half-done investigation waiting on input.

## Review before posting (interactive only)

The posted block is the deliverable — but in interactive mode the user is sitting _with_ you, so let them shape it before it lands on the ticket. Draft the plan, **present it in chat, fold in their edits, and write to the tracker only once they approve.** Posting an unreviewed plan and asking for validation _after_ it's already on the ticket is backwards when a human is right there to catch a wrong assumption or a missing constraint first.

**Present it as clean, rendered Markdown — never the raw tracker block.** Tracker-flavored syntax (collapsible fences, leading separators, footers) renders as literal noise in chat. So for the in-chat review, translate the block into normal Markdown: real headings instead of collapsible toggles (show every section expanded — the user wants to see all of it), keep the tables/bullets as-is, drop the syntax artifacts and the footer. Only assemble the tracker-flavored block at the moment you write it, after approval.

Autonomous (`--auto`) is the exception: there's no human to ask, so post directly — the tracker write is the deliverable and the only write. Never block a nightly run waiting on approval.
