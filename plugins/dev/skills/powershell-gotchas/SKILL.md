---
name: dev:powershell-gotchas
description: >-
  Knowledge base of PowerShell-on-Windows pitfalls and their tested workarounds.
  Consult this whenever you are about to write or run a non-trivial PowerShell
  command on Windows — especially git commits with multi-line messages, invoking
  native executables, here-strings, reaching for a Unix command (head/tail/which/
  touch/wc), setting environment variables, or anything where CRLF line endings
  might bite. Use it even for "simple" commands if they involve those areas, since
  PowerShell's syntax and available commands differ from bash in ways that silently
  corrupt output rather than erroring. When you hit a new PowerShell gotcha, append
  it here so it is not rediscovered the hard way.
---

# PowerShell Gotchas

A running knowledge base of things that bite when running commands through
PowerShell on Windows. PowerShell is *not* bash — much of the muscle memory
from POSIX shells produces wrong-but-silent results here. Each entry below is a
confirmed pitfall plus the workaround that actually works.

When you discover a new gotcha, **add an entry** using the template at the bottom.
Keep entries short and concrete: the symptom, the cause, the fix.

---

## Quick reference table

| Area | Don't | Do |
|------|-------|----|
| Multi-line `git -m` | here-string `@'...'@` in the Bash tool | here-string in the PowerShell tool, or repeated `-m` flags |
| `head -n` / `tail -n` | `head file` / `tail file` | `Get-Content file -TotalCount N` / `-Tail N` |
| `which cmd` | `which cmd` | `(Get-Command cmd).Source` |
| Discard stderr | `2>/dev/null` | `2>$null` |
| Env var (read) | `$VAR` | `$env:VAR` |
| Env var (set) | `export VAR=x` | `$env:VAR = 'x'` |
| Null device | `/dev/null` | `$null` |
| Run PS syntax via Bash tool | `if (...) {...}` in the Bash tool | use the PowerShell tool instead |

---

## Entries

### git commit -m with here-string leaks literal `@` in the Bash tool

**Symptom:** A commit made with `git commit -m @'...'@` ends up with a literal
`@` as its own line at the very start and a `@` at the end of the message.

**Cause:** The command ran through the Bash tool. Bash reads `@'...'@` as a
literal `@`, a quoted string, and another literal `@`. PowerShell 7 parses the
same here-string correctly.

**Fix:** Run it in the PowerShell tool, with the closing `'@` at column 0:

```powershell
git commit -m @'
:sparkles: subject line

Body paragraph explaining why.
'@
```

From the Bash tool, use one `-m` flag per paragraph or `git commit -F file`.

---

### Unix text commands don't exist

**Symptom:** `head`, `tail`, `which`, `touch`, `wc -l`, `sed`, `awk` either error
or resolve to something unexpected.

**Fix:** Use the PowerShell equivalent:

- `head -n N file` → `Get-Content file -TotalCount N`
- `tail -n N file` → `Get-Content file -Tail N`
- `which cmd` → `(Get-Command cmd).Source`
- `touch file` → `if (-not (Test-Path file)) { New-Item -ItemType File file }`
  (never `New-Item -Force` on an existing file — it truncates it)
- `wc -l file` → `(Get-Content file | Measure-Object -Line).Lines`

For reading, searching, and editing files, prefer the dedicated Read/Grep/Glob/Edit
tools over any of these anyway.

---

### Environment variables and the null device

**Symptom:** `$VAR` is empty; `export VAR=x` errors; `2>/dev/null` writes a file
called `null`.

**Fix:**

- Read: `$env:NAME` (not `$NAME`)
- Set for the session: `$env:NAME = 'value'`
- Inline-for-one-command (bash `VAR=x cmd`): `$env:NAME = 'x'; cmd` — PowerShell
  has no inline env-var prefix
- Redirect/discard: `$null`, e.g. `cmd 2>$null` or `cmd > $null`

---

### CRLF line-ending churn on `git add`

**Symptom:** `git` warns `LF will be replaced by CRLF the next time Git touches it`,
and files show as modified with no real content diff (`git diff file` shows nothing).

**Cause:** Windows checkouts normalize line endings; a file can appear "modified"
purely from EOL conversion.

**Fix:** Before committing such a file, confirm there's a real change with
`git diff <file>`. If the diff is empty (EOL-only), leave it out of the commit
rather than staging a noise change. The warning itself is harmless.

---

### Running PowerShell syntax through the Bash tool errors out

**Symptom:** A command like `if (Test-Path x) { Remove-Item x }` fails with
`syntax error near unexpected token '{'` — the parentheses-and-brace form is
not valid bash.

**Cause:** The Bash tool runs commands through bash, not PowerShell. PowerShell
control-flow (`if (...) {...}`, `foreach`, `Test-Path`, cmdlets) is bash syntax
error territory; bash expects `if [ ... ]; then ... fi`.

**Fix:** Run PowerShell syntax through the PowerShell tool, not the Bash tool.
Reserve the Bash tool for POSIX/bash one-liners. If you must invoke PowerShell
from a bash context, wrap it: `pwsh -c '<powershell here>'`.

---

## Adding a new entry

Append under `## Entries` using this shape — keep it to symptom / cause / fix so
it stays scannable:

```markdown
### <short title of the gotcha>

**Symptom:** What you observe going wrong.

**Cause:** Why PowerShell does this (one or two sentences).

**Fix:** The command/pattern that works, with a code block if useful.
```

Also add a one-line row to the **Quick reference table** when the gotcha has a
clean don't/do pair.
