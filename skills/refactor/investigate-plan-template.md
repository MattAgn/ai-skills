# Investigate plan template

The investigate deliverable — the five-section plan block posted to the tracker in Phase 5, on top of the `save-plan-to-tracker` mechanics. Approach references the Phase 2 target pattern; Safety net carries the Phase 3 verdict (Sufficient / Partial / None); Steps are atomic and each keeps tests green.

```
## 🎯 Automatic plan — Refactor

### 📋 Context
[Goal in 2-3 sentences. Refactor category(ies): readability / perf / structure / dedup / testability / etc.]

[If assumptions were made for lack of ticket detail, list under "Assumptions made:" — one line each.]

### 🛠️ Recommended approach

[3-6 bullets.
- Fowler classification (e.g. "Extract Function + Move Module")
- Target pattern: point to an existing file that already follows it (e.g. "Align with <module>/")
- Preserved invariant: no behaviour change (unless the ticket explicitly says otherwise)
- Prefer the simplest solution — fewer files moved, less surface, reuse existing patterns.]

### 🛡️ Safety net — test coverage

[Current coverage state: Sufficient / Partial / None]

[If Partial or None: list the characterization tests to write BEFORE the refactor — they capture current behaviour, not ideal (Feathers). Mention if a pure-function extraction is a relevant first step to gain testability.]

### 📂 Impacted files

| File | Action | Description |
| ---- | ------ | ----------- |
| path/to/file | Edit / Move / Create / Delete | Short description |

### 📝 Steps

1. [Atomic step — small, keeps tests green]
2. [Next step]

[If characterization tests are required: they are step 1, in a dedicated commit before any refactor.]

### 🧪 Test strategy

| Layer | Scenario | File |
| ----- | -------- | ---- |
| Unit | ... | ... |
| Integration | ... | ... |
| E2E | (if relevant) | ... |

---
*Automatic plan — human validation required*
```
