---
name: "deploy-guardian"
description: "CI/CD deployment watchdog that monitors pipeline status, enforces deployment policies, calculates DORA metrics, and provides rollback decision support. Parses pipeline logs, runs canary analysis, and tracks post-deploy anomalies. Use when monitoring deployments, diagnosing build failures, evaluating rollback decisions, or enforcing deployment freeze windows."
---

# Deploy Guardian

Monitors CI/CD pipelines across environments, reports failures with root-cause context, enforces deployment policies, and provides data-driven rollback recommendations.

## Workflow

1. **Monitor pipelines** — poll CI/CD status across staging and production. Log each deploy event with service, environment, status, timestamp, and author.
2. **Detect failures** — parse pipeline logs to extract root cause. Identify the failing step, error message, and responsible commit. Notify the author immediately.
3. **Enforce policies** — before allowing deploys, verify:
   - Not in a freeze window (check schedule)
   - Required approval gates are satisfied
   - Canary thresholds are met (error rate <2x baseline, p99 latency <1.5x baseline)
4. **Calculate DORA metrics** — compute from deploy history:
   - Deployment frequency: deploys per day/week
   - Lead time: commit timestamp → production deploy timestamp
   - MTTR: incident start → service restored
   - Change failure rate: failed deploys / total deploys
5. **Assess post-deploy health** — compare against baseline for 2 hours post-deploy. If any threshold breached, escalate:
   - Error rate: rollback if >2x baseline
   - P99 latency: rollback if >1.5x baseline
   - CPU/memory: alert if >1.3x baseline
6. **Recommend action** — rollback, hold, or proceed. If rollback recommended, verify rollback succeeded by re-checking metrics within 5 minutes.

## Rollback Decision Matrix

| Metric | Normal | Watch | Rollback |
|--------|--------|-------|----------|
| Error rate vs baseline | <1.2x | 1.2–2x | >2x |
| P99 latency vs baseline | <1.2x | 1.2–1.5x | >1.5x |
| CPU usage vs baseline | <1.2x | 1.2–1.3x | >1.3x |

## Example: Deploy Status Report

```
Deploy Status — 2026-02-22

| # | Service    | Env     | Status            | Time  | Author |
|---|------------|---------|-------------------|-------|--------|
| 1 | user-api   | prod    | Deployed          | 09:14 | @sarah |
| 2 | payments   | prod    | Failed            | 11:45 | @alex  |

Failure #2: Step=Database migration, Error="relation already exists"
Commit: a3f91bc — "Add subscription tier columns"
Action: Fix migration or mark idempotent. @alex notified.
```