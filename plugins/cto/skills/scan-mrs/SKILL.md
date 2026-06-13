---
name: cto:scan-mrs
description: >
  掃描指派給 zhaizhai 的所有待 review GitLab MR，判斷哪些需要處理，
  逐一以 CTO 視角產生深度 review 草稿，確認後發佈至 GitLab，
  並可將審查摘要寫入對應工程師的 1on1 紀錄。
  支援「指定 MR」捷徑、分輪審查（第一輪 / 第二輪）、
  發佈前事實查核、與同類模組對照。
---

# MR 掃描與審查 Skill

## 設定

- GitLab 帳號：`zhaizhai`
- 工具：`glab` CLI（已登入 git.ttech.cc）
- 本地 repo 快取目錄：`~/.claude/review-repos/`（clone 後永久保留，複用）

---

## 觸發模式

| 模式 | 觸發例 | 跳過步驟 |
|------|--------|----------|
| 全掃描 | 「掃 MR」、「scan mrs」 | — |
| 指定 MR | 「看一下 transaction/server!72」、「review 這個 MR」 | Step 1、3 |
| 本地分支捷徑 | 同上，且 cwd 已在該 MR 的 `source_branch` | Step 1、3、4a |

指定模式下直接跳到 Step 2 判斷輪次，再進 Step 4。

---

## Step 0：載入 CTO 設定

讀取 `~/.claude/plugins/cto/config.json`：

- **若存在**：取出 `1on1_dir` 作為 1on1 目錄路徑
- **若不存在**：自動進入 `cto:setup` 設定流程，完成後再繼續

---

## Step 1：抓取 MR 清單

```bash
glab api "merge_requests?state=opened&scope=all&reviewer_username=zhaizhai"
```

記下每個 MR 的：
- `project_id`、`iid`、`title`、`web_url`
- `references.full`（如 `transaction/server!55`）
- `author.username`
- `source_branch`、`target_branch`

---

## Step 2：判斷是否需 review + 第幾輪

逐一執行（勿並行，避免 rate limit）：

```bash
glab api "projects/{project_id}/merge_requests/{iid}/notes"
```

**判斷邏輯：**

1. 找出最後一則 body 包含 `By: AI Agent` 的 note，記為 `last_review_at`
2. 從未被 AI review → **第一輪**
3. `last_review_at` 之後有 system note（`system == true`，body 含 `"added"` 且含 `"commit"`）→ **第二輪**
4. `last_review_at` 之後有他人留言（`system == false`，`author.username != "zhaizhai"`）→ **第二輪**
5. 以上皆否 → 跳過

**第二輪額外動作：**

- 讀最後一則 `By: AI Agent` note 的完整內容（上一輪疑慮清單）
- 讀作者在其後的回覆 + 新 commit 訊息
- 在 Step 4d 報告開頭強制輸出「上一輪疑慮處理狀態」簡表

---

## Step 3：顯示掃描結果摘要

```
✅ 需要 review（N 個）
  - transaction/server!55 — [標題]（第 X 輪）

⏭️ 跳過（已 review 且無新動靜）（N 個）
  - transaction/server!47 — [標題]
```

詢問：「要開始逐一 review 嗎？還是只處理特定的？」

---

## Step 4：逐一 Review

### 4a. 準備本地 repo

從 `web_url` 推算 SSH URL：

```bash
jq -nr --arg u "$WEB_URL" '
  $u | sub("^https?://"; "") | split("/") as $s
  | $s[0] as $h
  | (($s[1:] | join("/")) | split("/-/")[0]) as $p
  | "git@\($h):\($p).git"
'
```

```bash
# 首次 clone
git clone {ssh_url} ~/.claude/review-repos/{project_path}

# 已存在則 fetch
git -C ~/.claude/review-repos/{project_path} fetch --all --prune
git -C ~/.claude/review-repos/{project_path} checkout {source_branch}
git -C ~/.claude/review-repos/{project_path} pull
```

> **本地分支捷徑**：若 cwd 已在 `source_branch`，跳過此步，改用 `git diff {target_branch}...HEAD` 取得 diff。

### 4b. 取得變更檔案清單

```bash
glab api "projects/{project_id}/merge_requests/{iid}/changes" --jq '.changes[].new_path'
```

> **Fallback**：`--jq` 在部分環境（如 Windows PowerShell）會失敗，改用 `glab api "..."` 取原始 JSON 後在主程中 parse。

### 4c. 讀取完整檔案內容 + 同類模組對照

對每個 changed file：
- source branch 版本：直接讀 `~/.claude/review-repos/{project_path}/{file_path}`
- target branch 版本：`git -C ~/.claude/review-repos/{project_path} show {target_branch}:{file_path}`

**同類模組對照（變更為新模組／整合層時必做）：**

1. 在 repo 內找 1–2 個已存在、同類別的模組（同一資料夾下的 sibling、或目錄結構相同的其他子系統）
2. 對照其分檔慣例：錯誤碼定義、router、路徑常數、設定檔、命名前綴
3. 報告中明確列出「與既有同類模組不一致」的項目

**整合文件對照：** 若 repo 內存在 `AGENTS.md`、`.claude/docs`、或與本次變更主題相關的整合 / 規格文件，先讀並對照需求。

### 4d. 產生 review 草稿

以 **CTO / 資深後端 reviewer** 視角審查，依專案實際技術棧套用對應 style guide（Go → `golang-style-guide`、Python → `python-style-guide`）。

**六大面向：**

1. **Coding style** — 命名、格式、錯誤處理、import 風格
2. **Architecture** — 分層、依賴方向、模組邊界
3. **文件** — 註解、API 契約、migration 說明
4. **可簡化的邏輯** — 重複邏輯、過深分支、過度設計
5. **慣例對齊（Convention parity）** — 是否沿用 repo 內同類模組結構，有沒有「重新發明一套」、死碼（定義了但未使用的常數／error code）
6. **具體修改建議** — 每個問題含可操作的修法（檔案路徑、符號名稱、範例）

**分輪輸出策略：**

| 輪次 | 觸發 | 輸出重點 |
|------|------|----------|
| 第一輪 | 首次 review；integration 類；語意/文件不明 | ❓ 待確認為主；✅ 已對照部分；**少給**修法 |
| 第二輪 | 上次 AI review 後有新 commit 或作者回覆 | 先輸出「上一輪疑慮：已解 / 未解」，再給 ❌ 需修正 |

**輸出格式：**

```markdown
## 審查摘要：{MR 標題}（第 X 輪）

### 📋 上一輪疑慮處理狀態（僅第二輪）
| 上一輪項目 | 狀態 | 備註 |
|------------|------|------|

### ✅ 做得好的地方
- ...

### ❌ 需要修正
| 嚴重度 | 問題 | 修法 |
|--------|------|------|
| [高]   | ...  | ...  |
| [中]   | ...  | ...  |

### ❓ 建議追問
- ...

### 💡 整體評估
綜合判斷，含這次審查觀察到的工程師思維模式。

By: AI Agent
```

**格式規則：**

- **❌** = 確定要改；**❓** = 需作者/客戶確認或設計取捨 — 不確定就放 ❓，不要硬塞進 ❌
- 避免多層巢狀 bullet
- `[高]` 每份 review 建議 ≤ 3 項
- 使用者明確要求時才附上評分表（功能完成度／文件對齊／慣例對齊／可維護性 各 /10）

### 4e. 發佈前事實查核

**規則：任何 `[高]` 項，發佈前必須用程式路徑驗證；不確定 → 改放 ❓，不寫進 ❌。**

Checklist：

- 錯誤實際回傳型別？`errors.Is` / 對應語言的等價檢查是否成立？
- 描述的程式路徑是否真的會被執行到？
- 是「設計取捨」還是「確定 bug」？分不清就降為 ❓
- 「像 AI 生成後未自審」這類觀察僅能依特徵**推測**，不作事實指控

### 4f. 呈現草稿給使用者確認

顯示草稿後問：「這份 review 可以發佈嗎？還是需要調整？」支援多輪修訂。

### 4g. 發佈 review

使用者確認後：

```bash
glab mr note {iid} -R {project_path} --message "{review_content}"
```

### 4h. 詢問是否寫入 1on1 紀錄

發佈後主動詢問：

> 「要將這次對 {author.username} 的審查摘要寫入 1on1 紀錄嗎？」

依 `~/.claude/plugins/cto/references/1on1-writing-rules.md` 規則寫入。

**寫入位置判斷：**

| 內容 | 寫入位置 |
|------|----------|
| 單次 MR 具體問題、本次審查意見 | `{1on1_dir}/{author.username}/YYYY-MM.md` |
| 第二輪 review | 同年月檔以 `---` 追加；可更新先前項目的勾選狀態 `[ ]` → `[x]` |
| 跨 MR 重複出現的行為模式 / 成長跡象 | `_notes.md`（依該檔格式） |

`YYYY-MM.md` 條目格式：

```markdown
## YYYY-MM-DD 架構審查：{MR 標題}（第 X 輪）

**背景**：{一句話說明此 MR 的目的}

**工程師提案重點**：
- ...

**審查意見**：
- [高] ...
- [中] ...

**審查歷程**（可選）：第幾輪、作者如何回應、哪些疑慮已解

**觀察到的思維模式**：
{工程師展現的決策習慣、盲點或成長跡象}

**後續追蹤**：
- [ ] {退回修改的項目}
```

同步更新 `index.md`（類型填 `架構審查`）。

判斷是否同步更新 `_notes.md`：若這次觀察與過去 review 構成重複模式，才追加到 `_notes.md` 的對應段落，不是每次都寫。

### 4i. 無問題時詢問是否 merge

若草稿中無任何 ❌ 項目，詢問：「此 MR 無問題，要直接 merge 嗎？」

使用者確認後：

```bash
glab mr merge {iid} -R {project_path} --remove-source-branch
```

---

## 注意事項

- **絕對不要在未經使用者確認前自動發佈**
- Pipeline 失敗時口頭告知，不寫進 review comment
- Diff 為空或無實質變更，告知後跳過
- 遇 `glab` 指令錯誤，顯示原始錯誤並停止
- `--jq` 在 Windows PowerShell 易失敗，改取原始 JSON 後 parse

---

## 參考文件

- `~/.claude/plugins/cto/references/1on1-writing-rules.md`：1on1 寫入規則（共用）
- `~/.claude/plugins/cto/skills/arch-review/references/general-criteria.md`：通用審查標準
