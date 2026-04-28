# 1on1 紀錄寫入規則

所有需要寫入 1on1 相關檔案的 skill（`cto:1on1`、`cto:1on1-add`、`cto:arch-review`）共用此規則。

---

## 目錄結構

```
<1on1_dir>/
└── <工程師名字>/
    ├── index.md             # 所有會後紀錄的一行摘要 + 月份連結，1on1 前先讀這個
    ├── YYYY-MM.md           # 當月會後紀錄（cto:1on1、cto:arch-review 寫入）
    ├── YYYY-MM-agenda.md    # 當月待討論議題清單（cto:1on1-add 寫入）
    └── _notes.md            # 長期行為模式、成長軌跡（非流水帳，定期整理）
```

`<1on1_dir>` 由 `~/.claude/plugins/cto/config.json` 的 `1on1_dir` 欄位決定。

**職責分工**：
- `YYYY-MM-agenda.md`：會前議題草稿，`cto:1on1-add` 維護
- `YYYY-MM.md`：會後討論記錄，`cto:1on1` 與 `cto:arch-review` 維護
- `index.md`：只索引會後紀錄（`YYYY-MM.md` 的內容）

---

## index.md 格式

每次寫入 `YYYY-MM.md` 後，同步在 `index.md` 新增一行：

```markdown
# <工程師名字> 1on1 Index

| 日期 | 類型 | 主題 | 檔案 |
|------|------|------|------|
| YYYY-MM-DD | <類型> | <一句話主題摘要> | [YYYY-MM](./YYYY-MM.md) |
```

**類型**對應各 skill：
- `cto:1on1` → `1on1`
- `cto:arch-review` → `架構審查`

若 `index.md` 不存在，先建立標題與表格 header，再追加資料行。

---

## _notes.md 格式

不是每次都寫，只有觀察到**跨時間的行為模式**時才更新：

```markdown
# <工程師名字> 長期觀察

## 思維模式
- <觀察描述>（初次觀察：YYYY-MM）

## 成長跡象
- <成長描述>

## 長期追蹤項目
- [ ] <待追蹤事項>
```

---

## 初次建立

若 `<1on1_dir>/<工程師名字>/` 目錄不存在，自動建立目錄與空白檔案：
- `index.md`（含標題與表格 header）
- `_notes.md`（含標題）

`YYYY-MM.md` 與 `YYYY-MM-agenda.md` 按需建立，不預先建立空白檔。

---

## 多筆紀錄分隔

同月多筆記錄以 `---` 分隔，依時間順序排列（適用於 `YYYY-MM.md` 與 `YYYY-MM-agenda.md`）：

```markdown
## YYYY-MM-DD <第一筆標題>

...內容...

---

## YYYY-MM-DD <第二筆標題>

...內容...
```
