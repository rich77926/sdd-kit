# SDD 開發流程手冊

Java + React/Ant Design 專案，用 Claude Code 多 agent 執行規格驅動開發。
本文件是流程總覽；設定檔在 `.claude/`，範本在 `specs/*/\_template/`。

---

## 1. 這套流程要解決什麼

單一 agent 做複雜功能有三個固定失敗模式：

- **規格漂移** — 實作到一半忘記約束，架構悄悄變成另一個樣子
- **契約偏移** — 後端回 `{notif_type:"comment"}`、前端等 `{type:"COMMENT"}`，
  各自測試都過，整合才炸
- **無聲回歸** — 迭代時弄壞舊功能，沒人發現，因為沒有回歸清單

對應的三個機制：

| 失敗模式 | 機制 |
|---|---|
| 規格漂移 | 規格是磁碟上的檔案，agent 每次從檔案讀，不靠對話記憶 |
| 契約偏移 | `plan.md` 的介面契約 + `openapi.yaml` CI 檢查 |
| 無聲回歸 | 完整快照的 `[UNCHANGED]` 標記 = 強制回歸清單 |

### 什麼時候不要用

- 單一檔案、不動介面、20 行以內的改動
- 純重構、修 bug（讓既有 AC 重新成立）
- 個人專案的探索期

流程的成本只在**多模組、會動介面、或需要平行實作**時才回本。

---

## 2. 前置：先做骨架，不做完整架構

AI 讓改動變便宜，但讓**不一致**變貴。Agent 靠 pattern matching 工作，
它會複製它在 codebase 裡看到的東西——包括你的三種錯誤處理寫法。

所以：**慣例要早定，抽象要晚做。** 這兩件事方向相反。

| 先定（改起來貴，agent 會亂猜） | 延後（agent 很願意過度設計） |
|---|---|
| 模組邊界與資料夾結構 | 功能層的抽象與介面泛化 |
| 核心資料模型：identity、租戶、時間 | 外部服務 adapter 通用化 |
| 錯誤形狀與錯誤碼規則 | 效能最佳化、快取層 |
| 認證授權模型 | 設定驅動、plugin 機制 |
| 測試分層與擺放位置 | 「未來可能換 DB」的抽象層 |
| 命名與 API 慣例 | 事件匯流排、CQRS |

### 做法：一片會走路的骨架

不要寫架構文件叫 AI 照做。改成親自（或高度盯著）做完一片最薄的端到端切片：

1. 挑一個最無聊、但會經過所有層的功能，通常是某 entity 的建立與查詢
2. 從 API 入口、驗證、授權、service、資料存取、migration，到單元與整合測試，全打通
3. 只要能跑，不用漂亮。但命名、錯誤處理、測試寫法要是你真心想長期用的樣子
4. **這片你自己看過每一行才算完成**

之後每個 agent 都會 Explore 到這片並模仿它。一片能跑的參考實作，
對 agent 的約束力遠大於二十頁架構文件。

> **隱性風險**：AI 移除了「這個架構很難用」的摩擦訊號。以前架構錯了，
> 工程師第二週就會抱怨；現在 agent 會毫無怨言地把爛架構實作到第八個 feature。
> 骨架階段親自看過每一行，是你最後一次能便宜地感受架構好不好用的時機。

---

## 3. Repo 結構

前後端**放同一個 repo**。這不是偏好問題：契約變更必須是一個 atomic commit，
spec-reviewer 才看得到完整 diff、才抓得到契約偏移。拆兩個 repo，
reviewer 只能看半邊。

該拆開只有三種情況：前端被第三方使用、發版節奏差很多、不同團隊要權限隔離。

```
<service>/
├── CLAUDE.md
├── .claude/
│   ├── agents/               七個 agent 定義
│   ├── commands/             五個 slash command
│   └── agent-memory/         (執行後自動產生，project scope)
│
├── specs/
│   ├── domains/<domain>/     常駐規格
│   │   ├── spec.md           FR/AC 全集，編號永不重用
│   │   ├── contracts.md      對外介面
│   │   └── history.md        演進紀錄 + 實作狀態
│   └── changes/<change>/     一次性變更
│       ├── proposal.md  spec.md  plan.md  tasks.md  STATUS.md
│       └── archive/          已收尾
│
├── docs/adr/                 架構決策紀錄
├── contracts/openapi.yaml    ← 介面單一事實來源
│
├── backend/src/main/java/com/<org>/<proj>/
│   ├── <domain>/             ← 依領域切，不依層切
│   │   ├── api/  domain/  service/  repository/
│   ├── common/               錯誤形狀、tenant context、分頁、稽核
│   └── config/               Security(OIDC)、CORS、Jackson
│
├── frontend/src/
│   ├── api/generated/        ← 由 openapi.yaml 產生，禁止手改
│   ├── features/<domain>/    ← 名稱對齊 backend domain
│   ├── components/  theme/  lib/
│
└── deploy/
    ├── docker/               前後端各一 Dockerfile
    └── helm/                 一份 chart，兩個 Deployment
```

### 兩個對 agent 特別重要的細節

**`<domain>` 名稱前後端必須一致。** Explore 靠 Glob/Grep 找東西，
名稱對齊它一次撈到兩側；不對齊它會漏一半，然後 implementer 在只看到
半邊的情況下動工。

**後端依領域切不依層切。** 不要 `controller/` `service/` `repository/`
三個頂層資料夾——一個變更的異動會散在三處，任務拆解的「檔案」欄位很醜，
worktree 平行實作也更容易撞。

### 讓契約真的被強制

`plan.md` 的介面契約只是文字承諾。加這三步讓 build 擋住它：

1. 後端用 springdoc 產生 `openapi.yaml`，commit 進 `contracts/`
2. 前端用 openapi-typescript / orval 從同一份產生 `api/generated/`
3. CI 重新產生 openapi.yaml 並與 committed 版本 diff，不一致就 fail

Agent 改欄位名稱毫不猶豫，所以這步在多 agent 開發下的投報率比人工開發高得多。

### 其他配套

- CI 加 **path filter**：`frontend/**` 的改動不要觸發 Java build
- Ant Design theme 集中在 `theme/`，用 ConfigProvider design token。
  放任 agent 各自寫 inline style，三個變更後就會有四種藍色
- `api/generated/` 進 CLAUDE.md 禁改清單。不然 implementer 遇到型別對不上時，
  最省力的解法就是去改那個檔案

---

## 4. 規格的兩層模型

這是整套流程的核心設計。

```
specs/domains/<d>/     常駐 · 永久形狀 · 對齊 backend package
specs/changes/<c>/     一次性 · 變更形狀 · 做完歸檔
```

只用其中一層都會壞：

- **只留 changes/** → 要知道現況得把二十份變更從頭疊起來。
  人不會做，agent 更不會，它會讀最近三份就開工。
- **只留 domains/** → reviewer 看不出這輪改了什麼、什麼該維持不變，
  回歸驗收沒有依據。

### domains/ 的不變量

`domains/<d>/spec.md` = **已核准的規格**，不是已上線的功能。
實作狀態記在 `history.md` 的狀態欄：

| 狀態 | 意義 |
|---|---|
| `spec-approved` | 規格已核准併入，尚未實作或實作中 |
| `shipped` | 實作完成並通過驗收 |
| `abandoned` | 核准後放棄，相關條目已標 removed |

這一欄是必要的。少了它，讀 spec.md 的人分不出 FR-14 是已上線還是只是計畫。

### changes/spec.md 是完整快照，不是 delta

implementer 讀一份自足的文件，不必自己把 base 和 delta 疊起來——
agent 疊這個尤其不可靠。但完整快照會丟失回歸訊號，
所以每條要帶狀態標記：

```markdown
---
domain: emission
base: specs/domains/emission/spec.md @ a3f8c21
change: 2026-09-emission-scope3
---

FR-1  [UNCHANGED] 系統必須支援係數 CSV 上傳
FR-6  [MODIFIED]  係數支援 scope1、scope2、scope3
      原文：係數僅支援 scope1、scope2
FR-14 [ADDED]     系統必須支援 scope3 類別 1-8 的係數輸入

AC-7  [MODIFIED] …（原文：…）
AC-21 [ADDED]    Given … / When … / Then …
AC-1~AC-6, AC-8, AC-10~AC-20  [UNCHANGED]
```

一份文件同時具備兩種性質：

- implementer 讀它 = 完整規格
- reviewer `grep ADDED|MODIFIED` = 本次驗收項；`grep UNCHANGED` = 回歸清單
- `base:` 記來源 commit，事後查得到「當初照哪個版本做的」

### 三條硬規則

1. **編號永不重用、永不重排。** 移除的條目改標記 `[removed YYYY-MM-DD]`，
   保留原文不刪整行。不然所有歷史引用會錯位。
2. **UNCHANGED 必須涵蓋所有既有編號。** 這是回歸清單，漏一條就是一條
   從此不再被驗證——而且是安靜地不再被驗證。
3. **同一 domain 同時只允許一份未收尾的變更。** 兩份並行會撞編號
   （兩邊都以為下一個是 FR-15），UNCHANGED 清單也會互相失效。

---

## 5. 完整流程

```
  需求
   │
   ├─ [主線] 需求釐清 ──────────── 不派 subagent
   │
   ├─ Explore ───────────────────── 現況盤點
   │
   ├─ spec-writer ────────────────→ changes/<c>/spec.md
   │     新 domain 模式 / 迭代模式（自動判斷）
   │
   ├─ [人] 審核規格 ★
   ├─ [主線] 併入 domains/ ──────── history 記 spec-approved
   │                                此後 domains/ 凍結
   │
   ├─ architect ──────────────────→ changes/<c>/plan.md
   ├─ [人] 確認介面契約 ★
   │
   ├─ task-splitter ──────────────→ changes/<c>/tasks.md
   ├─ [人] 確認任務 ★
   │
   ├─ implementer × N ────────────  依依賴圖批次派工
   │     worktree 隔離，可平行
   │
   ├─ spec-reviewer ──────────────  ADDED/MODIFIED 驗收 + UNCHANGED 回歸
   │     不可合併 → 修正 → 派【新的】reviewer 重審
   │
   └─ spec-merger ────────────────  history 改 shipped、歸檔、處理放棄項
```

★ = 停下來等人確認，不要一路跑到底。規格與設計是人要簽字的東西。

### 為什麼是「核准即併入」而不是「上線後併入」

實作階段不需要讀 `domains/`（它讀 `changes/<c>/spec.md`），
所以提前併入不影響 pipeline。好處是 domain spec 永遠反映最新的
已核准狀態，不會落後。代價是要靠 `history.md` 的狀態欄區分
「已核准」與「已上線」——一欄就解決。

### 被放棄的變更

規格核准後併進 domain，實作才發現做不到或被砍掉。
**不要 revert**，那會讓 history 斷掉。改成標記：

```
FR-14 [removed 2026-09-20] 系統必須支援 scope3 類別 1-8
      放棄原因：金流商不支援，改由 2026-10-scope3-manual 處理
```

### slash commands

```
/spec <domain> <描述>     規格
/plan <change>            設計
/tasks <change>           拆解
/build <change>           派工實作
/review <change>          驗收 + 收尾
```

---

## 6. 尺寸規則

決定尺寸的只有兩件事：**這份 spec 人會不會真的讀完，
reviewer 能不能在一次冷讀裡看完整個 diff。**

這兩件事一破，整條 pipeline 的價值就沒了。

| 指標 | 舒適區 | 該拆了 |
|---|---|---|
| ADDED+MODIFIED 的 FR | 5–12 | > 15 |
| ADDED+MODIFIED 的 AC | 10–25 | > 30 |
| 介面契約 | 1–4 組 | > 5 |
| 任務數 | 5–12 | > 15 |
| 總 diff | < 800 行 | > 1500 行 |
| 實作時間 | 1–5 天 | > 1.5 週 |

**介面契約數量最靈敏。** 超過五組代表橫跨太多模組邊界，
對不齊的風險非線性上升。看到這個爆掉，優先於其他指標拆。

拆太小的成本是流程開銷（可逆），拆太大的成本是規格漂移與漏審（不可逆）。
不確定時一律往小的切。

### 怎麼切：垂直，不要水平

錯誤示範是切成 `checkout-backend` 和 `checkout-frontend`。
這樣切出來沒有一個能單獨驗收，AC 寫不出來，整合風險留到最後。

用 SPIDR：

- **S**pike — 不確定的技術先獨立成調查變更，只產出結論
- **P**ath — 依流程路徑：先做信用卡分期，超商分期另開
- **I**nterface — 先做 API 與後台，前端 UI 下一片
- **D**ata — 先支援 3/6 期，12/24 期之後再加
- **R**ule — 先做基本費率，促銷免息規則另開

切完用 INVEST 驗一次。**V（單獨上線是否有價值）不過關就是切錯了**，
通常代表不小心切成水平層。

### 校準

上面的數字是起點。跑三四個變更後回頭看：

- reviewer 開始回報「AC-x 未實作」但其實有做 → diff 太大讀漏了，門檻往下調
- 同一週開了五個變更且互相依賴 → 切太細，門檻往上放

真正的指標是**你自己讀 spec.md 時有沒有在跳著看**。有的話，
不管數字多漂亮，那份都太大了。

---

## 7. 進度記錄與跨 session 接續

### 狀態放檔案，不放 session

Session resume 不能當記錄機制：會被 compaction 壓縮、
30 天後 transcript 被清掉、換台機器就沒了。

三層記錄：

1. **`STATUS.md`** — 每個變更一份，階段、任務勾選、決策紀錄、阻塞、放棄項
2. **`history.md`** — 每個 domain 一份，規格演進與實作狀態
3. **git commit** — 訊息帶階段標記與編號，見 `docs/adr/spec-history.md`

`STATUS.md` 的「決策紀錄」最容易被省略但最有價值。三週後你不會記得
為什麼手續費放後端算。

### 下次怎麼接上

```bash
cd <repo>
claude
> 繼續 2026-09-emission-scope3
```

CLAUDE.md 每 session 自動載入，其中的規則會讓它先讀 STATUS.md。
**這是主要方式**，不依賴任何 session 狀態。

需要撿回上次對話脈絡時才用 session：

```bash
claude -c              # 接續此目錄最近一次對話
claude -r              # 選單挑一個 session
claude --fork-session  # 接續但開新 session id
```

在 session 裡用 `/rename` 取名，之後 `-r` 選單才認得出來。

### Session resume 的限制

- **subagent 只能在同一 session 內 resume。** transcript 在
  `~/.claude/projects/{project}/{sessionId}/subagents/agent-{id}.jsonl`。
  內建的 Explore 與 Plan 是一次性的，不回傳 agent ID，resume 不了。
- **預設 30 天後清掉**（`cleanupPeriodDays`）。長週期的變更一定會踩到。

Agent memory（`.claude/agent-memory/`）不受影響，但它記的是
「這專案測試怎麼跑」這類慣例，不是變更進度，兩者不要混用。

---

## 8. Agent 一覽

| Agent | model | 工具 | 職責 |
|---|---|---|---|
| `Explore` | haiku | 唯讀 | 現況盤點（覆寫內建，僅為省成本） |
| `spec-writer` | opus | 唯讀+Write | 新 domain 規格 / 迭代完整快照 |
| `architect` | opus | 唯讀+Write | 技術設計與介面契約 |
| `task-splitter` | sonnet | 唯讀+Write | 拆成可獨立驗證的任務 |
| `implementer` | sonnet | 全開 + worktree | 單一任務 test-first 實作 |
| `spec-reviewer` | opus | 唯讀（拿掉 Edit/Write） | 冷讀 diff 驗收 + 回歸 |
| `spec-merger` | haiku | 讀寫 | 狀態更新與歸檔 |

### 設計取捨

- **spec-writer 無法提問。** Claude Code 把 `AskUserQuestion` 從所有 subagent
  拿掉了，所以它把疑問寫成「待釐清問題」丟回主線。
  **需求釐清一定要留在主線對話。**
- **spec-reviewer 同時用 `tools` 白名單與 `disallowedTools`** 排除 Edit/Write。
  兩者同時設定時是先套 disallowedTools 再解析 tools，雙保險。
  它**沒有**設 `memory`——啟用 memory 會自動打開 Read/Write/Edit，
  與唯讀意圖衝突，不如不設。
- **implementer 設 `isolation: worktree`**，平行跑不會互蓋。
  但也因此看不到其他 implementer 尚未合併的成果——所以 architect
  的介面契約必須寫死。
- **重審換新 agent。** 修正後別叫同一個 reviewer 複查，
  它的 context 已被前一輪汙染。

---

## 9. 安裝

```bash
# 1. 複製設定到專案根目錄
cp -r sdd-kit/.claude       <your-repo>/
cp    sdd-kit/CLAUDE.md     <your-repo>/
cp -r sdd-kit/specs         <your-repo>/
cp -r sdd-kit/docs          <your-repo>/

# 2. 改 CLAUDE.md 裡的 package 路徑與技術棧描述

# 3. .gitignore 加一行
echo ".claude/agent-memory-local/" >> <your-repo>/.gitignore

# 4. 重啟 Claude Code
#    首次建立 .claude/agents/ 目錄後必須重啟才會載入
cd <your-repo> && claude

# 5. 驗證
> /agents          # 確認七個 agent 都在
```

`.claude/agents/`、`.claude/commands/`、`specs/`、`docs/` **都要進版控**，
團隊共用並一起改進。`.claude/agent-memory/`（project scope）也建議進版控。

---

## 10. 已知限制與注意事項

- **subagent 看不到你的對話歷史。** 派工時務必把檔案路徑講明，
  讓它自己去讀。整條 pipeline 靠檔案接力，不靠對話傳遞。
- **subagent 的 description 合計超過 15,000 tokens** 會在啟動時警告。
  細節寫進 system prompt，description 保持一句話。
- **subagent 結果會回流到主線 context。** 平行跑很多個各自回傳長篇結果，
  主線 context 會被吃掉。所以每個 agent 的「回報給主線」段落都要求精簡。
- **consolidation 由人主導。** 累積到 40–50 條 AC、或你開始跳著讀時，
  停下來清理：刪 `(removed)`、重新分組、切出新 domain。
  **不要交給 agent**——它會很樂意幫你重排編號，然後所有歷史引用失效。
- **每 3–4 個變更做一次慣例檢查。** 派唯讀 agent 掃「同一件事有幾種做法」，
  發現分歧就收斂並補進 CLAUDE.md。慣例漂移是這套流程主要的長期失敗模式，
  而且它累積得很安靜。
