---
name: "power-dev"
displayName: "Dev Workflow"
description: "Developer workflow toolkit: commit messages, releases, bug post-mortems, code review, QA, and architecture decision records"
keywords: ["commit", "commit message", "conventional commits", "release", "changelog", "bug", "post-mortem", "learning notes", "code review", "PR", "pull request", "QA", "test", "ADR", "architecture decision", "PowerShell", "autofix", "grill me", "stress test", "plan"]
author: "powersafe.lin@gmail.com"
---

# Dev Workflow Power

This power provides developer workflow assistance across the full development lifecycle: from writing commit messages to releasing versions, capturing bugs, reviewing code, and recording architecture decisions.

Config is stored at `~/.kiro/powers/dev/config.json`. On first use of any workflow that requires config, check if this file exists. If not, prompt:

> "What directory should I use for plan journals? (default: `~/plan-journal`)"

Then write: `{ "plan_journal_dir": "<path>" }` to that file.

---

## Workflow: Commit Message

**Triggers:** user asks to write/format/validate a commit message, or wants to commit staged changes.

1. Run `git --no-pager diff --staged` to inspect staged changes.
2. Run `git --no-pager log --oneline -15` to learn the repo's commit style (emoji format, scope conventions, etc.).
3. Draft a message following **Conventional Commits 1.0.0** with emoji:

   ```
   <emoji> <type>[optional scope]: <description>

   [optional body — bullet points only, focus on WHY not WHAT]
   ```

4. Common types and emoji:

   | Type | Emoji | Use when |
   |------|-------|----------|
   | feat | ✨ | new feature |
   | fix | 🐛 | bug fix |
   | chore | 🔧 | maintenance |
   | docs | 📝 | documentation |
   | refactor | ♻️ | restructuring without behavior change |
   | test | ✅ | tests |
   | perf | ⚡ | performance improvement |
   | release | 🔖 | version tag |

5. Rules:
   - Emoji is **required**
   - Body uses **bullet points**, not prose
   - Each bullet: 1–2 sentences max, explain WHY
   - Omit body if subject line is self-explanatory
   - Align emoji/scope style with the repo's existing commits

6. After drafting, ask: "Shall I run `git commit` with this message?"

---

## Workflow: Release

**Triggers:** user asks to prepare a release, update changelog, or tag a version.

1. **Update Changelog** (`CHANGELOG.md` in repo root; create from template if missing):
   - Read recent git log: `git --no-pager log --oneline <last-tag>..HEAD`
   - Group changes by type (Added / Changed / Fixed / Removed)
   - Verify date and version number follow semver
   - Update version comparison links block at the bottom (replace `[Unreleased]` link, add new version link, re-add `[Unreleased]` at top)

2. **Review with user** — summarize changes and proposed version number, wait for confirmation.

3. **Commit**: `git add CHANGELOG.md` → commit with message `:bookmark: <version>`

4. **Tag & Push**:
   - Check existing tag format: `git tag --list --sort=-version:refname | head -10`
   - `git tag <version>` → `git push origin --tags`

5. **Merge to release branch** (if applicable):
   - Check for `release`, `master`, `main` branches
   - If found, ask user whether to merge current branch in

---

## Workflow: Bug Post-Mortem (Learning Notes)

**Triggers:** user asks to record a bug, post-mortem, or "what went wrong"; after completing a fix session.

Delegate the writing to a **background sub-agent** (model: haiku) to avoid blocking conversation.

1. Main agent extracts from conversation:
   - Date (`YYYY-MM-DD`)
   - Error type(s): `typo` / `knowledge-gap` / `architecture-defect` / `requirement-misunderstanding` / `race-condition` / `technical-debt`
   - Problem description, root cause, fix, prevention strategy
   - **User's intent** — why they wanted the change, what risk they were avoiding
   - Related file paths (if any)

2. Spawn background sub-agent with all of the above as a self-contained prompt.

3. Sub-agent writes to `.claude/docs/learning-notes/YYYY-MM-DD/<type>.md` with this structure:

   ```
   <tl;dr>
   - When to recall this: <context/trigger>
   - Don't: <2–4 anti-patterns>
   - Do: <2–4 correct patterns>
   - Intent: <one-line user goal>
   </tl;dr>

   ## Why the user wanted this change (Intent)
   <derive from conversation — what risk/goal drove this>

   ## Problem description
   ## Root cause
   ## Fix
   ## Prevention
   ```

4. Sub-agent updates `.claude/docs/learning-notes/index.md` (append row only, never overwrite).

---

## Workflow: Pre-Review (Code Review Prep)

**Triggers:** user says "review this", "pre-review", or wants to check changes before submitting a PR.

Two-stage review:

**Stage 1 — Quick scan** (before reading full diff):
- Run `git --no-pager diff HEAD` or read the provided diff
- Flag obvious issues: typos in names, dead code, missing error handling at boundaries, test coverage gaps

**Stage 2 — Deep review**:
- Read changed files in full
- Check: logic correctness, performance (N+1, unnecessary queries), naming consistency, test coverage
- For SQL: flag index usage, lock contention, N+1 patterns

Output format:
```
## Pre-Review: <branch or feature name>

### ❌ Must fix before merge
- [High] <issue> → <specific fix with file:line>

### ⚠️ Suggestions
- <optional improvement>

### ✅ Looks good
- <positive notes>
```

---

## Workflow: Post-QA

**Triggers:** user finished a QA session and wants to capture test insights, or asks "post-qa".

1. Ask what was tested and what was found (if not already in conversation).
2. Summarize:
   - Test scenarios covered
   - Bugs found (with severity)
   - Gaps identified (untested scenarios)
   - Confidence level for release
3. Optionally write a QA summary to `.claude/docs/qa-notes/YYYY-MM-DD.md`.

---

## Workflow: Architecture Decision Record (ADR)

**Triggers:** user wants to record an architecture decision, mentions "ADR", or "let's document this decision".

Create an ADR file at `docs/adr/YYYY-MM-DD-<slug>.md`:

```markdown
# ADR: <title>

**Date:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated | Superseded

## Context
<Why is this decision needed? What forces are at play?>

## Decision
<What was decided?>

## Consequences
### Positive
- ...
### Negative / Trade-offs
- ...

## Alternatives considered
- **<Alt A>**: <why rejected>
- **<Alt B>**: <why rejected>
```

---

## Workflow: Autofix (Test-Driven Bug Fixing)

**Triggers:** user wants to fix a failing test or bug autonomously, mentions "autofix".

1. Identify the failing test(s) — run the test suite if needed.
2. Read the failure output and the relevant source files.
3. Form a hypothesis about the root cause.
4. Apply the minimal fix.
5. Re-run the test to verify.
6. If still failing, iterate (up to 5 rounds before reporting stuck).
7. Once passing, offer to write a learning note via the Bug Post-Mortem workflow.

---

## Workflow: Grill Me (Plan Stress-Test)

**Triggers:** user says "grill me", "stress-test my plan", or wants thorough questioning of a design.

Interview the user relentlessly about every aspect of their plan:
- Walk down each branch of the decision tree
- Resolve dependencies between decisions one-by-one
- For each question, provide your **recommended answer**
- Ask questions **one at a time**
- If a question can be answered by exploring the codebase, explore first

---

## Workflow: Archive Plan

**Triggers:** user wants to archive or journal a completed plan/task.

Save a plan summary to `<plan_journal_dir>/YYYY-MM-DD-<slug>.md` (read config for `plan_journal_dir`):

```markdown
# Plan: <title>

**Date:** YYYY-MM-DD
**Status:** Completed

## Goal
## What was done
## Key decisions
## Lessons learned
```

---

## PowerShell Gotchas

**Triggers:** user is writing PowerShell scripts or asks about PowerShell behavior.

Key differences from Bash to watch for:
- `&&` does not work in PowerShell — use `;` or `if ($LASTEXITCODE -eq 0)` chaining
- String interpolation uses `"$var"` (double quotes only; single quotes are literal)
- `$null` vs empty string: check with `-eq $null`, not `-z`
- Paths use `\` but PowerShell also accepts `/`; prefer `Join-Path` for cross-platform safety
- Exit codes: check `$LASTEXITCODE` (external commands) vs `$?` (cmdlets)
- `Write-Host` vs `Write-Output`: Host goes to console only, Output goes to pipeline
