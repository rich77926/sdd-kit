# <domain 名稱> 對外契約

本 domain 暴露給其他 domain 與前端的介面。變更此檔案等同破壞性風險評估點。

## HTTP API
對應 `contracts/openapi.yaml` 的路徑：
- `GET  /api/v1/<domain>/...`
- `POST /api/v1/<domain>/...`

## 資料型別
欄位命名、大小寫、null 與缺欄位的語意差別，在此寫死。

## 錯誤形狀
沿用 `backend/.../common/` 的統一錯誤結構，本 domain 專屬錯誤碼：
| 錯誤碼 | 意義 | HTTP status |
|---|---|---|

## 事件 / 訊息
（如有）

## 被誰依賴
- <domain>：唯讀依賴 …
- frontend/src/features/<domain>

## 變更紀錄
| 日期 | 變更 | 相容性 |
|---|---|---|
