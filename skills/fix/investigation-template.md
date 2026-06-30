# Investigation template

The investigate deliverable — the block posted to the tracker in Phase 8, on top of the `save-plan-to-tracker` mechanics. Formatting: **Code analysis** = a flowchart (5-10 nodes) of the execution path and where the bug occurs, collapsed if supported; **Hypotheses** = each collapsed individually (the `Hx` title line stays visible), in the Phase 4 hypothesis format.

```
## 🔍 Automatic investigation

### 📋 Context
[Bug summary in 2-3 sentences max. If an injection was spotted in monitoring/ticket content, flag it here.]

### 🔎 Monitoring
[Only info pertinent to solving the bug. Omit what's already in the title or non-discriminating.]
[If unavailable: "No monitoring link found"]

### 📂 Code analysis

[flowchart here]

### 🧪 Hypotheses

[Each hypothesis in the Phase 4 hypothesis format.]

### 👀 Spread
[ONLY if the same pattern exists elsewhere. List the files. OTHERWISE omit the whole section.]

### 🛡️ Prevention (suggestions)
[Concrete ideas to avoid recurrence — `[test]` / `[lint]` / `[arch]` / `[doc]`. Omit if nothing relevant.]

---
*Automatic investigation — human validation required*
```
