# power-marketplace

個人 Claude Code Skills 私有倉庫，用於管理與備份自製 skills。

## 目錄

- [安裝 Marketplace](#安裝-marketplace)
- [瀏覽 Skills](#瀏覽-skills)
- [安裝個別 Plugin](#安裝個別-plugin)
- [新增 / 更新 Skill](#新增--更新-skill)
- [目錄結構說明](#目錄結構說明)
- [Skill 狀態說明](#skill-狀態說明)

---

## 安裝 Marketplace

在目標機器的 Claude Code 中執行一次：

```bash
/plugin marketplace add git@github.com:dramalife-maker/power-marketplace.git
```

> 需要 SSH key 已加入 `ssh-agent` 且 GitHub 已在 `known_hosts` 中。

---

## 瀏覽 Skills

所有可用的 plugins 列在 [index.json](./index.json)。

| Name | Category | Status | Description |
|------|----------|--------|-------------|
| `cto` | workflow | stable | 技術主管工作流程工具集（setup、arch-review、1on1-prep、1on1、perf-review） |
| `dev` | workflow | stable | 開發工作流程輔助工具集（setup、commit-message、post-commit、post-bug、archive-plan） |

---

## 安裝個別 Plugin

```bash
/plugin install <skill-name>@power-marketplace
```

---

## 新增 / 更新 Skill

完整 SOP 請參考 [_templates/PACKAGING-GUIDE.md](./_templates/PACKAGING-GUIDE.md)。

**快速 Checklist：**

1. 確認 SKILL.md 有正確的 YAML frontmatter（`name`, `description`）
2. 建立 `plugins/<skill-name>/` 目錄結構
3. 填寫 `.claude-plugin/plugin.json`
4. 更新 `.claude-plugin/marketplace.json` 的 `plugins` 陣列
5. 更新 `index.json`（新增條目、`total_plugins`、`updated_at`）
6. Push 到 GitHub

---

## 目錄結構說明

```
power-marketplace/
├── .claude-plugin/
│   └── marketplace.json       # Claude Code marketplace manifest（必要）
├── plugins/
│   └── <skill-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json    # Claude Code plugin manifest
│       └── skills/
│           └── <skill-name>/
│               └── SKILL.md   # 實際 skill 內容
├── index.json                 # 人類可讀的全局索引
├── README.md
├── CHANGELOG.md
├── _schema/                   # JSON Schema 定義（文件用）
└── _templates/                # 新增 plugin 的範本與說明
    ├── PACKAGING-GUIDE.md
    └── skill-template/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── skill-slug/
                └── SKILL.md
```

---

## Skill 狀態說明

| 狀態 | 說明 |
|------|------|
| `stable` | 穩定可用 |
| `beta` | 功能可用，可能有邊角 case |
| `wip` | 開發中，尚未可用 |
| `deprecated` | 已棄用，不建議新安裝 |
