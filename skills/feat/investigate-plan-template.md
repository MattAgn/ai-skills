# Investigate plan template

The deliverable format for an `--investigate` run — composed in Phase 3, posted in Phase 4 via `save-plan-to-tracker`. Approach references the Phase 2 similar feature; Steps are atomic and each leaves tests green; Test strategy reuses Phase 2 patterns.

```
## 🎯 Automatic plan — Feature

### 📋 Context
[Need summarized in 2-3 sentences. Feature type: new screen / module / extension / component.]

[If assumptions were made for lack of detail in the ticket, list them here under "Assumptions made:" — one line each.]

### 🛠️ Recommended approach

[3-6 bullets. Reference the similar pattern found in the code (e.g. "Follow the same pattern as <module>/"). Prefer the simplest solution — reuse existing components/hooks, fewer files touched, no needless abstractions.]

### 📂 Impacted files

| File | Action | Description |
| ---- | ------ | ----------- |
| path/to/file | Create | New screen X |
| path/to/other | Edit | Add route |

### 📝 Steps

1. [Atomic step 1]
2. [Atomic step 2]

### 🧪 Test strategy

| Layer | Scenario | File |
| ----- | -------- | ---- |
| Unit | ... | ... |
| Integration | ... | ... |
| E2E | (if relevant) | ... |

---
*Automatic plan — human validation required*
```
