---
name: deep-code-audit
description: Deep internal engineering audit of a LARGE, MATURE codebase WE OWN and maintain (post-takeover or pre-hardening), run as a tiered multi-agent workflow. Hunts security, payment-correctness, schema-drift, logic, reliability, performance, and supply-chain defects across the whole repo, verifies each finding against the live source of truth (DB schema, advisors, running app), and ships a severity-ranked engineering findings report that feeds triage and PRs. Use when the user says "deep code audit", "full audit on a mature codebase", "audit the codebase we maintain", "harden this app", "no-holds-barred audit", or "/deep-code-audit". NOT for: a client-facing for-sale viability deliverable (use /full-code-audit), a fast top-risks triage (use /light-code-review), or our own pre-ship diffs (use /review or code-review).
---

# /deep-code-audit - Deep internal audit of a mature codebase we maintain

Brings exhaustive, cost-disciplined multi-agent rigor to a codebase **we already own**. The goal is to find and rank every real defect so we can triage and fix, not to produce a sales document and not to decide whether to take the code over (that decision already happened).

## Boundary - pick the right tool

| Skill | Audience | Output | When |
|---|---|---|---|
| **deep-code-audit** (this) | us, the maintaining team | severity-ranked engineering findings report -> triage -> PRs | a large codebase we OWN; "find everything wrong" |
| `full-code-audit` | the client / a buyer | branded UNPRICED DOCX viability verdict | pre-takeover, for-sale decision deliverable |
| `light-code-review` | us | 2-3 agent top-risks triage, ~5 min | a fast read before committing to deeper work |
| `code-review` / `/review` | us | our own staged-diff review | before we ship our own commits |

If the user wants a client deliverable or a takeover/viability verdict, STOP and use `full-code-audit`.

## Usage

```
/deep-code-audit <path>                 # audit the mature codebase at <path>
/deep-code-audit <path> <project-name>   # resolve output + Obsidian to that project
```

If `<path>` is omitted, confirm cwd. The codebase must already be cloned locally.

## Read first

- `../../shared/methodology.md` - the four-lens model (SAST / SCA / DAST / secrets) + the verify-against-source-of-truth principle. This is the "why".
- `../../shared/dimensions.md` - the audit dimension matrix: a stack-agnostic CORE set (incl. C9 AI-codegen footguns) + stack-specific packs. This is the "what to hunt".
- `../../shared/recurring-holes.md` - the mandatory PASS/FAIL/N-A checklist + the 2026 AI-codegen footgun pass + supply-chain hardening list. This is the "known killers to confirm".
- `../../shared/workflow-skeleton.md` - the tiered multi-agent Workflow recipe (find -> adversarially verify -> synthesize) with the model-tiering + agent-count rules. This is the "how to run it".
- `../../shared/severity-map.md` - P0-P3 vocabulary + A-F grading (shared with the other audit skills).
- `templates/deep-audit-report.md` - the findings-report deliverable structure.

## Workflow

### Step 1 - Scope and shape
Resolve the path. Capture header metrics so the fan-out is sized to reality, not a template:
```bash
git ls-files | wc -l                                  # total tracked
git ls-files '*.ts' '*.tsx' | xargs wc -l | tail -1   # LOC of the primary language
git ls-files 'src/app/api/**/route.ts' | wc -l        # API surface
git grep -l "use server" | wc -l                      # server-action surface
git ls-files 'supabase/migrations/*.sql' | wc -l      # schema surface
```
Detect the stack (Stage 1 of `../../shared/engine.md`) and select the dimension packs from `../../shared/dimensions.md`. **Exclude non-shipping dirs** from the deep read (docs, specs, plans, analysis dumps) - treat them as context, not audit targets. Announce the excluded dirs.

### Step 2 - Build/health baseline
Run the stack's typecheck/build (`tsc --noEmit`, `pnpm build`, `expo export`, `flutter analyze`). A green build is a BASELINE, not proof - this skill explicitly assumes green-build-but-broken-runtime is possible (see methodology). Record build status as dimension evidence.

### Step 3 - Size the run and GET EXPLICIT GO (mandatory gate)
Deep audits fan out wide and the Workflow tool can spend millions of tokens fast (real incident 2026-06-01: 33 agents / 2.55M tokens). Before launching:
- Compute the real fan-out from `../../shared/workflow-skeleton.md`: N finder/sharder agents + a CAPPED verify layer + 1 synthesis. **The verify layer is `finders + (P0/P1 count x refuters)` and the P0/P1 count is UNKNOWN until finders return - on a legacy codebase it dominates.** Announce a RANGE, not a point estimate (the 2026-06-06 Ora run was 88 agents / ~5M tokens against a ~35 point-estimate - a 2.5x overrun). For a ~250K-LOC legacy app say "30-90 agents depending on blocker count" and damp the verify term (P0-only, or batch verifiers) if you expect a rich yield.
- **TIER MODELS**: finders + verifiers run `model:'sonnet'`; ONLY the final synthesis runs Opus (omit model). The pre-tool-workflow-budget hook BLOCKS an all-Opus fan-out.
- **ANNOUNCE the projected agent count to the user and wait for an explicit go.** This is non-negotiable - never auto-launch a deep audit.

### Step 4 - Run the audit workflow
Build the script from `../../shared/workflow-skeleton.md`: **pipeline by dimension** (each dimension finds, then EACH finding is adversarially verified the moment that dimension completes - no barrier), tiered models, capped verify. Findings carry `file:line` + a reproduction or a source-of-truth check. Reviewers are READ-ONLY; the audit never edits code.

### Step 5 - Verify findings against the source of truth (the differentiator)
A green build lies; trust the running system. For each surviving finding, confirm against ground truth where possible (NOT against tsc):
- **Schema drift / DB**: check referenced columns against live `information_schema.columns` (Supabase MCP `execute_sql`); run `get_advisors` for RLS/security lints.
- **Performance**: `pg_stat_statements` for real slow queries, not guessed ones.
- **Payments**: Stripe **test mode** only if test keys are available; otherwise keep payment findings static + DAST-via-browser and SAY SO.
- **Runtime behavior**: a live browser walk (Chrome MCP) for the flows the findings touch.
Drop any finding that the source of truth refutes. Note tools that were unavailable (no Stripe keys, etc.) under Residual Risk - never imply coverage you did not have.

### Step 6 - Synthesize the report
One Opus synthesis agent dedupes, severity-orders (BLOCKER->HIGH->MEDIUM->LOW), and writes the report per `templates/deep-audit-report.md` to `<repo>/audit/YYYY-MM-DD-deep-audit.md`:
- Executive summary + finding counts by severity and dimension
- Severity-ranked findings table (id, dimension, `file:line`, severity, one-line, verified-against)
- Per-finding detail: evidence, root cause, blast radius, recommended fix, effort
- A **recurring-holes checklist** result (the stack pack's mandatory items, each PASS/FAIL/N-A)
- What's solid (balance earns trust)
- Residual risk + coverage gaps (unavailable tools, excluded dirs)
- A triage table: which findings become PRs now vs backlog
Mirror a dated entry to Obsidian `Projects/<name>/Progress Log` if it is a tracked project.

### Step 7 - Triage, do NOT auto-fix
Present the report. **Report first; fixing is a separate, triaged step.** Propose which findings become PRs (safe/mechanical first: schema-drift renames, idempotency keys, missing webhook-sig checks). Do not open fix PRs during the audit run - the user triages. Each fix later goes through the normal `/review` -> commit -> PR gate.

## Cost / runtime
Heavier than every other review tier by design. ~25-40 min wall clock + a real token spend; that is why Step 3's announce-and-confirm gate exists. The payoff is exhaustive, source-of-truth-verified coverage of a codebase too large for one context.

## Common mistakes
| Mistake | Fix |
|---------|-----|
| Auto-launching the workflow | Step 3 gate: announce agent count, get explicit go |
| All-Opus fan-out | Tier: sonnet finders/verifiers, Opus only for synthesis (budget hook blocks otherwise) |
| Uncapped per-finding verify layer | Cap verify to HIGH/MEDIUM; LOW findings skip the verify fan-out |
| Trusting the green build as proof | Verify against live schema/advisors/running app (Step 5) |
| Claiming payment coverage without Stripe keys | State the gap in Residual Risk; keep payment findings static + browser-DAST |
| Auditing docs/specs/analysis dirs | Exclude non-shipping dirs; they are context only |
| Reviewers editing code | Read-only; fixes are a separate triaged PR step |
| Inventing findings in synthesis | Synthesis stays in lane - no finding a reviewer did not flag and a verifier did not confirm |
| Producing a branded client DOCX | Wrong tool - that is `full-code-audit` |
