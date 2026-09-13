# <domain 名稱> 規格演進

狀態說明：
- `spec-approved` — 規格已核准併入，尚未實作或實作中
- `shipped` — 實作完成並通過驗收
- `abandoned` — 核准後放棄，相關條目已標記 removed

| 日期 | 變更 | 影響編號 | 狀態 |
|---|---|---|---|
| YYYY-MM-DD | init | FR-1~FR-n, AC-1~AC-m | spec-approved |

---

## 查詢歷史版本
```bash
git log --oneline -- specs/domains/<domain>/spec.md
git show <sha>:specs/domains/<domain>/spec.md
git log -S "FR-14" -- specs/domains/<domain>/
```
