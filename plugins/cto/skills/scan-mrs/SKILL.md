---
name: cto:scan-mrs
description: >
  掃描指派給 zhaizhai 的所有待 review GitLab MR，判斷哪些需要處理，
  逐一以 CTO 視角產生深度 review 草稿，確認後發佈至 GitLab，
  並可將審查摘要寫入對應工程師的 1on1 紀錄。
---

# MR 掃描與審查 Skill

## 設定

- GitLab 帳號：`zhaizhai`
- 工具：`glab` CLI（已登入 git.ttech.cc）
- 本地 repo 快取目錄：`~/.claude/review-repos/`（clone 後永久保留，複用）

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

## Step 2：判斷哪些 MR 需要 review

逐一執行（勿並行，避免 rate limit）：

```bash
glab api "projects/{project_id}/merge_requests/{iid}/notes"
```

**判斷邏輯：**

1. 找出最後一則 body 包含 `By: AI Agent` 的 note，記為 `last_review_at`
2. 從未被 AI review → 需要 review
3. `last_review_at` 之後有 system note（`system == true`，body 含 `"added"` 且含 `"commit"`）→ 需要 re-review
4. `last_review_at` 之後有他人留言（`system == false`，`author.username != "zhaizhai"`）→ 需要 re-review
5. 以上皆否 → 跳過

---

## Step 3：顯示掃描結果摘要

```
✅ 需要 review（N 個）
  - transaction/server!55 — [標題]

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

### 4b. 取得變更檔案清單

```bash
glab api "projects/{project_id}/merge_requests/{iid}/changes" --jq '.changes[].new_path'
```

### 4c. 讀取完整檔案內容

對每個 changed file：
- source branch 版本：直接讀 `~/.claude/review-repos/{project_path}/{file_path}`
- target branch 版本：`git -C ~/.claude/review-repos/{project_path} show {target_branch}:{file_path}`

### 4d. 產生 review 草稿

以 **CTO / 資深後端 reviewer** 視角審查，依專案實際技術棧套用對應 style guide（Go → `golang-style-guide`、Python → `python-style-guide`）。若專案有 `AGENTS.md` 或 `.claude/docs`，開 review 前先讀並套用。

**必須涵蓋五大面向：**

1. **Coding style** — 命名、格式、錯誤處理、import 風格是否與專案一致
2. **Architecture** — 分層、依賴方向、模組邊界是否違反既有架構
3. **文件** — 註解、API 契約、migration 說明是否足以理解「為何這樣改」
4. **可簡化的邏輯** — 重複邏輯、過深分支、過度設計
5. **具體修改建議** — 每個問題須含可操作的修法（檔案路徑、符號名稱、範例）

**輸出格式：**

```
## 審查摘要：{MR 標題}

### ✅ 做得好的地方
- ...

### ⚠️ 需要關注的問題
- [嚴重度：高/中/低] 問題描述 → 建議做法

### ❓ 建議追問的問題
- ...

### 💡 整體評估
綜合判斷，包含這次審查觀察到的工程師思維模式。

By: AI Agent
```

### 4e. 呈現草稿給使用者確認

顯示草稿後問：「這份 review 可以發佈嗎？還是需要調整？」

### 4f. 發佈 review

使用者確認後：

```bash
glab mr note {iid} -R {project_path} --message "{review_content}"
```

### 4g. 詢問是否寫入 1on1 紀錄

發佈後主動詢問：

> 「要將這次對 {author.username} 的審查摘要寫入 1on1 紀錄嗎？」

使用者確認後，依 `~/.claude/plugins/cto/references/1on1-writing-rules.md` 的規則，在 `{1on1_dir}/{author.username}/YYYY-MM.md` 新增：

```markdown
## YYYY-MM-DD 架構審查：{MR 標題}

**背景**：{一句話說明此 MR 的目的}

**工程師提案重點**：
- ...

**審查意見**：
- [高] ...
- [中] ...

**觀察到的思維模式**：
{工程師展現的決策習慣、盲點或成長跡象}

**後續追蹤**：
- [ ] {退回修改的項目}
```

同步更新 `index.md`（類型填 `架構審查`）。

### 4h. 無問題時詢問是否 merge

若草稿中無任何 ⚠️ 問題，詢問：「此 MR 無問題，要直接 merge 嗎？」

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

---

## 參考文件

- `~/.claude/plugins/cto/references/1on1-writing-rules.md`：1on1 寫入規則（共用）
- `~/.claude/plugins/cto/skills/arch-review/references/general-criteria.md`：通用審查標準
