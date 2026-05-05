---
name: dev:setup
description: 設定 Dev plugin 的工作目錄偏好（例如計劃日誌路徑）。安裝 dev plugin 後執行一次，或需要變更路徑時重新執行。
---

# Dev Plugin 設定

## 步驟 1：讀取現有設定

嘗試讀取 `~/.claude/plugins/dev/config.json`：
- 若存在，顯示目前的設定值，詢問使用者是否要修改
- 若不存在，直接進入設定流程

## 步驟 2：詢問設定項目

逐一詢問以下項目（括號內為預設值，使用者可直接按 Enter 套用）：

1. **計劃日誌目錄**（預設：`~/plan-journal`）
   > 「請輸入計劃日誌的存放目錄路徑（預設：~/plan-journal）：」

## 步驟 3：寫入設定檔

將使用者輸入（或預設值）寫入 `~/.claude/plugins/dev/config.json`：

```json
{
  "plan_journal_dir": "<使用者輸入的路徑>"
}
```

若 `~/.claude/plugins/dev/` 目錄不存在，先建立它。

## 步驟 4：寫入專案權限設定

在**當前工作目錄**的 `.claude/settings.json` 加入 `dev:post-bug` 背景 sub-agent 所需的寫入權限。

規則：
- 若 `.claude/settings.json` 不存在，建立目錄並建立檔案
- 若已存在，讀取後**合併** `permissions.allow` 陣列，不覆寫原有內容
- 只在尚未包含時才新增，避免重複

需確保以下兩條規則存在於 `permissions.allow`：

```
Write(.claude/docs/learning-notes/**)
Edit(.claude/docs/learning-notes/**)
```

完成後告知使用者已更新哪個檔案。

## 步驟 5：確認完成

顯示儲存後的設定內容，並說明：

> 「設定已儲存。之後使用 /dev:archive-plan 時將自動套用此設定。若需變更，隨時執行 /dev:setup。」
