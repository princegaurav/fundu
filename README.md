# Fundu

> An evidence-first coding agent for GitHub Copilot CLI that verifies every change before you see it.

```bash
copilot plugin install princegaurav/fundu
```

---

## What is Fundu?

Fundu is a [GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli/about-github-copilot-in-the-cli) plugin that runs every coding task through a strict verification loop before presenting results. It builds, tests, lints, checks completeness, and has multiple AI models adversarially review the code — all before you see a single line.

**You get proof, not promises.**

---

## The Problem

AI agents write code and say "Done!" They claim builds pass without running them. They hallucinate test results. They forget to update half the files in a multi-file task. And when something breaks in production, that's your problem.

Fundu is built to fix this.

---

## Features

### 🔨 The Fundu Loop

Every Medium/Large task runs through:

1. **Understand & Recall** — parses request, checks session history for past work on the same files
2. **Survey & Pushback** — finds existing patterns to reuse; pushes back on bad ideas *before* writing code
3. **Baseline Snapshot** — records build/test/lint state into SQL *before* touching anything
4. **Implement** — minimal, surgical changes following existing codebase patterns
5. **The Forge** — build → tests → lint → completeness check → adversarial review by up to 3 AI models
6. **Evidence Bundle** — structured report generated from SQL, not written prose
7. **Auto-commit** — clean commit with rollback command shown

### ✅ Completeness Check (Anti-Laziness)

The most common AI failure: changing 2 out of 8 files and declaring done. Fundu guards against this:

- Cross-references `git diff` against the planned file list before declaring complete
- Runs `grep` to confirm zero old references remain after renames
- The same model that implemented must verify — no delegation
- Result is recorded in the SQL ledger; missing = incomplete

### ⚔️ Adversarial Review

All reviewer agents are launched in parallel. **No fixes are implemented until ALL verdicts are in.** This prevents churn from acting on partial information. Findings are then deduplicated and implemented in a single pass.

- **Medium tasks**: 1 reviewer (GPT-5.3-Codex)
- **Large tasks / red-flagged files**: 3 reviewers in parallel (GPT, Gemini, Claude Opus)

### 📊 SQL Evidence Ledger

Every build result, test run, lint check, completeness verification, and reviewer verdict is `INSERT`ed into a SQLite table. The Evidence Bundle at the end is a `SELECT` — not a claim.

```
🔨 Fundu Evidence Bundle
task: fix-auth-redirect · size: M · risk: 🟡

Phase     | Check          | Result | Detail
----------|----------------|--------|------------------
baseline  | build          | ✓      | exit 0
baseline  | tests          | ✓      | 247 passed
after     | build          | ✓      | exit 0
after     | tests          | ✓      | 249 passed (+2)
after     | completeness   | ✓      | all 4 files verified
after     | diagnostics    | ✓      | 0 errors
review    | gpt-5.3-codex  | ✓      | No issues

Regressions: None · Confidence: High · Rollback: git revert HEAD
```

### 📦 18 Bundled Skills

Fundu ships with 18 curated skills from [skills.sh](https://skills.sh/) that activate automatically:

| Category | Skills |
|----------|--------|
| **Meta** | `find-skills`, `skill-creator`, `fundu-workflow` |
| **Workflow** | `systematic-debugging`, `brainstorming`, `writing-plans` |
| **Frontend** | `frontend-design`, `ui-ux-pro-max`, `next-best-practices` |
| **React** | `vercel-react-best-practices`, `vercel-react-native-skills` |
| **Documents** | `pdf`, `pptx`, `docx`, `xlsx` |
| **Backend** | `supabase-postgres-best-practices` |
| **Tools** | `agent-browser`, `remotion-best-practices` |

---

## Install

```bash
copilot plugin install princegaurav/fundu
```

Then open GitHub Copilot CLI and select the **Fundu** agent.

---

## How It Works

### Task Sizes

| Size | Definition | Completeness Check | Adversarial Reviewers |
|------|-----------|--------------------|-----------------------|
| Tiny | Read-only, docs | No | None |
| Small | Single file, low risk | No | None |
| Medium | Bug fix, feature, refactor | ✅ Yes | 1 |
| Large | Multi-file, new architecture | ✅ Yes | 3 |

Files touching auth, crypto, payments, migrations, or CI/CD are automatically promoted to **Large**.

### Session Memory

Fundu remembers build commands, test commands, and past failures across conversations — so it never asks the same question twice.

### Pushback

Before writing code, Fundu evaluates whether the request is a good idea at both the implementation and requirements level. Disagreements surface as:

```
⚠️ Fundu pushback: [concern]
Recommend: [alternative]
```

You can override. But you'll know.

---

## Credits

Fundu is built on top of [**Anvil**](https://github.com/burkeholland/anvil) by [Burke Holland](https://postrboard.com) — the original evidence-first coding agent for GitHub Copilot CLI. Fundu extends Anvil with:

- 18 bundled skills from [skills.sh](https://skills.sh/) (replacing the Context7 MCP dependency)
- A mandatory completeness check that catches partial multi-file implementations
- A parallel-first adversarial review rule (all verdicts before any fixes)
- The `fundu-workflow` self-documenting skill
- Supabase/Postgres, Remotion, and skill-creator skill additions

The core verification loop, SQL evidence ledger, adversarial review architecture, and pushback mechanism are all from Anvil. Full credit to Burke Holland for the original design.

---

## License

MIT
