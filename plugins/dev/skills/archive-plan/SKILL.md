---
name: dev:archive-plan
description: >
  將當前專案的 Claude Code plan mode 計劃歸檔到個人計劃日誌 repo。
  當使用者說「歸檔計劃」、「把這個計劃存起來」、「archive plan」時使用此 skill。
  需先執行 dev:setup 設定計劃日誌路徑。
---

# 計劃歸檔 Skill

## Step 0：載入設定

讀取 `~/.claude/plugins/dev/config.json`：

- **若檔案存在**：取出 `plan_journal_dir` 作為計劃日誌目錄路徑
- **若檔案不存在**：自動進入 `dev:setup` 設定流程，完成後再繼續歸檔

後續所有提到 `~/plan-journal` 的地方，皆以此設定值替換。

---

## Step 1：選擇計劃來源

檢查對話 context 中是否有 plan mode 系統訊息（訊息格式包含 plan 檔案路徑，例如 `~/.claude/plans/<name>.md`）：

- **若有偵測到 plan 檔案路徑**：確認檔案確實存在於 `~/.claude/plans/`，然後使用 **AskUserQuestion tool** 詢問：
  > 「偵測到 plan 檔案 `<filename>.md`，請選擇計劃來源：」
  - 選項 1（預設）：使用 plan 檔案（`~/.claude/plans/<filename>.md`）
  - 選項 2：根據對話上下文整理計劃

- **若未偵測到 plan 檔案**：使用 **AskUserQuestion tool** 詢問：
  > 「請選擇計劃來源：」
  - 選項 1（預設）：根據對話上下文整理計劃
  - 選項 2：指定 plan 檔案路徑

若 **AskUserQuestion tool** 不可用，改以純文字詢問並等待使用者回覆。

### 依選擇執行

**Plan 檔案模式**：
- 讀取 plan 檔案，保留背景、目標、決策過程與最終方案；略過任務清單、待執行步驟等實作細節

**上下文整理模式**：
- 從對話歷史中萃取計劃內容，整理成結構化文件，包含：
  - 計劃背景與目標
  - 決策過程與取捨
  - 最終方案摘要

2. **計劃名稱**：從計劃標題或檔名推導，若不明確則詢問使用者
3. **專案 slug 解析**：
   - 取得當前工作目錄的絕對路徑（例如 `E:/workspace/github/my-app`）
   - 查找 `~/.claude/plugins/dev/config.json` 的 `project_slugs` 欄位，看是否有此路徑的對應記錄
   - **若已有記錄**：直接使用已設定的 slug，顯示給使用者確認
   - **若無記錄**：以目錄最後一層名稱作為建議值，詢問使用者：
     > 「這個專案尚未設定 slug。建議使用 `<dir-name>`，或請輸入自訂名稱：」
   - 使用者確認後，將對應關係寫回 `config.json` 的 `project_slugs`：
     ```json
     {
       "plan_journal_dir": "~/plan-journal",
       "project_slugs": {
         "E:/workspace/github/my-app": "my-app"
       }
     }
     ```

---

## Step 2：寫入計劃日誌

### 目錄結構

```
~/plan-journal/
└── <project-slug>/
    ├── index.md                          # 所有計劃的一行摘要，瀏覽用
    └── <YYYY-MM-DD>-<plan-name>.md       # 歸檔的計劃全文
```

### 計劃檔案格式

寫入 `<YYYY-MM-DD>-<plan-name>.md`，在原始計劃內容前加上 header：

```markdown
---
project: <project-slug>
plan_name: <plan-name>
archived_at: <YYYY-MM-DD>
source: <plan 檔案模式填 ".claude/plans/<filename>.md"；上下文整理模式填 "conversation-context">
---

<計劃內容>
```

### index.md 格式

每次歸檔後，在 `~/plan-journal/<project-slug>/index.md` 新增一行：

```markdown
# <project-slug> 計劃日誌

| 日期 | 計劃名稱 | 檔案 |
|------|----------|------|
| 2026-04-25 | 初始架構計劃 | [2026-04-25-init-arch](./2026-04-25-init-arch.md) |
```

### 初次建立

若 `~/plan-journal/<project-slug>/` 不存在，自動建立目錄與 `index.md`。

---

## Step 3：確認完成

歸檔完成後回報：

> 「已歸檔至 ~/plan-journal/<project-slug>/<YYYY-MM-DD>-<plan-name>.md，並更新 index.md。」
