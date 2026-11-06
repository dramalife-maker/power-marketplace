---
name: dev:pre-review
description: >-
  Commit/merge 前的兩階段 code review — Stage 1 資深工程師（正確性、架構、死碼、
  contract/相容性、build），Stage 2 資深 QA 接力（測試覆蓋缺口、部署/runtime/回滾風險、
  驗證計畫）。當變更涉及 SQL／資料庫（migration、schema、ORM/GORM 查詢）時，Stage 1
  平行加派資深 DBA 審查 query 效率、索引、schema 與 migration 安全性。Report-only，
  絕不改檔。當使用者說「pre-review」「兩階段 review」「資深工程師與資深 QA review」
  「eng + QA review」「commit 前幫我 review」，或想要一份兼顧工程正確性與 QA／上線
  風險的嚴格 pre-commit／pre-merge 審查時觸發 — 即使沒講「兩階段」也適用。預設審查未
  commit 的 diff；可帶參數指向 branch、commit range 或 PR。
---

# 兩階段 Code Review（資深工程師 → 資深 QA）

把一份程式碼變更交給獨立的 reviewer 依序審查：Stage 1 資深工程師看「對不對、乾不乾淨」（若變更涉及 SQL／資料庫，再加一位資深 DBA 平行審查 query／索引／schema／migration），Stage 2 資深 QA 接力看「上線會不會出事、怎麼驗證」。全程**只報告、不改動任何檔案**，最後彙整成一份結論，由使用者決定採納哪些。

為什麼分階段而非一次審完：工程正確性、資料庫健全性、上線品質是三種不同的思考模式，混在一個 prompt 裡會互相稀釋。讓 QA 拿到前面的發現再接力，能順著已指出的風險往「實際後果與驗證」深挖，而不是各說各話。DBA 之所以平行於工程師（而非另開一階段），是因為它與工程正確性是同一層「變更本身對不對」的獨立視角，無先後依賴。

## 何時用

使用者要求對「尚未 commit / 尚未 merge」的變更做把關時。常見觸發語：「pre-review」「資深工程師與資深 QA 進行 2 階段 review」「兩階段 review」「eng + QA review」「commit 前幫我 review」。即使沒講「兩階段」，只要意圖是「commit/merge 前要一份兼顧工程與 QA 的嚴格審查」，就適用。

## 流程總覽

1. 決定 review 對象（diff / branch / commit 範圍 / PR）
2. 偵測專案技術棧與是否涉及 SQL／資料庫，挑選 reviewer agent 與檢查重點
3. **Stage 1 — 工程審查**：派資深工程師 agent；若變更涉及 SQL／資料庫，同時平行派資深 DBA agent
4. **Stage 2 — 資深 QA**：把 Stage 1 的全部發現（工程師 ＋ DBA）摘要餵給 QA agent，做品質／上線審查
5. 彙整所有結論呈現給使用者

順序原則：Stage 1 的工程師與 DBA 彼此獨立、可在同一輪**平行**派發；但 Stage 2（QA）必須在 Stage 1 全部完成後才開始，因為它要接力前面的發現。

---

## Step 1 — 決定 review 對象

預設審查**未 commit 的變更**：`git diff HEAD`（含 staged 與 unstaged）。未追蹤的新檔不在其中，另用 `git ls-files --others --exclude-standard` 列出並納入審查範圍。

若使用者帶了參數，依下列判斷對象，並先確認範圍再往下：

- **branch 名 / commit 範圍**（如 `feat/x`、`main..HEAD`、`abc123..def456`）→ `git diff <range>`
- **PR / MR 號**（如 `#42`、`!42`）→ 用 `gh pr diff <n>` 或 `glab mr diff <n>` 取得 diff
- **明確檔案清單** → 只審那些檔案的 diff

先跑一次 `git diff --stat <對象>` 確認檔案範圍與規模，並在派工前**排除明顯無關的檔案**（例如他人或先前殘留的未追蹤檔、自動產生物），告知使用者你審查的範圍。

把對象的取得方式（具體 git/gh/glab 指令）一併寫進給 agent 的 prompt，讓 agent 自己重跑取得完整 diff 與上下文，而不是只貼一段你截好的片段。

## Step 2 — 偵測技術棧、挑選 reviewer

先快速判斷專案語言／框架（看 `go.mod`、`package.json`、`pyproject.toml`、`Cargo.toml`、`pom.xml` 等），用來決定：

- **subagent_type**：
  - Go 專案 → `go-dev`
  - 其他語言若有對應的專門 agent（檢視當前可用 agent 清單）→ 用它
  - 沒有對應專門 agent → `general-purpose`
- **build/驗證指令**：依語言給 agent 正確的編譯與靜態檢查指令（Go: `go build ./...` / `go vet ./...`；Node: `tsc --noEmit` / `npm run lint`；Python: `ruff` / `mypy`；Rust: `cargo build` / `cargo clippy`…）。挑專案實際使用的那套（看 Makefile、scripts、CI 設定）。
- **語言特有檢查項**：例如 Go 看未用 import 與 error wrapping、TS 看型別與 null 安全、Python 看型別註記與例外處理。
- **是否加派資深 DBA**：掃描 diff 是否涉及 SQL 或資料庫層，命中任一即在 Stage 1 平行加派 `dev:dba` agent：
  - `.sql` 檔、migration 檔（schema DDL、index、constraint、欄位型別／nullability 變更）
  - ORM／query builder 程式碼（例如 GORM：`.Where/.Joins/.Preload/.Select/.Raw/.Group/.Having` 等）、或 `database` / `repository` / `models` 目錄下的查詢或 schema 變更
  - 原生 SQL 字串、stored procedure、view、trigger
  純應用邏輯、與資料存取無關的變更則**不需** DBA —— 不要為了用而用。

不要假設語言或技術棧；先確認再下指令。

---

## Stage 1 — 工程審查（資深工程師 ＋ 視情況資深 DBA）

派資深工程師 agent（依 Step 2 選定的 subagent_type）。**若 Step 2 判定涉及 SQL／資料庫**，在同一輪訊息一併派 `dev:dba` agent（兩者獨立、平行，縮短總時間）。兩個 prompt 都必須包含：變更背景脈絡、review 對象的取得指令、檢查面向、輸出格式、以及「只審查不改動」的硬性要求；工程師額外要求「跑編譯／靜態檢查驗證」。

### 資深工程師 prompt 範本

```
你是一位資深〈語言〉工程師，對一組「尚未 commit/merge」的變更做嚴格 code review。
專案：〈路徑〉。

## 變更背景
〈一句話交代這次變更要解決什麼、有哪些已拍板的設計決策。若有相關 issue/PR 一併附上。〉

## review 對象
執行〈git diff HEAD ／ git diff main..HEAD ／ gh pr diff N 等〉取得完整 diff，
並排除〈無關檔案〉。涉及檔案大致為：〈清單〉。

## 逐項給結論
1. 正確性：邏輯、邊界條件、競態、錯誤處理是否正確。
2. 架構與一致性：是否符合周邊既有風格與抽象；有無更簡單的作法。
3. 死碼 / 殘留：移除某機制後，是否還有殘留引用、變成無人使用的 exported symbol、
   失真的註解或文件。
4. 未用 import / 變數：受影響檔案的 import 增減是否乾淨。
5. API / contract 影響：對外回應結構、函式簽名、DB schema 的變更與其 blast radius。
6. 其他資深工程師該抓的：命名、log、可觀測性、效能熱點。

## 驗證
實際執行〈build 指令〉與〈靜態檢查指令〉確認可編譯、無 lint 問題，並回報結果。

## 限制
只做 review，**不要修改任何檔案**。

## 輸出格式
以 **Blocker / Should-fix / Nit / 讚許** 分類，每項標明〈檔案:行號〉與具體理由。
最後給一個分類數量的小結表。
```

### 資深 DBA prompt 範本（僅在涉及 SQL／資料庫時派發）

```
你是一位資深 DBA，對一組「尚未 commit/merge」的變更做資料庫面審查。
專案：〈路徑〉，資料庫：〈PostgreSQL / MariaDB / ClickHouse… 看專案實際使用〉，
ORM：〈如 GORM；若有〉。

## 變更背景
〈同工程師的背景〉

## review 對象
〈同工程師的取得指令〉，聚焦其中涉及 SQL／schema／查詢的部分：〈檔案清單〉。

## 逐項給結論
1. 查詢正確性與效率：是否有 N+1、缺索引導致的 full scan、SELECT *、
   未用上既有索引、不必要的 JOIN／排序。
2. 索引設計：新查詢是否需要新索引；既有索引是否足夠或冗餘；複合索引欄位順序。
3. Schema 設計：欄位型別、nullability、預設值、約束、外鍵是否合理。
4. Migration 安全性：DDL 是否會長時間鎖表、是否需要 online DDL／分批 backfill、
   是否可回滾、對既有資料的相容性。
5. ORM 生成的 SQL：確認 ORM 查詢實際生成的 SQL 與意圖一致（必要時推導或實跑 EXPLAIN）。
6. 目標資料庫的方言正確性：語法／函式是否符合該 DB 引擎。

## 限制
只做 review，**不要修改任何檔案、不要對正式資料庫執行寫入或 DDL**。
（唯讀的 EXPLAIN／查 schema 可行，但需確認連線安全。）

## 輸出格式
以 **Blocker / Should-fix / Nit / 讚許** 分類，每項標明〈檔案:行號 或 SQL 片段〉與具體理由。
```

收到結果後，**保留 Stage 1 的完整發現**（工程師 ＋ DBA，尤其 Blocker 與 Should-fix 清單），它是 Stage 2 的輸入。

## Stage 2 — 資深 QA 審查

在 Stage 1 全部完成後（工程師 ＋ 若有 DBA），派 QA agent。Prompt 必須把 **Stage 1 的發現摘要（含 DBA）**帶進去，請 QA 在其基礎上補品質視角，不要重複前面已說的。若涉及資料庫，QA 應把 DBA 點出的 migration／schema 風險轉化為部署順序與回滾考量。

給 agent 的 prompt 範本：

```
你是一位資深 QA 工程師，對一組「尚未 commit/merge」的變更做第二階段品質審查
（第一階段資深工程師 review 已完成，發現摘要見下）。專案：〈路徑〉。

## 變更背景
〈同 Stage 1 的背景〉

## review 對象
〈同 Stage 1 的取得指令與檔案範圍〉

## 第一階段發現（工程師 ＋ DBA；請在此基礎上補 QA 視角，不要重複）
〈貼上 Stage 1 工程師與 DBA 的 Blocker / Should-fix / Nit 摘要〉

## 逐項給結論（聚焦「會不會出事」與「怎麼驗證」，而非 code style）
1. 測試覆蓋缺口：哪些行為值得補測試、該測哪些 case。但要**務實**判斷：
   以這個專案現況的測試慣例來看，補測試是合理還是 over-engineering？
2. 部署 / runtime 風險：新舊版本並存（rolling deploy）、殘留狀態、快取 /
   schema 相容、外部 contract 變更對下游的影響。
3. 回滾風險：回到舊版會發生什麼？有無資料損毀或需預備的降級。
4. 向後相容：序列化 / 反序列化、API 欄位增減、DB migration 的雙向相容。
5. 驗證計畫：給一組可實際執行的步驟（指令 / API 呼叫 / log 觀察 / 監控指標）
   來確認上線後行為正確。

## 限制
只做 review，**不要修改任何檔案**。

## 輸出格式
以 **上線阻擋項 / 上線前建議 / 驗證計畫 / 可接受風險** 分類，分輕重、務實。
```

---

## Step 5 — 彙整呈現

把所有階段結論整理成一份精簡報告給使用者，包含：

- **總覽**：Stage 1 工程師（Blocker/Should-fix/Nit/讚許 數量）、Stage 1 DBA（若有，同樣分類數量）、Stage 2 QA（有無上線阻擋項）。
- **必須處理**：合併工程師 ＋ DBA 的 Blocker/Should-fix 與 QA 的上線阻擋項，去重。
- **建議但可選**：各方的 Nit、上線前建議。
- **上線 / 回滾須知**：QA 點出、需使用者或 SRE 知悉的風險（含 DBA 的 migration／鎖表／回滾考量）。
- **驗證計畫**：QA 給的可執行步驟。

明確收尾：**這是 report-only**，沒有改動任何檔案。詢問使用者要採納哪些、是否要你接著修正。不要自作主張改檔案。

## 務實原則

- **不要為了湊數而報問題**。沒有 Blocker 就說沒有；讚許該給就給。雜訊會讓報告失去價值。
- **尊重專案慣例**。例如某 repo 整層沒有測試，那「補測試」多半是 over-engineering 而非缺陷 —— QA 要點出落差但給出符合現實的判斷。
- **檔案:行號要具體**，讓使用者能直接跳轉。
- **遇到跨 repo / 看不到的依賴**（例如下游服務怎麼消費某 API 欄位），明說這是假設、需使用者確認，不要臆測成事實。
- **DBA 是條件式的**：只在真的碰到 SQL／schema／查詢時才加派；純邏輯變更硬塞 DBA 只會產生雜訊。且 DBA 嚴禁對正式資料庫做寫入或 DDL，最多唯讀 EXPLAIN／查 schema。
