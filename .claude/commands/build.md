---
description: 依 tasks.md 派工實作。用法 /build <change-slug>
---

讀 `specs/changes/$1/tasks.md` 的依賴圖，依批次派工：

1. 第一批一律是契約與 migration 類任務，序列執行。
2. 之後每一批，對無依賴關係的任務同時派多個 implementer。
3. 每批完成後更新 `specs/changes/$1/STATUS.md` 的任務勾選狀態。
4. 任一 implementer 回報「規格矛盾」或「阻塞」時，**停止該批次所有派工**，
   回報給我確認後再繼續。

全部完成後更新 STATUS.md 階段為「待驗收」。
