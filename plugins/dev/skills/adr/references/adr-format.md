# ADR 文件格式（Nygard 簡約格式）

## 模板（新增）

```markdown
# ADR-<NNN>: <標題>

**Status:** Accepted

**Date:** YYYY-MM-DD

## Context

<描述驅動這個決策的問題背景、現況與限制條件。說明為什麼需要做出這個決策。>

## Decision

<描述最終決定的內容，以及選擇這個方向的理由。>

## Consequences

### 正面
- <好處 1>
- <好處 2>

### 負面
- <代價或風險 1>
- <代價或風險 2>
```

## 模板（Supersede 舊決策時）

新增 `Supersedes` 欄位：

```markdown
# ADR-<NNN>: <標題>

**Status:** Accepted

**Date:** YYYY-MM-DD

**Supersedes:** ADR-<舊編號>

## Context

<說明舊決策為何不再適用，以及新的問題背景。>

## Decision

<新的決定內容與理由。>

## Consequences

### 正面
- <好處 1>

### 負面
- <代價或風險 1>
```

被 supersede 的舊 ADR，Status 行改為：

```markdown
**Status:** Superseded by ADR-<新編號>
```

## 欄位說明

| 欄位 | 說明 |
|------|------|
| Status | `Accepted` / `Superseded by ADR-XXX` |
| Date | 格式 `YYYY-MM-DD`，填寫建立日期 |
| Supersedes | 僅在 supersede 情境填寫，指向被取代的 ADR 編號 |
| Context | 聚焦在「問題是什麼」，不要帶入解法 |
| Decision | 說明「做什麼」以及「為什麼」，避免只寫結論 |
| Consequences | 正反面都要誠實列出，不要只寫好處 |

## 命名規則

- 檔名：`ADR-<NNN>-<title-slug>.md`
- 編號補零至 3 位：`001`, `002`, ..., `010`, `011`
- Slug 使用小寫英文 kebab-case，反映標題語意
