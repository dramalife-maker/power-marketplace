# Changelog 撰寫規則

**重要：撰寫 changelog 前必須先查詢正確的當天日期，使用命令：`powershell -Command "Get-Date -Format 'yyyy-MM-dd'"`**

## 基本格式
- 格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-TW/1.0.0)
- 版本號遵循 [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
- 使用正體中文撰寫
- 每個版本都需要包含發布日期，格式為 YYYY-MM-DD，發布日期為當下日期
- 比較兩個版本之間的差異時，使用 [read-git-log-rule.mdc](read-git-log.md) 的做法

## 版本區塊結構
```markdown
## [版本號] - YYYY-MM-DD
### 分類標題
- :emoji: 變更描述
```

## 分類標題
必須使用以下分類之一：
- `Feature` - 重大新功能
- `Changed` - 既有功能的變更
- `Fixed` - 問題修復
- `Removed` - 移除的功能
- `Security` - 安全性更新

## Emoji 使用規則
使用特定的 emoji 來標示變更類型：
- `✨` - 新功能
- `🔨` - 功能調整
- `🐛` - 錯誤修復
- `🗃` - 資料庫相關變更
- `🩹` - 簡單修復
- `👨‍💻` - 開發工具或流程改善
- `🌱` - 新增遊戲或資源
- `⚡` - 效能改善
- `🔑` - 安全性相關
- `♻` - 重構
- `🗑` - 移除功能
- `👷` - CI 相關
- `🦺` - 防護措施
- `💥` - 重大變更

## 變更描述規則
1. 使用簡潔明確的描述
2. 以動詞開頭（如：新增、修改、修復、移除等）
3. 若有特定範圍，使用括號標示，例如：`(easygame)`
4. 若提及程式碼元素，使用反引號包覆，例如：`PlayerType`
5. 需理解多個commit變更的內容，並把相關的commit message 組合起來，使用簡潔明確的描述。

## 版本連結
在檔案底部維護版本比較連結：
```markdown
[Unreleased]: https://git.ttech.cc/transaction/server/-/compare/v當前版本...develop
[版本號]: https://git.ttech.cc/transaction/server/-/compare/v前一版本...v當前版本
```

## 注意事項
1. 永遠保持 [Unreleased] 區段在最上方
2. 新的版本永遠加在 [Unreleased] 區段之後
3. 每個版本的變更都需要有對應的 git tag
4. 避免在已發布的版本中進行修改
5. 確保每個變更都有適當的分類和 emoji 標記
