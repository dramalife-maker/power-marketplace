---
name: dev:everything-search
description: Search the whole Windows machine (all drives, instant, index-based) for a file or folder by name/path using the Everything CLI (`es`, must be on PATH). Use when the user asks to find/locate a file or folder "on this PC / on my computer / anywhere on disk", not just inside the current repo. Do NOT use for searching file contents or for scoped, in-repo lookups — use Grep/Glob for those.
---

# Everything Search (es.exe)

Wraps voidtools' Everything CLI to find files/folders by name across the entire machine instantly, instead of walking the filesystem.

## Prerequisite

Requires `es` (the Everything CLI) to be on PATH, and the Everything background service (Everything.exe) running and indexing. If `es` isn't found, or a command fails with `Error 8: Everything IPC not found`, tell the user the CLI/service isn't set up and fall back to Glob/Bash `find` — do not retry in a loop.

## When to use vs Glob/Grep

- Glob/Grep: searching within the current project/repo.
- everything-search: "where is this file on my computer", "find X anywhere on disk", cross-drive/cross-folder lookups outside the current repo.

## Usage

Binary: `es` (resolved via PATH). Always cap results to avoid flooding context.

```bash
es -n 20 <keyword>              # plain name/path substring search, max 20 results
es -n 20 -r "<regex>"           # regex search
es -n 20 -whole-word <name>     # whole word match
es -n 20 /a-d <keyword>         # files only (no folders)
es -n 20 /ad <keyword>          # folders only
es -n 20 -path "<dir>" <keyword># limit search to a subtree
es -n 20 -p <keyword>           # match against full path, not just filename
```

In the Bash tool (Git Bash), `/a-d` and `/ad` get rewritten into Windows paths. Prefix the command with `MSYS_NO_PATHCONV=1`, or run it in the PowerShell tool.

Key flags:
- `-n <num>` — max results (always set this)
- `-r`/`-regex` — regex search
- `-i`/`-case` — **enables** case-sensitive matching (opposite of grep's `-i`!). Never add this on Windows unless the user explicitly wants case-sensitive matching — default (with or without `-r`) is already case-insensitive, which matches Windows filesystem semantics.
- `-w`/`-whole-word` — whole word match
- `-p`/`-match-path` — match full path instead of filename only
- `/a-d` — files only, `/ad` — folders only
- `-path <dir>` — restrict to subtree
- `-sort <field>` — e.g. `-sort date-modified-descending` for newest first

Full flag reference: https://www.voidtools.com/support/everything/command_line_interface/
