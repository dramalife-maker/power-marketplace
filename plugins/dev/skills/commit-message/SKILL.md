---
name: dev:commit-message
description: Guide for writing commit messages following Conventional Commits specification with emoji support. Use when asked to create commit messages, format commits, or validate commit message structure.
metadata:
    author: "power"
    version: "1.0.1"
---

# Commit Message Guide

This skill provides comprehensive guidance for writing commit messages that follow the Conventional Commits 1.0.0 specification with emoji support.

## Purpose

Ensure all commit messages in the project follow a consistent, standardized format that:
- Makes commit history explicit and readable
- Enables automated tooling (changelog generation, versioning, etc.)
- Aligns with Semantic Versioning principles
- Includes emoji for visual clarity

## When to Use

Use this skill when:
- Creating or formatting commit messages
- Validating commit message structure
- Explaining commit message conventions
- Generating commit messages from code changes

## Workflow for Suggesting Commit Messages

When a user asks for a suggested commit message, follow this process:

1. **Check staged changes**: First, run `git --no-pager diff --staged` to see what files are staged for commit
2. **Analyze changes**: Review the diff output to understand:
   - Which files were modified
   - What types of changes were made (features, fixes, refactors, etc.)
   - The scope and impact of the changes
3. **Generate commit message**: Based on the staged changes, create an appropriate commit message following the format and rules below
4. **Validate**: Ensure the message follows Conventional Commits specification with emoji support

## Core Format

```
<emoji> <type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Body Format Rules

- **Use bullet points**, not prose paragraphs
- **Each bullet: 1–2 sentences max** — state the point and stop
- Focus on **why**, not what (the diff already shows what changed)
- Omit the body entirely if the subject line is self-explanatory

**Good:**
```
- Replace ⚠️ with ❌ to signal "must fix before merge", not just a warning.
- Each issue now requires an actionable fix with file path or code snippet.
- Step 4 skips confirmation if user intent is already explicit in args.
```

**Bad:**
```
This commit improves the review format by changing the warning emoji to a
red cross emoji in order to make it clearer to reviewers that the issues
listed are blocking issues that must be resolved before the MR can be merged...
```

## Key Rules

1. **Emoji is REQUIRED** - Every commit must start with an appropriate emoji
2. **Type is REQUIRED** - Must be one of: `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, etc.
3. **Description is REQUIRED** - Short summary of changes
4. **Scope is OPTIONAL** - Contextual information in parentheses
5. **Breaking Changes** - Use `!` after type/scope or `BREAKING CHANGE:` footer

## References

- [Conventional Commits Specification](references/commit-message.md) - Complete specification with examples and emoji guide
