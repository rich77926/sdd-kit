---
description: 依已核准的規格產出技術設計。用法 /plan <change-slug>
---

委派 architect，讀 `specs/changes/$1/spec.md` 與相關 domain 的 contracts.md，
產出 `specs/changes/$1/plan.md`。

回報後把「介面契約」清單呈現給我確認，特別確認：
- 每組契約的欄位命名與大小寫是否寫死
- 是否為破壞性變更，若是，遷移方案是什麼

我確認後才進入下一階段。
