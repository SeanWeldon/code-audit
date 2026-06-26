# {AppName} - Light Code Review

**Prepared by:** Bolder Apps
**Date:** {YYYY-MM-DD}
**Codebase reviewed:** {repo URL or zip filename}
**Stack:** {framework + versions}
**Method:** Fast multi-agent triage (security + primary stack + robustness). Not a comprehensive audit.

---

## Verdict

> **{TAKEOVER VIABLE | VIABLE - STABILIZATION REQUIRED | TAKEOVER NOT VIABLE / READY / NOT READY TO SHIP}**

{1-2 sentences: the headline read and what it means for the client in plain English.}

---

## Top Findings

The highest-signal issues surfaced in a fast pass. Severity reflects practical risk.

| Severity | Area | Finding | Evidence |
|----------|------|---------|----------|
| {BLOCKER/HIGH/MEDIUM/LOW} | {area} | {one-line description} | `path/to/file.ts:42` |
| … | … | … | … |

{5-8 rows max. A light review surfaces the top risks, not an exhaustive list.}

---

## What's Working Well

- {positive observation}
- {positive observation}

---

## Recommended Next Step

{ONE clear recommendation. Examples:
- "Commission a full code audit - the security and data-layer findings warrant a comprehensive review before takeover."
- "Fixes are well-scoped and low-risk; proceed to a stabilization sprint."
- "Findings are minor; the codebase is in good shape for takeover."}

---

*This is a light review - a fast triage of the highest-risk areas. A full code audit (multi-agent + cross-model verification, 10-dimension scorecard) is available as a follow-on for a comprehensive picture and a priced remediation plan.*
