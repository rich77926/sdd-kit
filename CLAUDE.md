# 專案開發規範

> 本檔案每個 session 自動載入。修改前先確認不會與 `.claude/agents/` 的定義衝突。

## 技術棧與目錄

- 後端 Java（`backend/`），依 **domain** 切分 package，不依技術層切分
- 前端 React + Ant Design（`frontend/`），`src/features/<domain>/` 名稱與後端 domain 一致
- 介面契約單一事實來源：`contracts/openapi.yaml`
- `frontend/src/api/generated/` 由 openapi.yaml 產生，**任何情況下都不得手動修改**
- Ant Design 樣式集中在 `frontend/src/theme/`，不寫 inline style

---

## 規格驅動開發

### 目錄

```
specs/
├── domains/<domain>/     常駐規格：spec.md / contracts.md / history.md
└── changes/<change>/     一次性變更：proposal / spec / plan / tasks / STATUS
    └── archive/          已收尾的變更
```

### 規格生命週期

1. `domains/<d>/spec.md` 代表「**已核准的規格**」，不代表已實作。
   實作狀態看 `history.md` 的狀態欄。
2. 規格經我審核通過後，**立即**併入 `domains/`，history 記為 `spec-approved`；
   實作驗收通過後由 spec-merger 改為 `shipped`。
3. **實作階段只讀寫 `changes/<c>/`，不得修改 `domains/`。**
   核准後到收尾之間，domains/ 保持凍結。
4. 變更在實作階段被放棄時，走一次規格流程把該條標記
   `[removed YYYY-MM-DD]` 並註明原因，**不使用 git revert**。
5. 查詢歷史版本用 git，指令見 `docs/adr/spec-history.md`。

### 編號規則

- FR / NFR / AC 編號**永不重用、永不重排**
- 移除的條目改標記保留，不刪整行
- consolidation（清理、重新分組、拆分 domain）**由人主導**，不交給 agent

### 迭代規則

1. 開工前先確認目標 domain 是否已有 `domains/<d>/spec.md`。有就走迭代模式。
2. `changes/<c>/spec.md` 是**完整快照**，每條標記
   `[ADDED]` / `[MODIFIED]` / `[UNCHANGED]`。implementer 只讀這一份。
3. `[UNCHANGED]` 是回歸清單，必須涵蓋所有既有編號，不可遺漏。
4. **同一個 domain 同時只允許一份未收尾的變更。** 需要並行就先合併成一份。
5. 未收尾的變更不得進入下一輪迭代。

---

## 階段與派工

| 階段 | 由誰執行 | 前置條件 |
|---|---|---|
| 需求釐清 | **主線對話，不派 subagent** | — |
| 現況盤點 | Explore | 需求方向已定 |
| 撰寫規格 | spec-writer | 待釐清問題已回答 |
| 併入 domains | **主線** | 我已確認規格 |
| 技術設計 | architect | 規格已核准 |
| 任務拆解 | task-splitter | 我已確認 plan.md |
| 實作 | implementer（每任務一個） | tasks.md 已確認 |
| 驗收 | spec-reviewer | 所有任務完成 |
| 收尾 | spec-merger | 驗收通過 |

### 派工規則

1. **每個階段結束後停下來給我確認**，不要一路跑到底。
2. 需求釐清一律留在主線。subagent 無法向使用者提問，
   把釐清丟給它只會得到一堆假設。
3. 實作依 tasks.md 的依賴圖派工：無依賴的同批可同時派多個 implementer，
   有依賴的等前置回報完成。
4. 任一 subagent 回報「規格矛盾」或「阻塞」時，**停止該批次所有派工**，
   回主線確認後再繼續。不要讓其他 agent 帶著錯誤前提繼續做。
5. spec-reviewer 判定「不可合併」時，把問題清單交回 implementer 修正，
   修完派**新的** spec-reviewer 重審。不沿用同一個——它的 context
   已被前一輪汙染，冷讀的價值就沒了。
6. 每完成一個階段或一項任務，**立刻更新 STATUS.md**，不要等我提醒。
7. 每次開始工作前，先讀 `specs/changes/*/STATUS.md` 確認進度，
   向我確認要接續哪一個變更。

### slash commands

`/spec <domain> <描述>` → `/plan <change>` → `/tasks <change>`
→ `/build <change>` → `/review <change>`

---

## 尺寸規則

建立新變更前先估規模。超過以下任一項，先拆再開 spec：

| 指標 | 門檻 |
|---|---|
| ADDED + MODIFIED 的 FR | > 15 |
| ADDED + MODIFIED 的 AC | > 30 |
| 介面契約 | > 5 |
| 任務數 | > 15 |
| 預估 diff | > 1500 行 |

- 拆分一律**垂直切**，每片能獨立驗收與獨立上線
- **禁止按技術層切分**（不得出現 `xxx-backend` / `xxx-frontend` 這類變更）
- 拆完用 INVEST 檢查，特別確認「單獨上線是否對使用者有價值」
- 不確定該不該拆時，選擇拆

### 不走本流程的情況

- 單一檔案、不動介面、20 行以內
- 修 bug 讓既有 AC 重新成立（直接改 + 補測試）
- 純重構，行為完全不變

修 bug 時若發現是 AC 本身寫錯，則要走迭代流程 MODIFIED 該條 AC。

---

## 架構紀律

1. **慣例要早定，抽象要晚做。** 這兩者方向相反，不要混為一談。
2. 參考實作優先於文件。新的 API endpoint 照既有 domain 的結構寫。
3. **不做規格沒要求的抽象化。** 看到「未來可能需要」就停手。
   通用層、設定機制、plugin 架構一律要有明確的 FR 才做。
4. 架構決策寫進 `docs/adr/`，記錄「為什麼」。
5. 每 3–4 個變更做一次慣例檢查：派唯讀 agent 掃「同一件事有幾種做法」，
   發現分歧就收斂回一種，並把結果補進本檔案。

---

## 禁改清單

- `frontend/src/api/generated/**`
- `specs/domains/**`（僅主線在核准時、spec-merger 在收尾時可改）
- `specs/changes/archive/**`
