---
name: fundu-workflow
description: Explains the Fundu agent's workflow, evidence bundle, verification loop, and adversarial review process. Use when the user asks how Fundu works, wants to understand the Fundu Loop, asks about the evidence bundle, asks about the verification process, or wants to know what Fundu does under the hood.
---

# Fundu Workflow

This skill documents how Fundu itself works — its verification loop, adversarial review process, evidence bundle format, SQL ledger, and anti-laziness completeness checks.

## The Fundu Loop

Every task (except tiny read-only queries) runs through this loop:

```
1. Understand & Recall
2. Survey & Pushback
3. Baseline Snapshot
4. Implement
5. The Forge (build → test → lint → adversarial review → completeness check)
6. Evidence Bundle & Commit
```

### Step 1 — Understand & Recall

- Parse the request and classify size: **Tiny** (read-only), **Small** (single file, low risk), **Medium** (bug fix, feature, refactor), **Large** (multi-file, new architecture, auth/crypto/payments)
- Query `session_memory` for past work on relevant files
- Check git state — if uncommitted changes exist, stash or commit them first

### Step 2 — Survey & Pushback

- Search the codebase for existing patterns that solve or relate to the request
- If the approach is wrong, introduces tech debt, or duplicates existing code → raise a **Fundu pushback** and stop for confirmation
- Pushback format:
  ```
  ⚠️ Fundu pushback: [clear description of the concern]
  Recommend: [alternative approach]
  ```

### Step 3 — Baseline Snapshot

Before touching any code, record the project's current state into the SQL ledger:

```sql
INSERT INTO fundu_checks (task_id, phase, check_name, passed, output)
VALUES ('task-id', 'baseline', 'build', 1, 'exit 0');
```

Captures: build status, test results, lint/diagnostic errors.

**If you have zero baseline rows, you skipped this step. Go back.**

### Step 4 — Implement

- Make the minimal change that satisfies the request
- Reuse existing patterns — don't invent new abstractions if one already exists
- One logical change at a time; no scope creep

### Step 5 — The Forge

After implementing, run verification in order:

1. **Build** — must pass (exit 0)
2. **Tests** — must not regress (new failures = blocking)
3. **Lint/diagnostics** — zero new errors
4. **Completeness check** — see Anti-Laziness section below (REQUIRED for all Medium/Large tasks)
5. **Adversarial review** (Medium/Large only)

All results are `INSERT`ed into `fundu_checks`.

### Step 6 — Evidence Bundle & Commit

The evidence bundle is a `SELECT` from `fundu_checks` — not prose, not claims.

```
🔨 Fundu Evidence Bundle
task: <task-id> · size: M · risk: 🟡

Phase     | Check              | Result | Detail
----------|--------------------|--------|------------------
baseline  | build              | ✓      | exit 0
baseline  | tests              | ✓      | 247 passed
after     | build              | ✓      | exit 0
after     | tests              | ✓      | 249 passed (+2)
after     | diagnostics        | ✓      | 0 errors
after     | completeness       | ✓      | all 5 files verified
review    | gpt-5.3-codex      | ✓      | No issues
review    | gemini-3-pro       | ✓      | No issues

Regressions: None
Confidence: High
Rollback: git revert HEAD
```

---

## Anti-Laziness: Completeness Check (REQUIRED)

**The most common failure mode in large tasks is declaring completion after changing only some of the required files.**

### The Problem

When a task requires changes across multiple files (e.g., rename a variable in 8 files, update an API across 5 routes, apply a pattern to all components), a model may:
- Change 2–3 files and say "Done"
- Update some occurrences but miss others
- Forget entire files that were mentioned in the plan

### The Completeness Check Protocol

For **every Medium and Large task**, after implementing, perform a completeness check **using the same model that did the implementation** (not delegated, not assumed):

#### Step 1 — Build the expected file list upfront

Before writing any code, list every file that needs to change:

```sql
INSERT INTO fundu_checks (task_id, phase, check_name, passed, output)
VALUES ('task-id', 'plan', 'files-to-change', 1, 
  'src/auth.ts, src/routes/login.ts, src/routes/logout.ts, tests/auth.test.ts, README.md');
```

#### Step 2 — Verify each file was actually changed

After implementing, run `git diff --name-only HEAD` (or check `git status`) and cross-reference against the plan:

```bash
git diff --name-only HEAD
```

For each file in the plan, confirm it appears in the diff. If a file is missing:
- Either it genuinely didn't need to change (document why)
- Or it was missed — fix it before proceeding

#### Step 3 — Search for missed occurrences

For rename/replace tasks, use grep/ripgrep to verify no old references remain:

```bash
# Check no old symbol name remains
grep -r "oldFunctionName" . --include="*.ts" --include="*.tsx" | grep -v node_modules
```

Zero results = complete. Any results = incomplete.

#### Step 4 — Record the completeness verdict

```sql
INSERT INTO fundu_checks (task_id, phase, check_name, passed, output)
VALUES ('task-id', 'after', 'completeness', 1, 
  'All 5 planned files changed. grep confirms 0 remaining old references.');
```

### Completeness Check Examples

**Rename a function across codebase:**
```bash
# After renaming, verify zero old references remain
grep -r "calculateTotal" . --include="*.ts" | grep -v ".d.ts" | grep -v node_modules
# Expected: 0 results
```

**Update API endpoint in all callers:**
```bash
# Find all files that called the old endpoint
grep -r "/api/v1/users" . --include="*.ts" --include="*.tsx" | grep -v node_modules
# Expected: 0 results (all updated to /api/v2/users)
```

**Apply pattern to all components:**
```bash
# Verify all components have the new pattern
grep -rL "useErrorBoundary" src/components/ --include="*.tsx"
# Expected: 0 results (L = files that do NOT contain the pattern)
```

### Hard Rules

1. **Never declare a task complete without running the completeness check** for Medium/Large tasks
2. **The same model that implemented must verify** — do not delegate this to a subagent or assume it's fine
3. **Show the grep/diff output in the evidence bundle** — not just "checked, looks good"
4. **If any file is missing from the diff, fix it** — no exceptions, no "I'll do it next time"

---

## The SQL Ledger

Fundu uses a SQLite table to track all verification evidence:

```sql
CREATE TABLE IF NOT EXISTS fundu_checks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id TEXT NOT NULL,
  phase TEXT NOT NULL,         -- 'plan', 'baseline', 'after', 'review'
  check_name TEXT NOT NULL,    -- 'build', 'tests', 'lint', 'completeness', reviewer model
  passed INTEGER NOT NULL,     -- 1 = pass, 0 = fail
  output TEXT,                 -- command output or reviewer verdict
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**Why SQL?** Prose can be hallucinated. A missing `INSERT` proves the check didn't happen. The bundle is a query result, not a claim.

## Task Sizing Guide

| Size | Definition | Reviewers | Completeness Check |
|------|-----------|-----------|-------------------|
| Tiny | Read-only, docs, config comment | None | No |
| Small | Single file, low risk, clear scope | None | No |
| Medium | Bug fix, feature addition, refactor | 1 | **Yes** |
| Large | Multi-file, new architecture, auth/crypto/payments | 3 | **Yes** |

## Red-Flagged File Types (Always Large)

Any task touching these is auto-promoted to Large:
- Authentication / authorization code
- Cryptography / secrets handling
- Payment processing
- Database migrations
- CI/CD pipeline configs

## Bundled Skills

Fundu ships with 18 skills that activate automatically based on context:

| Skill | Triggers when... |
|-------|-----------------|
| `find-skills` | User asks about discovering new skills |
| `agent-browser` | User needs browser automation or web testing |
| `systematic-debugging` | A bug or unexpected behavior is encountered |
| `brainstorming` | User wants to design a feature before building |
| `writing-plans` | User has requirements and needs an implementation plan |
| `pdf` | Any PDF file is mentioned |
| `pptx` | Any .pptx / presentation is mentioned |
| `docx` | Any Word document is mentioned |
| `xlsx` | Any spreadsheet file is mentioned |
| `frontend-design` | User wants to build a UI component or page |
| `next-best-practices` | Writing or reviewing Next.js code |
| `vercel-react-best-practices` | React/Next.js performance optimization |
| `vercel-react-native-skills` | React Native or Expo development |
| `ui-ux-pro-max` | UI/UX design, color palettes, accessibility |
| `skill-creator` | User wants to create or improve a skill |
| `remotion-best-practices` | Remotion / React video development |
| `supabase-postgres-best-practices` | Postgres queries, schema, Supabase |
| `fundu-workflow` | User asks how Fundu works |
