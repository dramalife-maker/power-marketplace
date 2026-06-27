---
name: "power-cto"
displayName: "CTO Workflow"
description: "Tech lead toolkit: architecture reviews, 1on1 management, performance reviews, and GitLab MR scanning"
keywords: ["arch review", "architecture review", "1on1", "one-on-one", "performance review", "perf review", "quarterly review", "MR", "merge request", "scan MR", "code review", "tech lead", "CTO", "engineering manager", "team lead", "review engineer"]
author: "powersafe.lin@gmail.com"
---

# CTO Workflow Power

This power provides tech-lead workflow assistance: architecture reviews, 1on1 session management, quarterly performance reviews, and GitLab MR scanning with automated review publishing.

Config is stored at `~/.kiro/powers/cto/config.json`. On first use of any workflow that requires config, check if this file exists. If not, prompt:

> "What directory should I use for 1on1 records? (default: `~/1on1`)"

Then write: `{ "1on1_dir": "<path>" }` to that file.

Throughout these workflows, `<1on1_dir>` refers to the value from this config.

---

## Workflow: Architecture Review

**Triggers:** user says "review this architecture", "review this design", "look at this diff", "arch review", or submits a design document / PR diff / code draft for technical review.

### Step 1: Gather context

Confirm (use what's already in conversation, don't re-ask):
1. Engineer's name — for writing to 1on1 records
2. Review target — design doc, code draft, PR diff, or mixed
3. Project standards source — prefer `CLAUDE.md` in current or root dir; fall back to general criteria below

### Step 2: Execute the review

**For design documents / architecture descriptions:**
- Logical correctness and completeness
- Scalability and performance considerations
- Missing edge cases and failure modes
- Compatibility with existing systems
- Whether decisions are well-justified (alternatives evaluated?)

**For code / PR diffs:**
- Logic correctness
- Performance impact (DB queries, index usage, N+1)
- Better alternatives
- Test coverage adequacy

**SQL specialist review:** If the code contains SQL queries (ORM, raw SQL, migrations), analyze for:
- Index usage and lock contention
- N+1 query patterns
- Transaction boundary correctness
- Security (injection risks)

Report SQL issues with severity: High / Medium / Low.

### Step 3: Output format

```
## Review Summary: <feature name>

### ✅ Done well
- ...

### ❌ Must fix
- [High] <issue> → <specific fix with file path or code snippet>
- [Medium] ...
- [Low] ...

### ❓ Questions to ask the engineer
- ...

### 💡 Overall assessment
<Synthesis judgment, including observed thinking patterns of the engineer.>
```

> Use ❌ not ⚠️ — ⚠️ is ambiguous. ❌ means "must fix before merge." Every issue must include an actionable fix.

### Step 4: Write to 1on1 record (optional)

After review, ask: "Should I write this summary to `<1on1_dir>/<engineer>/`?"

If confirmed, append to `<1on1_dir>/<engineer>/YYYY-MM.md`:

```markdown
## YYYY-MM-DD Architecture Review: <feature name>

**Background**: <one-sentence purpose>

**Engineer's proposal highlights**:
- ...

**Review findings**:
- [High] ...
- [Medium] ...

**Observed thinking patterns**:
<Decision habits, blind spots, or growth signs from this review>

**Follow-up**:
- [ ] <items sent back for revision>
```

Update `index.md` with a new row (type: `架構審查`).

### General review criteria (when no CLAUDE.md exists)

- Separation of concerns and layering
- Naming consistency with the codebase
- Error handling at system boundaries
- Absence of premature optimization
- Testability of the design

---

## Workflow: 1on1 Preparation

**Triggers:** user says "prepare 1on1", "prep for 1on1 with <name>", or "look at <engineer>'s records".

### Step 1: Identify engineer

Use name from conversation if clear; otherwise ask.

### Step 2: Read history

Read these files in order (skip if missing):
1. `<1on1_dir>/<engineer>/index.md` — timeline of past sessions
2. `<1on1_dir>/<engineer>/_notes.md` — long-term patterns and tracked items
3. `<1on1_dir>/<engineer>/YYYY-MM.md` — current month's post-session notes
4. `<1on1_dir>/<engineer>/YYYY-MM-agenda.md` — current month's pre-collected topics

If the directory doesn't exist at all, proceed with an empty framework.

### Step 3: Generate prep report

```markdown
# 1on1 Prep: <engineer> (YYYY-MM-DD)

## Open follow-up items
<!-- From past YYYY-MM.md: unchecked [ ] items with source date -->
- [ ] <item> (from YYYY-MM-DD)

## Long-term observations to revisit
<!-- From _notes.md tracked items -->
- <observation>

## Pre-collected topics (from agenda)
<!-- From 1on1-add entries -->
- [High] **<topic>** — <one-line context>

## Suggested new topics
<!-- Derived from recent context: review outcomes, growth signals, blockers -->
1. <topic>
2. <topic>

## Already discussed this month
<!-- Brief summary of existing YYYY-MM.md entries -->
- YYYY-MM-DD: <one-line summary>
```

Show `(none)` for empty sections; don't skip them.

---

## Workflow: 1on1 Session Notes

**Triggers:** user is in or just finished a 1on1 meeting and wants to record it.

Capture the session and write to `<1on1_dir>/<engineer>/YYYY-MM.md`:

```markdown
## YYYY-MM-DD 1on1

**Discussion highlights**:
- ...

**Decisions / agreements**:
- ...

**Observed thinking patterns**:
<Engineer's growth signs, recurring concerns, or mindset shifts>

**Follow-up**:
- [ ] <action item with owner>
```

Update `<1on1_dir>/<engineer>/index.md` with a new row (type: `1on1`).

If the session surfaces a **long-term pattern** (something observed across multiple sessions), append to `_notes.md`.

---

## Workflow: Add 1on1 Topic

**Triggers:** user wants to queue a topic for a future 1on1 ("add this to next 1on1 with <name>", "note for 1on1").

Append to `<1on1_dir>/<engineer>/YYYY-MM-agenda.md`:

```markdown
### [High|Medium|Low] <topic title>
**Context**: <one-line background>
**Goal**: <what you want to discuss or decide>
```

Create the file if it doesn't exist.

---

## Workflow: Quarterly Performance Review

**Triggers:** user says "performance review", "perf review", "quarterly review", or "rate <engineer>".

### Step 1: Identify engineer and quarter

Derive quarter from current date (Q1=Jan–Mar, Q2=Apr–Jun, Q3=Jul–Sep, Q4=Oct–Dec). Confirm with user.

### Step 2: Read quarterly 1on1 records

Read all `YYYY-MM.md` files for the quarter plus `_notes.md`. If missing, proceed based on user's judgment.

### Step 3: Guide scoring — one dimension at a time

For each dimension, show related observations from records, then ask for a score (1–5) and rationale.

| Dimension | Description |
|-----------|-------------|
| Technical judgment | Can they evaluate options, spot AI output issues, make correct trade-offs at decision points? |
| Debugging ability | Do they find root causes systematically, not just patch symptoms? |
| Delivery | On-time, scope control, consistent output quality |
| Initiative | Proactively identifies problems and improvements without being told |
| Communication & collaboration | Cross-team comms, documentation quality, collaboration efficiency |
| Growth rate | Speed of absorbing feedback, improvement vs. last quarter |

Score definitions: 5 = far exceeds expectations | 4 = exceeds | 3 = meets | 2 = partially below | 1 = significantly below

### Step 4: Draft and confirm

Calculate average (round to 1 decimal). Show draft for user confirmation:

```markdown
## YYYY-QN Quarterly Performance

| Dimension | Score | Notes |
|-----------|-------|-------|
| Technical judgment | X | ... |
| Debugging | X | ... |
| Delivery | X | ... |
| Initiative | X | ... |
| Collaboration | X | ... |
| Growth rate | X | ... |
| **Total** | **X.X** | |

**Overall assessment**:
<Key highlights and areas for improvement this quarter>

**Next quarter focus**:
- <specific recommendation>
```

### Step 5: Write to performance record

Append to `<1on1_dir>/<engineer>/_perf.md` (never overwrite). Keep a cumulative trend table at the top, detailed records below in reverse-chronological order.

---

## Workflow: Scan & Review GitLab MRs

**Triggers:** user says "scan MRs", "scan mrs", "review MRs", or provides a specific MR reference.

**Tools required:** `glab` CLI (logged into git.ttech.cc), GitLab account: `zhaizhai`

### Quick mode (specific MR)

If user provides a specific MR (e.g., "transaction/server!72"), skip the scan and go directly to the review step.

### Full scan

```bash
glab api "merge_requests?state=opened&scope=all&reviewer_username=zhaizhai"
```

For each MR, check notes to determine review round:
- No prior AI review → **Round 1**
- AI review exists AND (new commits or author replies after it) → **Round 2**
- AI review exists, no new activity → **Skip**

AI review notes are identified by `By: AI Agent` in the body.

Show summary:
```
✅ Needs review (N)
  - project/repo!55 — [title] (Round X)

⏭️ Skipped (N)
  - project/repo!47 — [title]
```

### Review execution

For each MR to review:

1. Fetch diff: clone or fetch the source branch into `~/.kiro/review-repos/<project-path>/`
2. Read changed files (source branch vs. target branch)
3. Compare against sibling modules in the same directory for convention parity

**Six review dimensions:**
1. Coding style — naming, error handling, import style
2. Architecture — layering, dependency direction, module boundaries
3. Documentation — comments, API contracts, migration notes
4. Simplification — duplicated logic, deep nesting, over-engineering
5. Convention parity — does it follow existing module patterns? Dead code?
6. Actionable fixes — every issue includes a file path, symbol name, or example

**Round strategy:**
- Round 1: Favor ❓ (questions to ask) over ❌ (must fix); be conservative with hard judgments
- Round 2: Start with "Previous round issues: resolved / unresolved" table; focus ❌ on confirmed bugs

**Output format:**
```markdown
## Review: {MR title} (Round X)

### 📋 Previous round status (Round 2 only)
| Item | Status | Notes |
|------|--------|-------|

### ✅ Done well
- ...

### ❌ Must fix
| Severity | Issue | Fix |
|----------|-------|-----|
| [High] | ... | ... |

### ❓ Questions to ask
- ...

### 💡 Overall assessment

By: AI Agent
```

**Fact-check before publishing:** For any `[High]` item, verify with actual code paths before including it. If uncertain → downgrade to ❓.

4. Show draft to user for confirmation before publishing.
5. Publish: `glab mr note {iid} -R {project} --message "{review}"`
6. Ask if review summary should be written to `<1on1_dir>/<author>/YYYY-MM.md`.
7. If no ❌ items: ask if user wants to merge (`glab mr merge {iid} -R {project} --remove-source-branch`).

**Never publish without user confirmation.**
