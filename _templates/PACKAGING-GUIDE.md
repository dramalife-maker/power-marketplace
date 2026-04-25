# Packaging Guide — 把現有 skill 包裝成 Plugin

## 新 Plugin 的目錄結構

每個 plugin 的完整結構：

```
plugins/
└── <skill-name>/
    ├── .claude-plugin/
    │   └── plugin.json          ← Claude Code 識別用 manifest
    └── skills/
        └── <skill-name>/
            └── SKILL.md         ← 實際 skill 內容
```

---

## Step 1：確認 SKILL.md frontmatter

SKILL.md 需包含 YAML frontmatter：

```yaml
---
name: plugin-name:skill-slug
description: 一行說明這個 skill 做什麼
---

（skill 主體內容）
```

- `name` 格式為 `plugin-name:skill-slug`（避免與其他 marketplace 的 skill 撞名）
- `plugin-name` 對應 plugin 目錄名稱，`skill-slug` 對應 skill 目錄名稱
- 呼叫指令為 `/plugin-name:skill-slug`

---

## Step 2：建立 Plugin 目錄

```bash
mkdir -p plugins/<skill-name>/.claude-plugin
mkdir -p plugins/<skill-name>/skills/<skill-name>
```

複製 skill 檔：

```bash
cp /path/to/SKILL.md plugins/<skill-name>/skills/<skill-name>/SKILL.md
```

---

## Step 3：建立 plugin.json

複製範本：

```bash
cp _templates/skill-template/.claude-plugin/plugin.json plugins/<skill-name>/.claude-plugin/plugin.json
```

填入內容：

```json
{
  "name": "<skill-name>",
  "description": "<一行說明，引用 SKILL.md frontmatter 的 description>",
  "version": "1.0.0"
}
```

---

## Step 4：更新 .claude-plugin/marketplace.json

在根目錄的 `.claude-plugin/marketplace.json` 的 `plugins` 陣列新增條目：

```json
{
  "name": "<skill-name>",
  "source": "./plugins/<skill-name>",
  "description": "<一行說明>"
}
```

---

## Step 5：更新 index.json（人類可讀索引）

在 `index.json` 的 `plugins` 陣列新增：

```json
{
  "name": "<skill-name>",
  "display_name": "<Human Readable Name>",
  "version": "1.0.0",
  "category": "<git|workflow|code-review|browser|misc>",
  "description": "<一行說明>",
  "tags": ["tag1", "tag2"],
  "status": "stable",
  "path": "plugins/<skill-name>/",
  "updated_at": "<今天日期>"
}
```

同步更新 `meta.total_plugins` (+1) 與 `meta.updated_at`。

---

## Step 6：更新 README.md 表格

在「瀏覽 Skills」表格新增一行：

```markdown
| skill-name | category | stable | 一行說明 |
```

---

## 驗證 Checklist

- [ ] `plugins/<name>/.claude-plugin/plugin.json` 的 `name` = 目錄名稱
- [ ] `plugins/<name>/skills/<name>/SKILL.md` 存在
- [ ] `.claude-plugin/marketplace.json` 的 `plugins` 陣列已更新
- [ ] `index.json` 的條目與 `total_plugins` 正確
- [ ] Push 到 GitHub 後，在目標機器執行 `/plugin install <name>@power-marketplace` 驗證

---

## 安裝（其他機器）

首次設定：

```bash
/plugin marketplace add dramalife-maker/power-marketplace
# 或使用完整 SSH URL：
/plugin marketplace add git@github.com:dramalife-maker/power-marketplace.git
```

安裝個別 plugin：

```bash
/plugin install <skill-name>@power-marketplace
```

---

## 更新現有 Plugin

1. 修改 `plugins/<name>/skills/<name>/SKILL.md`
2. 更新 `plugins/<name>/.claude-plugin/plugin.json` 的 `version`（semver）
3. 更新 `index.json` 的對應條目 `version` 與 `updated_at`
4. 更新 `meta.updated_at`
5. 在 `CHANGELOG.md` 記錄變更
