# Severity Map & Grading Rubric (shared)

Client-facing deliverables use **BLOCKER / HIGH / MEDIUM / LOW** - the vocabulary the audit template and DOCX generator already use. Internal-style reviewers emit **P0-P3**. The synthesizer translates.

## P ↔ client severity

| Internal | Client | Meaning | Examples |
|---|---|---|---|
| P0 | **BLOCKER** | Ship/takeover-stopping | live secret in source, RLS/IDOR bypass, data-loss bug, app won't build |
| P1 | **HIGH** | Phase-0 mandatory before feature work | missing webhook sig, missing auth rate limit, broken core flow, no migrations in repo |
| P2 | **MEDIUM** | Existing feature broken/incomplete | silent error swallowing, partial feature, missing CORS on error path |
| P3 | **LOW** | Cleanup / hardening, deferrable | console logs in prod, dead deps, naming, magic values |

## Dimension grading rubric (A-F) - full audit scorecard

Grade each of the 10 dimensions (`dimensions-scorecard.md`) from the findings:

| Grade | Risk | Criteria |
|---|---|---|
| **A** | Low | Idiomatic, complete, no findings in this dimension |
| **B** | Low | Solid; only LOW findings |
| **C** | Moderate | Works but has ≥1 MEDIUM finding or notable gaps |
| **D** | High | ≥1 HIGH finding; needs Phase-0 attention before feature work |
| **F** | Critical | ≥1 BLOCKER; dimension is takeover-stopping as-is |

Overall grade = worst-weighted across dimensions, with Security and Build Health weighted heaviest (an F in either caps the overall at D).

## Readiness / verdict phrasing

- Any BLOCKER → **"TAKEOVER NOT VIABLE"** (or "NOT READY TO SHIP") - name the blocker(s) in one sentence.
- No BLOCKER, ≥1 HIGH → **"VIABLE - STABILIZATION REQUIRED"** - Phase-0 work mandatory first.
- MEDIUM/LOW only or clean → **"TAKEOVER VIABLE"** (or "READY") - proceed; defer cleanup.

These three verdict strings match `VERDICT_COLORS` in the DOCX generator - use them verbatim so the document colors render.
