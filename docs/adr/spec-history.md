# 規格歷史查詢

`specs/domains/<d>/spec.md` 只保留當前已核准的版本，歷史靠 git。

## commit 訊息慣例

| 階段 | 格式 |
|---|---|
| 規格核准併入 | `spec(<domain>): <變更名稱> 核准 — FR-6,14 / AC-7,9,21` |
| 技術設計 | `plan(<change>): <一句話>` |
| 任務拆解 | `task(<change>): 拆成 N 項` |
| 實作 | `feat(<domain>): T-03 <任務名>` |
| 收尾 | `spec(<domain>): <變更名稱> 完成 — FR-6,14 / AC-7,9,21` |

編號寫進 commit 訊息，`git log --grep` 才找得到。

## 常用查詢

```bash
# 某 domain 的規格演進
git log --oneline -- specs/domains/emission/spec.md

# 看某個時間點的完整規格
git show <sha>:specs/domains/emission/spec.md

# 某條 FR/AC 是何時進來、何時被改的
git log -S "FR-14" -- specs/domains/emission/

# 某次變更動了什麼
git log --grep "scope3" --oneline

# 兩個版本的規格差異
git diff <sha1> <sha2> -- specs/domains/emission/spec.md

# 已歸檔變更的完整脈絡
ls specs/changes/archive/
```

## 為什麼不在檔案裡留版本

`domains/<d>/spec.md` 的價值在於「打開就是當前狀態」。
在檔案裡堆疊歷史版本會讓它越來越難讀，而難讀的規格就不會被讀，
不被讀的規格就會失真。歷史交給 git，它本來就擅長這件事。
