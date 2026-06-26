# Audit dimensions - what to hunt

A stack-agnostic CORE set every audit runs, plus stack-specific PACKS added by detection. Each dimension maps to a finder agent (big dimensions shard into 2-3). Findings are tagged P0-P3 per `severity-map.md`. `light` runs a thin subset (C1/C3/C8 + the primary stack pack); `full` and `deep` run the whole matrix.

## CORE dimensions (every codebase)

| # | Dimension | Hunts for | Verify against |
|---|---|---|---|
| C1 | **Security / authz** | injection, IDOR / missing ownership checks, broken auth, exposed secrets, input validation at boundaries, CORS | live walk + DB advisors |
| C2 | **Data integrity** | schema drift (code columns vs real schema), stale generated types, migration consistency, nullable/constraint mismatches | `information_schema` |
| C3 | **Correctness / logic** | state machines, off-by-one, timezone/locale, money math, edge cases (empty/sold-out/expired) | live walk |
| C4 | **Reliability / errors** | swallowed errors, missing try/catch + timeouts on external calls, wrong HTTP status semantics, unhandled rejections | static + logs |
| C5 | **Performance** | N+1, missing indexes, serial awaits, render/RSC waterfalls, bundle bloat | `pg_stat_statements` |
| C6 | **Supply chain / secrets** | dep CVEs (SCA), secrets in source + git history, public-prefixed secrets, dead deps | OSV + grep |
| C7 | **CI/CD / infra** | `pull_request_target`, cache poisoning, Actions secret exposure, env scoping, deploy config | static |
| C8 | **Code quality / dead code** | `any`/`@ts-ignore`, dead branches, duplicate logic, magic values, console logging in prod | static |
| C9 | **AI-codegen footguns** (2026) | XSS (CWE-80) + log/output injection (CWE-117) - the two classes AI is worst at; cross-file dataflow holes (taint in route A, sink in service B); "feature implemented, role/ownership guard forgotten" | live walk + dataflow trace |

> **C9 runs on EVERY audit of an AI-built app** (assume that's most of them - see `methodology.md`). It is a fixed reviewer scope on top of the stack pack. Finders for C9 MUST trace dataflow *across files*, not scan one file at a time - that is the exact gap the models leave.

## Stack PACK: Next.js + Supabase + Stripe (canonical)

Add these on top of CORE when the stack is detected. This is the pack we built for Ora.

| # | Dimension | Hunts for | Verify against |
|---|---|---|---|
| P1 | **Payments / Stripe** | PaymentIntent lifecycle, idempotency keys, webhook = SOLE writer of payment state, money math (cents/fees/refunds/Connect payouts), double-charge / duplicate-order races | Stripe TEST mode only if keys present; else static + browser |
| P2 | **Supabase RLS / IDOR** | RLS enabled per table, COLUMN-level policies (not just row), ownership checks across every API route + server action, service-role client staying server-only | `get_advisors` + `information_schema` |
| P3 | **Auth / session** | stale-JWT entitlement spoofing (read fresh from DB server-side), post-signup session race, middleware coverage of protected routes | live walk |

Mobile/Capacitor folds into C1-C3 as a platform sub-check (deep links, native bridge, platform detection) unless the native surface is large enough to warrant its own finder.

## Mandatory recurring-holes checklist + AI-codegen footgun pass

These live in `recurring-holes.md` now (the stack-specific checklist, the
AI-codegen footgun pass, and the 2026 supply-chain hardening list). Run that
checklist as PASS / FAIL / N-A on every audit whose stack matches - never
silently omit an item.

## Boundary note
This dimension set is deeper than the generic 10 in `dimensions-scorecard.md`
(which feeds the client-facing `full-code-audit` DOCX scorecard). Use that 10-dim
scorecard for the for-sale grading; use THIS matrix to drive the finder fan-out.
