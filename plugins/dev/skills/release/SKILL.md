---
name: dev:release
description: Manage the software release process. Use when asked to prepare a release, update changelogs, or tag a version.
metadata:
    author: "power"
    version: "1.1.2"
---

# Release Assistant

This skill guides the AI through the formal release process, ensuring that Changelogs are accurate, commits are standardized, and tags are correctly pushed.

## Workflow

1.  **Update Changelog**:
    -   Find `CHANGELOG.md` file in the root directory. If not found, create it using the [Changelog Template](templates/changelog-template.md).
    -   Follow [Write Changelog Rules](references/write-changelog.md).
    -   Use [Read Git Log](references/read-git-log.md) to gather changes since the last version.
    -   Verify the date and version number.
    -   **Update the version comparison links block at the bottom of the file** (do not omit):
        -   Replace the existing `[Unreleased]` link to point at the new version tag: `[<new-version>]: .../compare/v<previous-version>...v<new-version>`
        -   Re-add `[Unreleased]: .../compare/v<new-version>...<current-branch>` at the top of the links block.
        -   If the links block is missing, add it per [write-changelog.md](references/write-changelog.md#版本連結).
2.  **Review Changes**:
    -   Summarize the changes and suggested version number to the user.
    -   Wait for user confirmation that the changes are correct.
3.  **Commit Release**:
    -   Once confirmed, commit the changes using `git add CHANGELOG.md`.
    -   Create a commit message following this format: `:bookmark: <version>`.
        -   Example: `:bookmark: v1.0.0`
    -   Wait for user confirmation that the commit is created.
4.  **Tag & Push**:
    -   Check existing tags to match format: `git tag --list --sort=-version:refname | head -10`.
    -   Create tag: `git tag <version>`.
    -   Push branch and tag: `git push origin <current-branch> <version>`.
5.  **Merge to Release Branch** (if applicable):
    -   Check whether any of the following branches exist: `release`, `master`, `main`:
        ```bash
        git branch --list release master main
        ```
    -   If one or more exist, record the current branch name, then use the **AskUserQuestion tool** to ask:
        > "Detected branch(es): `<branch-list>`. Would you like to switch to one of them and merge `<current-branch>` in?"
        - If multiple branches are found, list each as a separate option for the user to choose from.
        - Add a "Skip" option as the last choice.
    -   If the **AskUserQuestion tool** is not available, ask as plain text and wait for the user's response.
    -   If the user confirms:
        -   `git switch <target-branch>`
        -   `git merge <current-branch>`
        -   Report the result and stop.
    -   If the user declines, skip this step.

## References

-   [Write Changelog Rules](references/write-changelog.md)
-   [Read Git Log](references/read-git-log.md)
-   [Changelog Template](templates/changelog-template.md)
