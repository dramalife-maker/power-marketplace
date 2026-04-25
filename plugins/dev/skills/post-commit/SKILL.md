---
name: dev:post-commit
description: 請根據以下步驟，提供 commit message 的建議。
---

## 步驟 1：分析變更內容

執行以下指令，了解這次的 staged 變更：

```bash
git --no-pager diff --staged
```

## 步驟 2：依照 dev:commit-message skill 草擬訊息

依照 `dev:commit-message` skill 的規範，草擬一個 commit message。

規則：
1. 全使用英文描述，不使用中文。
2. 重點在於「為什麼這樣改」或「解決了什麼問題」，而非單純描述「改了什麼」。

## 步驟 3：對照 repo 慣用格式做最終微調

執行以下指令，查看這個 repo 最近的 commit 歷史：

```bash
git --no-pager log --oneline -15
```

觀察以下細節，並在不違背 `dev:commit-message` skill 核心規範的前提下，讓格式與 repo 現有風格一致：
- emoji 的使用方式（實際 emoji 或 `:emoji:` 格式，或是否省略）
- scope 是否為必填、常用哪些 scope
- 其他一致性的寫法習慣

最終輸出符合該 repo 慣例的 commit message 建議。

## 步驟 4：詢問是否執行 commit

提出建議後，詢問使用者：「若無問題，是否要協助執行 git commit？」

- 若使用者確認，執行 `git commit` 並帶入上述建議的 commit message。
- 若使用者有修改意見，先調整訊息再確認。
