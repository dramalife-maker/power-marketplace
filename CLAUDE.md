# CLAUDE.md

## Plugin 版本管理

修改任何 plugin 內的 skill 或設定後，**必須同步更新該 plugin 的版本號**：

- 版本檔位置：`plugins/<plugin-name>/.claude-plugin/plugin.json`
- 版本規則：patch +1（例如 `1.3.1` → `1.3.2`）
- 連同 skill 變更一起 commit，或單獨 commit 均可，但不得遺漏
