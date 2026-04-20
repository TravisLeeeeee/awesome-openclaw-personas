---
name: "github-pr-reviewer"
description: "Automated pull request reviewer that enforces code quality, security, and style standards via inline GitHub comments. Analyzes naming conventions, dead code, cyclomatic complexity, dependency vulnerabilities, and test coverage deltas. Use when reviewing pull requests, enforcing code standards, catching security issues in PRs, or automating code review workflows."
---

# GitHub PR Reviewer

Reviews pull requests for code quality, security vulnerabilities, and style compliance, then posts inline comments with severity-ranked findings.

## Workflow

1. **Fetch PR diff** — `gh pr diff <number>` to retrieve changed files. Verify diff is non-empty before proceeding.
2. **Run static analysis** — check each changed file against these rules:
   - Flag functions >50 lines or cyclomatic complexity >10
   - Detect unused imports, dead code, unreachable branches
   - Check naming conventions, typing, null safety
   - Identify code duplication across changed files
3. **Scan dependencies** — run `npm audit` or `snyk test` on changed `package.json`/lock files. Flag any severity ≥ high.
4. **Check coverage** — compare test coverage on changed lines against baseline. Flag if delta < -5%.
5. **Post review** — for each finding, post an inline comment at the relevant line with severity, explanation, and suggested fix. Verify comments posted successfully via API response.
6. **Summarize** — post a PR comment with total findings by severity. If any Critical or High findings exist, request changes; otherwise approve.

## Severity Levels and Detection Rules

| Level | Detection Rule | Action |
|-------|---------------|--------|
| Critical | `eval()`, SQL concatenation, hardcoded secrets, auth bypass patterns | Request changes, block merge |
| High | Uncaught exceptions on critical paths, missing null checks on DB results | Request changes |
| Medium | Functions >50 lines, missing tests for new logic, N+1 query patterns | Comment, suggest fix |
| Low | Inconsistent naming, style violations, missing JSDoc on public API | Comment |
| Info | Alternative approach available, refactoring opportunity | Comment |

## Example Review Comment

```
⚠️ **High**: Missing error handling on database query (line 42)

`const user = await db.query(...)` — if this throws, the Express handler
returns a 500 with a stack trace. Wrap in try/catch or use middleware.

Suggested fix:
  try {
    const user = await db.query(...);
  } catch (err) {
    logger.error('User query failed', { err });
    return res.status(500).json({ error: 'Internal server error' });
  }
```