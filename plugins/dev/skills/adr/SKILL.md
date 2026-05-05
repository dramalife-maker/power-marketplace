---
name: dev:adr
description: 在專案中建立或維護 Architecture Decision Record (ADR)。新增決策、或當決策改變時建立新 ADR 並標記舊的為 Superseded。當使用者說「記錄一個架構決策」、「建立 ADR」、「add adr」、「這個決策改變了」時使用。
---

## Step 0 — 判斷操作模式

使用 `AskUserQuestion` tool 詢問使用者要做什麼：

- **新增 ADR**：記錄一個新的架構決策
- **Supersede 舊 ADR**：某個決策已改變，需標記舊 ADR 並建立新的

根據回答進入對應流程。

---

## 模式 A：新增 ADR

### Step A1 — 確認下一個編號

列出 `.claude/docs/adr/` 目錄下所有 `.md` 檔案（排除 `index.md`），計算數量決定下一個編號。

- 若目錄不存在，下一個編號為 `001`
- 若已有 N 個 ADR，下一個編號為 N+1，補零至 3 位（例如 `002`, `010`）

### Step A2 — 收集內容

使用 `AskUserQuestion` tool 詢問：

- **標題**：這個決策的簡短名稱（例如「採用 Redis 作為快取層」）
- **Context**：什麼問題或背景促成這個決策？
- **Decision**：最終決定是什麼？為什麼？
- **Consequences**：正面影響與負面影響各是什麼？

Status 固定為 `Accepted`，日期為今天。

### Step A3 — 寫入文件與更新索引

依照 [references/adr-format.md](references/adr-format.md) 建立文件：

**檔案路徑：** `<project-root>/.claude/docs/adr/ADR-<NNN>-<title-slug>.md`

> `<project-root>` 為當前專案的根目錄（即 `CLAUDE.md` 所在位置，或 git repo 根目錄），ADR 存放於專案 scope，不是使用者全域目錄。

- `title-slug`：標題轉為小寫英文 kebab-case；中文標題用語意英文轉換

建立目錄（若不存在），寫入文件，然後更新 `<project-root>/.claude/docs/adr/index.md`：

```markdown
| ADR-<NNN> | <標題> | Accepted | <YYYY-MM-DD> |
```

若 `index.md` 不存在，以此格式建立：

```markdown
# ADR Index

| # | 標題 | 狀態 | 日期 |
|---|------|------|------|
| ADR-<NNN> | <標題> | Accepted | <YYYY-MM-DD> |
```

### Step A4 — 確認完成

顯示建立的檔案路徑與 ADR 編號。

---

## 模式 B：Supersede 舊 ADR

### Step B1 — 顯示現有 ADR

讀取 `<project-root>/.claude/docs/adr/index.md` 並顯示給使用者，讓他們確認要 supersede 哪一筆（例如「ADR-001」）。

### Step B2 — 收集新決策內容

使用 `AskUserQuestion` tool 詢問：

- **標題**：新決策的名稱
- **Context**：決策為何改變？新的背景是什麼？
- **Decision**：新的決定是什麼？為什麼比舊方案更好？
- **Consequences**：正面影響與負面影響

新 ADR 會加入 `Supersedes: ADR-<舊編號>` 欄位。

### Step B3 — 更新舊 ADR

找到對應的舊 ADR 檔案，將其 Status 行改為：

```
**Status:** Superseded by ADR-<新編號>
```

### Step B4 — 建立新 ADR 與更新索引

依照 [references/adr-format.md](references/adr-format.md) 建立新 ADR 文件（帶 `Supersedes` 欄位），然後更新 `<project-root>/.claude/docs/adr/index.md`：

- 新增新 ADR 的一列
- 將舊 ADR 的狀態欄位更新為 `Superseded by ADR-<新編號>`

### Step B5 — 確認完成

顯示：
- 舊 ADR 路徑（已標記 Superseded）
- 新 ADR 路徑與編號
