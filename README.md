# adversarial-review-skill

A Claude Code skill that conducts a two-phase adversarial security and code quality review, then automatically fixes the issues it finds.

## Installation

```bash
npx skills install jtljrdn/adversarial-review-skill
```

## Usage

```
/adversarial-review [feature-or-path]
```

Examples:

```
/adversarial-review src/auth
/adversarial-review the payment processing flow
/adversarial-review api/routes/users.ts
```

## How it works

The skill orchestrates a three-phase pipeline using subagents:

### Phase 1 — Adversarial Review

A subagent acts as a hostile reviewer, discovering all files related to the target feature and auditing them across seven categories:

1. **Security** — injection, auth bypass, data leaks, SSRF, secrets in code
2. **Correctness** — logic errors, race conditions, null handling, wrong return types
3. **Edge cases** — empty/huge inputs, special characters, concurrent access, malformed data
4. **Performance** — N+1 queries, unbounded loops, missing pagination, memory leaks
5. **API contract** — missing validation, inconsistent errors, wrong HTTP status codes
6. **Type safety** — `any` types, missing null checks, unvalidated external data
7. **Data integrity** — missing transactions, partial updates, orphaned records

Each issue is reported with a severity level (CRITICAL / HIGH / MEDIUM / LOW), file location, proof of exploitability, and a suggested fix.

### Phase 2 — Fix

A second subagent receives the full report and applies targeted fixes:

- All **CRITICAL** and **HIGH** issues are fixed
- **MEDIUM** issues are fixed where straightforward
- **LOW** issues are skipped unless trivial
- New dependencies are flagged but not installed

### Phase 3 — Report

A consolidated report is presented with:

- Review summary by category and severity
- Fix status table (FIXED / SKIPPED with reasoning)
- Remaining action items requiring manual attention

## Skill structure

```
adversarial-review-skill/
└── skills/
    └── adversarial-review/
        └── SKILL.md
```

## Configuration

| Frontmatter field          | Value                        |
| -------------------------- | ---------------------------- |
| `name`                     | `adversarial-review`         |
| `disable-model-invocation` | `true` (manual invoke only)  |
| `argument-hint`            | `[feature-or-path]`          |

This skill is user-invocable only — Claude will not trigger it automatically.