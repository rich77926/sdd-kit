---
description: 開始一次規格撰寫。用法 /spec <domain> <變更描述>
---

判斷 `specs/domains/$1/spec.md` 是否存在，決定 spec-writer 要跑新 domain 模式還是迭代模式。

然後委派 spec-writer，明確告訴它：
- 目標 domain：$1
- 需求內容：$ARGUMENTS
- 若為迭代模式，change slug 用 `<YYYY-MM>-$1-<描述 slug>`

spec-writer 回報後：
1. 把「待釐清問題」原文呈現給我，等我回答，不要自行假設。
2. 我回答完，再請 spec-writer 更新規格。
3. 我確認規格內容後，**你**（主線）負責把 changes/<c>/spec.md 的內容併入
   `specs/domains/$1/spec.md`（清除狀態標記），並在 history.md 新增一列，
   狀態填 `spec-approved`。之後 domains/ 進入凍結，直到收尾。
