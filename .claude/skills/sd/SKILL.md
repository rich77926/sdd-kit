---
name: sd
description: 依已核准的需求規格書（01）與系統分析書（02）撰寫或修訂系統設計書（docs/<domain>/03-系統設計書.md）與 API 契約 contracts/openapi.yaml。用法 /sd <domain>。當某個 domain 的 01、02 都已核准、分期已定案，要定 API 端點、資料表、程式結構、測試計畫與開發項目清單 D-n 時使用。這是文件驅動流程的第三階段，前一步是 sa，下一步是 dev。
---

# /sd — 系統設計

## 參數

呼叫時傳入的 args 就是 `<domain>`（例：`passport`）。下文的 `<domain>` 一律代換成它。
args 沒有給 domain 時，先問使用者，不要猜。

## 步驟

1. 確認 01、02 本版次皆為「已核准」，而且 02 的分期已定案，否則停下。
2. 派 architect（**SD 模式**），產出或修訂 `docs/<domain>/03-系統設計書.md` 與 `contracts/openapi.yaml`。
3. 回報後呈現給我：
   - API 端點與錯誤碼
   - 資料表與 migration
   - 開發項目清單：總數、順序、可平行項
   - 是否為破壞性變更
   - 待決事項

   開發項目超過 15 項時，先建議回到 `/sa <domain>` 調整分期。
4. 我核准後：
   - 03 的文件狀態與本版次修訂紀錄改為「已核准」
   - commit：`docs(<domain>): 03 系統設計書 vX.Y 核准`（連同 openapi.yaml）

停下來。下一步是 `/dev <domain>`。
