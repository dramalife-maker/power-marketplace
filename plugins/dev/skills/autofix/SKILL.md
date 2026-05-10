---
name: dev:autofix
description: Test-driven 自動 bug 修復迴圈。寫 failing test 框住 bug → 試 3 個 hypothesis → 修到綠或回報。語言無關。當使用者說「跑 autofix」、「自動修這個 bug」、「/dev:autofix」時觸發。
---

# autofix

Test-driven 的自動 bug 修復迴圈。**不是 grill-me 的設計階段、也不是 post-qa 的審查階段** — 是「bug 已經報出來、我要 skill 幫我框住並嘗試修」的執行階段。

語言無關，依專案實際的測試框架與命名慣例對照。

## 永遠不做

- ❌ Dry-run 模式（兩個 checkpoint 已經是停損點）
- ❌ Auto-commit（成功只 stage、commit 留給使用者）
- ❌ 成功時寫 learning-note（成功不算踩坑、是正常 debug）
- ❌ 跨 hypothesis 累積 diff（每個 hypothesis 失敗都 `git restore`）
- ❌ 動 Q4 scope 以外的檔案（除了新增的測試檔）

---

## Step 0 — Sanity Check

跑 `git status`。**working tree 不乾淨就拒絕啟動**（包含 staged / unstaged / untracked），印出 dirty 檔案清單，建議使用者先 `git stash` 或 `git commit`。

例外：使用者顯式帶 `--allow-dirty` 才繞過。

理由：迴圈會 `git restore` revert，dirty 狀態會被毀掉。

---

## Step 1 — 對齊 4 點（一次列、不一題一題問）

skill 第一輪輸出這段，等使用者一次回答：

```
開跑前確認 4 件事（一次回答即可）：

1. Bug 描述（一句話）：
2. 重現步驟（指令 / curl / 點擊路徑）：
3. Scope（可疑檔案/目錄；不知道請寫「探索」）：
4. 測試模式：
   - write-new（預設）：autofix 自己寫 failing test
   - use-existing:<path>：用現有的 failing test 當合約
   - skip：不寫測試（停止條件改用「build 過 + 你點頭」）
```

使用者帶了 args 就先吃 args、缺什麼補什麼。

第 3 點寫「探索」→ 進入 **Scope 探索子流程**：skill 用 grep / git log / stack trace 推測一個 scope 名單，給使用者點頭再進迴圈。

第 4 點選 `use-existing:<path>` → 跳過 Step 3 的寫測試，直接跑該測試確認真的 fail（綠的話走 11c escalate）。

第 4 點選 `skip` → 全程跳過測試比對；停止條件改成「build 通過 + 使用者每個 hypothesis 後點頭驗證」。

---

## Step 2 — 偵測測試指令（讀／確認／快取）

讀 `~/.claude/plugins/dev/config.json`。找 `autofix_projects[<當前 repo abs path>]`：

```json
{
  "plan_journal_dir": "~/plan-journal",
  "autofix_projects": {
    "E:/workspace/github/power-marketplace": {
      "test_cmd": "go test ./...",
      "test_glob": "**/*_test.go",
      "test_timeout_ms": 300000
    }
  }
}
```

**有命中**：用快取的 `test_cmd / test_glob / test_timeout_ms`，**不再問**。

**沒命中**：掃 `go.mod / Cargo.toml / package.json / pyproject.toml / Makefile` 推測，然後一句話確認：

```
偵測到這是 Go 專案。打算用：
- 測試指令：go test ./...
- 測試檔 glob：**/*_test.go
- 全 suite timeout：5 分鐘（300000ms）

對嗎？(y / 改)
```

使用者點頭（或修改後點頭）→ 寫回 `~/.claude/plugins/dev/config.json` 的 `autofix_projects[<abs path>]`。**不存在的目錄要先建立**。

`test_timeout_ms` 上限 `600000`（10 分鐘，Bash tool 限制）。

---

## Step 3 — 寫／驗 Failing Test

依 Step 1 第 4 點分支：

### write-new（預設）

1. LLM 依 bug 描述 + 重現步驟 + scope 寫一個測試。
2. 測試檔放在專案慣例位置（Go: 同 package `*_test.go`、Rust: `tests/` 或 `#[cfg(test)]` mod、JS/TS: 看 vitest/jest config、Python: `tests/test_*.py`）。
3. 跑這個測試（**只跑這個**，不跑全 suite），預期 FAIL。
4. **判定吻合**：fail 訊息要與 bug 描述對得上（LLM 自我判斷）。例如 bug 說「餘額沒扣」，fail 訊息要看到「balance」「expected 90, got 100」這類。訊息對不上 = 沒框到 bug。
5. **重試上限 1 次**：第 1 次寫的測試沒 fail 或訊息不吻合，允許重寫一次。第 2 次還不行 → escalate：

```
我寫不出能框住這 bug 的測試（試了 2 次都不對）。
最後一次的測試訊息：<...>
是不是 repro 步驟還缺東西？（取消 / 補資訊 / 改用 use-existing）
```

### use-existing:&lt;path&gt;

1. 跑指定測試。
2. **必須 FAIL**。PASS 直接 escalate：

```
你指的測試 <path> 現在是 PASS、不是失敗中的測試。
要切回 write-new 嗎？(y / 取消)
```

3. fail 訊息與 bug 描述吻合度自動判斷；不吻合也警告但允許繼續（因為使用者已明確指定）。

### skip

跳過 Step 3。停止條件改「build 通過 + 使用者每個 hypothesis 後點頭驗證」 — 進 Step 4 時每試完一個 hypothesis、stage 後問使用者「修好了嗎？」。

---

## Checkpoint 1 — Test Framing 確認

非 skip 模式都要過這關。skill 印：

```
✅ Failing test 已就位
   檔案：<path>
   測試名：<name>
   結果：FAIL（符合 bug 描述）
   錯誤訊息（節錄）：<...>

下一步：跑 baseline + 試 3 個 hypothesis。要繼續嗎？(y / 改測試 / 取消)
```

**必須等 explicit y/yes 才繼續**。回「改」就回 Step 3 重寫；「取消」結束 skill。

---

## Step 4 — Baseline

跑 `test_cmd`（全 suite）一次，記下「已經紅的測試清單」存進 skill 暫存（不寫盤、跑完丟）。

`--no-baseline` 旗標跳過此步，改用嚴格模式：之後 regression 判定 = 全 suite 必須全綠（任何紅燈就 revert）。

理由：你有些 repo 全 suite 太慢、跑兩次（baseline + 驗證）會痛。

---

## Step 5 — 修復迴圈（3 hypothesis × 1 attempt × revert）

**進迴圈前**：當前 working tree 應該只有「Step 3 新增的測試檔」。

依 likelihood 排出 3 個 hypothesis（LLM 自己排）。**逐一執行**：

### 對每個 hypothesis：

1. 出最小 patch — **只准動 Q4 scope 內檔案 + 測試檔**。違反就 abort 該 hypothesis。
2. 跑 target test：
   - **紅** → `git restore <scope 內檔案>` → 換下一個 hypothesis
   - **綠** → 進步驟 3
3. 跑全 suite（套 `test_timeout_ms`）：
   - 出現 baseline 沒有的「新紅燈」 → `git restore` → 換下一個
   - 沒有新紅燈 → **跳出迴圈進 Step 6a**

### 三個都失敗：

`git restore` 確認乾淨 → 進 **Checkpoint 2 / Step 6b**。

### Skip 模式特例

沒測試可比對時，每個 hypothesis 改成：
1. 出最小 patch
2. 跑 build（語言對應的 build 指令；偵測同 Step 2）
3. build 過 → stage 後問使用者「修好了嗎？」 — 點頭跳 Step 6a；搖頭 `git restore` 換下一個

---

## Step 6a — 成功收尾

成功的 hypothesis 已留在 working tree。執行：

1. `git add` — 加入「scope 內被改的檔案 + 新增的測試檔」（不 commit）
2. 印報告：

```
✅ Bug 已修復（hypothesis #<n> 命中）

成功的 hypothesis：
  <一行描述>

修改檔案（已 staged）：
  M  internal/billing/checkout.go
  A  internal/billing/checkout_test.go

下一步：
  git diff --staged   # 審查
  /dev:commit-message # 產 commit 訊息
```

3. **結束**。不寫 learning-note、不 auto-commit。

---

## Step 6b — 失敗收尾（Checkpoint 2）

3 個 hypothesis 都失敗時：

1. 確認 `git status` 乾淨（只剩 Step 3 的測試檔可能還在 — 也 revert 掉，避免污染）
2. 印報告：

```
❌ 3 個 hypothesis 都沒修好。已 revert 所有改動。

H1 <一行描述>
   失敗原因：<target test 仍紅 / 全 suite 多 N 個新紅 / build fail / scope 違規>
H2 <一行描述>
   失敗原因：...
H3 <一行描述>
   失敗原因：...

已委派 sub-agent 在背景寫 learning note（不阻塞）。
建議下一步：
  (a) 給更多線索我重跑
  (b) 你親自接手 debug
  (c) 取消
```

3. **委派 post-bug sub-agent 在背景寫 learning-note**（見下節）
4. **不等使用者回應**，skill 結束

---

## Learning Note 委派（只在 Step 6b 觸發）

不直接 invoke `dev:post-bug` skill（兩 skill 互呼會 context 爆炸）。改自己呼叫 `Agent` tool：

- `run_in_background: true`
- `model: "haiku"`
- `subagent_type: "general-purpose"`
- `prompt`：自包含、不依賴對話 context，內容包含：
  - 今日日期（`YYYY-MM-DD`）
  - 錯誤類型推測（typo / knowledge-gap / architecture-defect / requirement-misunderstanding / race-condition / technical-debt — 可多選）
  - Bug 描述、重現步驟、scope
  - 3 個失敗 hypothesis 的描述與失敗原因
  - 使用者意圖（從對話推：為什麼回報這 bug、想避免什麼）
  - 完整內嵌 [post-bug SKILL.md](../post-bug/SKILL.md) 的撰寫規範（TL;DR 格式、index.md 更新規則、資料夾結構等）
  - 明確指示：寫進 `<repo>/.claude/docs/learning-notes/<YYYY-MM-DD>/<分類>.md`、更新 `index.md`（**讀後新增列、禁止覆寫**）

寫成功不寫 learning-note（重申）。

---

## 邊緣情境快查

| 情境 | 處理 |
|------|------|
| 進場 working tree 不乾淨 | 拒絕，要 `--allow-dirty` 才繞過 |
| LLM 寫測試 2 次都框不到 bug | escalate，不硬上 |
| `use-existing` 指的測試是 PASS | escalate，不自動診斷 |
| 全 suite 超過 `test_timeout_ms` | Bash 砍掉、視同失敗該 hypothesis |
| baseline 太慢 | `--no-baseline` 旗標 + 嚴格模式 |
| Hypothesis 想改 scope 外檔案 | abort 該 hypothesis、視同失敗 |
| 3 個 hypothesis 全失敗 | Step 6b（revert + 背景寫 learning-note + 結束） |
| skip 模式 | 每個 hypothesis stage 後等使用者點頭 |

---

## 操作原則

- **誠實回報**：fail 訊息不吻合就講不吻合、不硬拗成功
- **明確點頭**：Checkpoint 1 必須等 explicit y/yes、不允許「使用者沒回就跑」
- **乾淨切段**：每個 hypothesis 失敗一定 `git restore`，不堆改
- **單一職責**：learning-note 只委派、不重做 post-bug 的工作
- **不擴張**：scope 外檔案永遠不動，包含成功 hypothesis 也不允許「順便改一下」
