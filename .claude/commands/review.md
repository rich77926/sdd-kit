---
description: 驗收。用法 /review <change-slug>
---

委派一個**全新的** spec-reviewer（不要沿用先前跑過的），
對照 `specs/changes/$1/spec.md` 冷讀完整 diff。

特別提醒它 UNCHANGED 區塊必須實際跑測試，不能只看程式碼沒動就假設沒壞。

回報後：
- 判定「不可合併」→ 把問題清單交回 implementer 修正，修完再派**另一個新的**
  spec-reviewer 重審。不要沿用同一個。
- 判定「可合併」→ 更新 STATUS.md 為「驗收通過」，然後委派 spec-merger 收尾。
