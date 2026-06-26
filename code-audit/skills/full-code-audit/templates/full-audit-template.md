# {AppName} - Code Audit

**Prepared by:** Bolder Apps
**Date:** {YYYY-MM-DD}
**Codebase reviewed:** {repo URL or zip filename}
**Stack:** {framework + versions, e.g. "Expo Router 4.0 / React Native 0.74 / Supabase"}
**Method:** Multi-agent review across 10 dimensions + independent Codex cross-model verification

---

## Verdict

> **{TAKEOVER VIABLE | VIABLE - STABILIZATION REQUIRED | TAKEOVER NOT VIABLE}**

{One paragraph, 3-4 sentences. Plain English. State the verdict, the headline reason, and the practical implication for the client. Avoid jargon.}

**By the numbers:**

| Metric | Value |
|--------|-------|
| Lines of code | {LOC} |
| Dependencies | {count} |
| Test files | {count} |
| Last commit | {date} |
| BLOCKER findings | {n} |
| HIGH findings | {n} |
| MEDIUM findings | {n} |

---

## Code Review Scorecard

Each of the 10 dimensions graded A-F with a risk level. Grades follow the rubric in `../../../shared/severity-map.md`. Security and Build Health are weighted heaviest.

| Dimension | Grade | Risk | Evidence |
|-----------|-------|------|----------|
| Build Health | {A-F} | {Low/Moderate/High/Critical} | `path:line` or note |
| Architecture Quality | {A-F} | {risk} | … |
| Backend / Database | {A-F} | {risk} | … |
| Navigation & Routing | {A-F} | {risk} | … |
| Feature Completeness | {A-F} | {risk} | … |
| Code Quality | {A-F} | {risk} | … |
| Security | {A-F} | {risk} | … |
| Missing Infrastructure | {A-F} | {risk} | … |
| Dependency Bloat | {A-F} | {risk} | … |
| Critical Bugs | {A-F} | {risk} | … |
| **Overall** | **{A-F}** | **{risk}** | worst-weighted; F in Security or Build Health caps at D |

---

## Findings Summary

Findings are severity-tiered. Severity reflects practical risk - not an academic score.

| Severity | Area | Finding | Evidence |
|----------|------|---------|----------|
| BLOCKER | {dimension} | {one-line description} | `path/to/file.ts:42` |
| HIGH | … | … | … |
| MEDIUM | … | … | … |
| LOW | … | … | … |

**Severity definitions:**

- **BLOCKER** - Ship/takeover-stopping. Cannot ship features until resolved.
- **HIGH** - Phase 0 mandatory. Must address before any feature work lands.
- **MEDIUM** - Phase 1. Existing feature works but is broken/incomplete.
- **LOW** - Cleanup / hardening. Can defer to Phase 3.

---

## Critical Findings (Blockers)

{For each BLOCKER finding, write a short narrative paragraph: what it is, why it stops takeover/ship, the specific files, and what fixing it looks like. Be concrete - the client should walk away knowing exactly what's wrong.}

### Finding 1: {short title}

{Narrative, 3-5 sentences. End with file/path references.}

### Finding 2: {short title}

{…}

---

## What's Working Well

{2-5 bullets of genuine positives. The audit must be balanced to be trusted - if there are no positives, say so, but look hard first.}

- {positive observation}
- {positive observation}

---

## Recommended Path

{Phased remediation by focus area. This audit is UNPRICED - hours are produced separately by the estimation step. List the phases and what each addresses; leave Hours/Weeks columns as placeholders or omit them.}

| Phase | Focus | Hours | Weeks |
|-------|-------|-------|-------|
| Phase 0 | Security & Stabilization - {BLOCKER/HIGH items} | TBD | TBD |
| Phase 1 | Core Workflow Fixes - {MEDIUM items} | TBD | TBD |
| Phase 2 | Spec Gap Features - {missing modules} | TBD | TBD |
| Phase 3 | Production Readiness - {infra/cleanup} | TBD | TBD |

> **Pricing handoff:** Run the `code-takeover-estimation` skill against this audit to produce the priced, task-level phased estimation CSV + DOCX. This audit is the review input; estimation is the single source of pricing truth.

---

## Assumptions and Dependencies

What we need from the client before remediation begins:

- {credential / access need}
- {missing documentation}
- {decisions the client must make}

---

## Engagement Options

- **Stabilization Sprint** - Phase 0 only, fixed scope
- **Full Takeover** - Phases 0-3
- **Hybrid** - Phase 0 stabilization + ongoing feature work T&M

---

## Residual Risk

{What was NOT verified: untested platform behavior, missing reviewer scopes, Codex cross-check status (ran / unavailable), areas needing a deeper follow-on engagement.}
