---
name: cto:setup
description: 設定 CTO plugin 的工作目錄偏好（例如 1on1 紀錄路徑）。安裝 cto plugin 後執行一次，或需要變更路徑時重新執行。
---

# CTO Plugin 設定

## 步驟 1：讀取現有設定

嘗試讀取 `~/.claude/plugins/cto/config.json`：
- 若存在，顯示目前的設定值，詢問使用者是否要修改
- 若不存在，直接進入設定流程

## 步驟 2：詢問設定項目

逐一詢問以下項目（括號內為預設值，使用者可直接按 Enter 套用）：

1. **1on1 紀錄目錄**（預設：`~/1on1`）
   > 「請輸入 1on1 紀錄的存放目錄路徑（預設：~/1on1）：」

## 步驟 3：寫入設定檔

將使用者輸入（或預設值）寫入 `~/.claude/plugins/cto/config.json`：

```json
{
  "1on1_dir": "<使用者輸入的路徑>"
}
```

若 `~/.claude/plugins/cto/` 目錄不存在，先建立它。

## 步驟 4：確認完成

顯示儲存後的設定內容，並說明：

> 「設定已儲存。之後使用 /cto:arch-review 時將自動套用此設定。若需變更，隨時執行 /cto:setup。」
