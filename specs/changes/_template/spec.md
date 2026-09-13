---
domain: <domain>
base: specs/domains/<domain>/spec.md @ <git short sha>
change: <YYYY-MM>-<domain>-<slug>
---

# <變更名稱>

> 本文件是**完整快照**：包含該 domain 的全部 FR/AC，
> 每條標記本次變更中的角色。implementer 只讀這一份。

## 目的

## 功能需求
FR-1  [UNCHANGED] …
FR-6  [MODIFIED]  <新文字>
      原文：<舊文字>
FR-14 [ADDED]     …

## 非功能需求
NFR-1 [UNCHANGED] …

## 驗收條件
AC-7  [MODIFIED] Given … / When … / Then …  → FR-6
      原文：…
AC-21 [ADDED]    Given … / When … / Then …  → FR-14
AC-1~AC-6, AC-8, AC-10~AC-20  [UNCHANGED]

## 介面影響
是否變更 contracts.md / openapi.yaml？相容擴充或破壞性？遷移方案？

## 影響 domain

## 範圍外

## 待釐清問題
Q-1 …（含暫定假設）

---
## 標記規則
- `[ADDED]` 本次新增 → reviewer 需驗收實作
- `[MODIFIED]` 本次修改 → reviewer 需驗收新行為 **且** 確認舊測試斷言已更新
- `[UNCHANGED]` 本次不變 → reviewer 需**實際執行**對應測試（回歸清單）
- 編號永不重用、永不重排
