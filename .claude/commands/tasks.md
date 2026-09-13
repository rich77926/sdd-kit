---
description: 拆解任務。用法 /tasks <change-slug>
---

委派 task-splitter，讀 `specs/changes/$1/` 的 spec.md 與 plan.md，
產出 tasks.md。

回報後呈現給我：任務總數、依賴圖、哪幾批可平行、FR 覆蓋對照表。
若任務數超過 15 項，先建議我拆分變更，不要直接開工。
