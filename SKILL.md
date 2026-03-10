---
name: adversarial-review
description: Conduct an adversarial review of a feature, then fix all issues found. Use when the user wants to stress-test, audit, or harden a feature.
disable-model-invocation: true
argument-hint: "[feature-or-path]"
---

# Adversarial Review & Fix

You are orchestrating a two-phase adversarial review of: **$ARGUMENTS**

## Phase 1 — Adversarial Review

Launch an agent (subagent_type: `general-purpose`) with the following mission. **Do NOT fix anything in this phase — only identify issues.**

Give the agent this prompt:

> You are a hostile adversarial reviewer. Your job is to tear apart the feature described as "$ARGUMENTS" and find every possible issue. Be ruthless and thorough.
>
> **Step 1 — Discover scope.** Use Glob and Grep to find all files related to the feature. Read every relevant file fully.
>
> **Step 2 — Review for issues across these categories:**
>
> 1. **Security** — injection, auth bypass, data leaks, SSRF, improper validation, missing rate limiting, secrets in code
> 2. **Correctness** — logic errors, off-by-one, race conditions, null/undefined handling, missing error handling, wrong return types
> 3. **Edge cases** — empty inputs, huge inputs, special characters, concurrent access, missing fields, malformed data
> 4. **Performance** — N+1 queries, unbounded loops, missing pagination, memory leaks, expensive re-renders, missing indexes
> 5. **API contract** — missing validation, inconsistent error responses, undocumented behavior, wrong HTTP status codes
> 6. **Type safety** — `any` types, missing null checks, incorrect type assertions, unvalidated external data
> 7. **Data integrity** — missing transactions, partial updates, orphaned records, missing cascades
>
> **Step 3 — Produce a structured report.** For each issue found, include:
> - **Category** (from the list above)
> - **Severity**: CRITICAL / HIGH / MEDIUM / LOW
> - **File & line(s)**
> - **Description** of the vulnerability or bug
> - **Proof / reasoning** — explain exactly how it could be triggered or exploited
> - **Suggested fix** (brief)
>
> Sort issues by severity (CRITICAL first). Be specific — cite exact code, not vague observations. Do NOT suggest stylistic or cosmetic changes. Only report real, actionable issues.

Wait for this agent to complete and capture its full report. 

Output the full report to the user and ask for any feedback or clarification. If the user answers wanting to continue, launch Agent 2 to fix. If not, refine the review report to the feedback given and reprompt the user to continue.

## Phase 2 — Fix

Launch a second agent (subagent_type: `general-purpose`) and pass it the full report from Phase 1.

Give the agent this prompt:

> You are a senior engineer. You have received an adversarial review report for the feature "$ARGUMENTS". Your job is to fix every CRITICAL and HIGH severity issue, and fix MEDIUM issues where the fix is straightforward.
>
> LOW severity issues should be noted but skipped unless trivial to fix.
>
> **Rules:**
> - Read each affected file before editing it.
> - Make minimal, targeted fixes — do not refactor or restructure unrelated code.
> - Do not break existing functionality.
> - If a fix requires a new dependency, note it but do not install it — flag it for the user.
> - If a fix is ambiguous or risky, skip it and explain why.
>
> **After all fixes are applied, produce a summary with:**
> - A table of every issue from the report with columns: Severity | File | Issue | Status (FIXED / SKIPPED) | What was changed
> - Any issues you skipped and why
> - Any new dependencies or manual steps required

Wait for this agent to complete and capture its summary.

## Phase 3 — Report

Present the user with a final consolidated report:

1. **Review summary** — how many issues were found by category and severity
2. **Fix summary** — the table from Phase 2 showing what was fixed vs skipped
3. **Remaining action items** — anything that still needs manual attention (skipped fixes, new dependencies, architectural concerns)

Keep the report concise and scannable.
