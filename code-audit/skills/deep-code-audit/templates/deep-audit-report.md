# {AppName} - Deep Code Audit (internal)

**Date:** {YYYY-MM-DD}
**Codebase:** {repo} @ {commit/branch}
**Stack:** {framework + versions}
**Method:** Tiered multi-agent audit ({N} agents) across {D} dimensions, findings verified against the live source of truth (DB schema, advisors{, Stripe test mode}, running app).
**Coverage gaps:** {e.g. "Stripe findings static + browser-DAST only - no test keys"; excluded dirs}

---

## Executive summary

{3-5 sentences, plain English: overall health, the headline risks, and what needs fixing first. State whether the green build hides runtime defects.}

| Severity | Count |
|---|---|
| BLOCKER (P0) | {n} |
| HIGH (P1) | {n} |
| MEDIUM (P2) | {n} |
| LOW (P3) | {n} |

By dimension: {one-line spread, e.g. "Payments 4, RLS 3, schema-drift 6, perf 2..."}

---

## Findings (severity-ranked)

| ID | Sev | Dimension | Location | One-line | Verified against |
|---|---|---|---|---|---|
| F-01 | BLOCKER | RLS/IDOR | `path:line` | {what} | get_advisors + live walk |
| ... | | | | | |

### Detail

#### F-01 - {title} [BLOCKER]
- **Location:** `file:line`
- **Evidence:** {code / repro / source-of-truth check that proves it}
- **Root cause:** {why}
- **Blast radius:** {what breaks / who is exposed}
- **Recommended fix:** {concrete}
- **Effort:** {S / M / L}
- **Verified:** {how it was confirmed against ground truth, or "static only - see Residual Risk"}

{repeat per finding; full detail for BLOCKER/HIGH, one-liners acceptable for LOW}

---

## Recurring-holes checklist (stack pack)

| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | Service-role key never client-side | PASS/FAIL/N-A | |
| 2 | RLS enabled + column-level per table | | |
| 3 | Stripe webhook signature verified | | |
| 4 | Webhook is sole writer of payment state | | |
| 5 | Entitlements read fresh server-side (not stale JWT) | | |
| 6 | Secrets correctly scoped (no public-prefixed secrets) | | |
| 7 | Rate limiting on auth + paid-API routes | | |

---

## What's solid

{Always include - balance earns trust. The parts that are well-built and should NOT be touched.}

---

## Residual risk & coverage gaps

- {Tools unavailable, e.g. no Stripe test keys -> payment findings unverified dynamically}
- {Excluded dirs treated as context only}
- {Anything a finder flagged but a verifier could not confirm against ground truth}

---

## Triage

| ID | Sev | Fix now (PR) | Backlog | Notes |
|---|---|---|---|---|
| F-01 | BLOCKER | x | | safe/mechanical |
| ... | | | | |

> Report first. Fixes are a separate, triaged step - each goes through `/review` -> commit -> PR. The audit does not auto-fix.
