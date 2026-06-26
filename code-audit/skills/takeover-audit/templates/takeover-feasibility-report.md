# {AppName} - Takeover Feasibility Audit

**Prepared by:** Bolder Apps
**Date:** {YYYY-MM-DD}
**Codebase:** {repo} @ {commit/branch}
**Stack:** {framework + versions}
**Method:** Tiered multi-agent audit ({N} agents): full security fan-out (SAST/SCA/DAST/secrets + C9 AI-codegen, adversarially verified) + 4 ownership finders (architecture, feature-completeness, operational-readiness, cost-to-own).
**Coverage:** {what was source-of-truth verified vs static; excluded dirs}

---

## 1. Verdict

> **{TAKEOVER VIABLE | VIABLE - STABILIZATION REQUIRED | TAKEOVER NOT VIABLE}**

{One paragraph, plain English: the call, the headline reason, and the practical implication. State whether this is a "finish and secure" job or a rebuild. This is the one thing a decision-maker reads.}

**By the numbers:**

| Metric | Value |
|--------|-------|
| Lines of code (shipping) | {LOC} |
| Dependencies | {count} |
| Test files / critical-path coverage | {count} / {%} |
| CI/CD | {present/absent} |
| Last commit / commit count / authors | {date} / {n} / {n} |
| BLOCKER / HIGH / MEDIUM / LOW | {n} / {n} / {n} / {n} |
| Feature completeness (real vs intent) | {%} |

---

## 2. Scorecard (10 dimensions, A-F)

Security + Build Health weighted heaviest; an F in either caps Overall at D (`severity-map.md`).

| Dimension | Grade | Risk | Evidence |
|-----------|-------|------|----------|
| Build Health | {A-F} | {risk} | {note} |
| Architecture Quality (O1) | {A-F} | {risk} | {file sizes / coupling} |
| Backend / Database | {A-F} | {risk} | {migrations in repo?} |
| Navigation & Routing | {A-F} | {risk} | {note} |
| Feature Completeness (O2) | {A-F} | {risk} | {real vs stub vs missing} |
| Code Quality | {A-F} | {risk} | {note} |
| Security | {A-F} | {risk} | {blocker count} |
| Missing Infrastructure (O3) | {A-F} | {risk} | {tests/CI/monitoring} |
| Dependency Bloat (O4) | {A-F} | {risk} | {declared vs imported} |
| Critical Bugs | {A-F} | {risk} | {note} |
| **Overall** | **{A-F}** | **{risk}** | worst-weighted |

---

## 3. Security Findings (exploitability)

Severity-ranked, deduped. Full detail for BLOCKER/HIGH; one-liners acceptable for LOW. This is the deep security battery; the verdict above already accounts for it.

| ID | Sev | Dimension | Location | One-line | Verified against |
|---|---|---|---|---|---|
| F-01 | BLOCKER | {dim} | `file:line` | {what} | {source-of-truth / static / runtime} |

### Detail (BLOCKER / HIGH)
#### F-01 - {title} [BLOCKER]
- **Evidence / Root cause / Blast radius / Fix / Effort (S/M/L) / Verified** {as in the deep template}

---

## 4. Ownership Assessment (cost to own)

The four ownership reads that drive the takeover-vs-rebuild call.

### O1 - Architecture & Maintainability: {grade}
{How painful to extend/own. God-files, coupling, seams, pattern consistency. Is a change localized or does it ripple?}

### O2 - Feature Completeness: {grade} ({%} real vs intent)
{Real vs scaffolded vs stubbed vs missing, mapped to the product's stated intent. Call out the "UI-complete, backend-stubbed" pattern if present. Which flows actually work end-to-end?}

### O3 - Operational Readiness: {grade}
{Tests, CI/CD, monitoring, logging, deploy + rollback, migrations-in-repo vs black-box DB, docs. Can we safely change + ship this on day one?}

### O4 - Cost-to-Own: {grade}
{Dependency bloat, dead code, EOL/abandoned deps, build fragility/speed, asset bloat, upgrade debt, bus-factor. The ongoing carrying cost.}

---

## 5. Phased Remediation Roadmap

UNPRICED - hours produced by `code-takeover-estimation`. Effort = S/M/L; weeks = rough range.

| Phase | Focus | Closes | Effort | Weeks | Depends on |
|-------|-------|--------|--------|-------|------------|
| Phase 0 | Security & Stabilization | {BLOCKER/HIGH ids} | {S/M/L} | {range} | {access/creds} |
| Phase 1 | Core Workflow Fixes | {MEDIUM ids} | {S/M/L} | {range} | Phase 0 |
| Phase 2 | Feature Gaps (O2) | {stubbed/missing} | {S/M/L} | {range} | backend access |
| Phase 3 | Production Readiness (O3) | {tests/CI/monitoring} | {S/M/L} | {range} | Phase 0 |

> **Pricing handoff:** run `code-takeover-estimation` against this audit for the priced task-level CSV + DOCX.

---

## 6. Takeover vs Rebuild

{Include the two-path table when the verdict is borderline or NOT VIABLE; otherwise a one-line "rebuild not warranted - foundation is sound" suffices.}

| Path | Effort | Risk |
|------|--------|------|
| Takeover + remediation | {S/M/L, weeks} | {reason} |
| Greenfield rebuild | {S/M/L, weeks} | {reason} |

{If NOT VIABLE: explain which fundamental issues compound and why fixing approaches/exceeds a rebuild. Recommend rebuilding using the existing app as the spec.}

---

## 7. Strategic Advice

1. **The call:** {take over / stabilize-then-own / rebuild} - {one-line why}.
2. **Phase 0 (non-negotiable before any feature work):** {the sequenced security + stabilization set}.
3. **Sequencing / dependencies:** {what must come first - e.g. migrations into repo before schema work; secret rotation before granting team access}.
4. **Access / info to demand before signing:** {prod creds (rotate-first), test account, live DB/schema, original spec, deploy access, domain/provider ownership} - each tied to a finding that stays unverifiable without it.
5. **Hidden ownership costs:** {EOL upgrades coming due, native-build fragility, single-author bus factor - the carrying costs that are not bugs}.
6. **Effort sizing (UNPRICED):** {per-phase S/M/L + week ranges; pricing -> estimation}.

---

## 8. What's Solid

{Always include - balance earns trust. The parts that are well-built and should NOT be touched. This is also what makes a takeover attractive vs a rebuild.}

---

## 9. Residual Risk & Coverage Gaps

- {Source-of-truth tools unavailable (no DB/Stripe/live access) -> which findings stay static}
- {Excluded dirs treated as context only}
- {Anything a finder flagged that a verifier could not confirm}
- {Codex status if run; ownership reads are structural, not runtime-verified}
