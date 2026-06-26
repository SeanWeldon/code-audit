---
name: full-code-audit
description: Comprehensive client-facing code audit of a whole codebase using the multi-agent + Codex cross-model review engine, the four-lens methodology (SAST/SCA/DAST/secrets), the mandatory recurring-holes checklist, the 2026 AI-codegen footgun pass, and opportunistic source-of-truth verification. Produces a 10-dimension scorecard, severity-tiered findings, critical-findings narrative, and a branded DOCX - UNPRICED (hands pricing off to code-takeover-estimation). Use when a client requests a paid code audit / code review of an existing app, before a takeover decision, or when the user says "full code audit", "audit this codebase", "/full-code-audit". For a faster top-risks triage use /light-code-review instead; for a codebase we already own use /deep-code-audit; for OUR own pre-ship diffs use /review or the code-review skill.
---

# /full-code-audit - Comprehensive client code audit

Brings internal-grade review rigor (parallel multi-agent fanout + independent Codex cross-check + the four-lens methodology + the recurring-holes checklist) to a client-facing deliverable. Replaces the *weakest link* of `code-takeover-estimation` (its single-Explore-agent review) and feeds it: this skill produces the audit; estimation produces the price.

**Governing principle (`../../shared/methodology.md`):** a green build is a baseline, not proof. Assume the code is AI-built and audit for what the 2026 models measurably miss (XSS, log injection, cross-file dataflow, forgotten ownership/role guards). Where live access exists, verify findings against the running system; where it does not, mark them static and say so. Never imply coverage we did not have.

**Boundary:** this reviews a WHOLE external/client codebase and ships a branded client deliverable. It does NOT review our own diffs - use `/review`, `/review-staged`, or `precheck` for that. For a codebase we already own, use `/deep-code-audit` (adds the tiered Workflow + full source-of-truth verification).

## Usage

```
/full-code-audit <path>                 # audit the codebase at <path>
/full-code-audit <path> <client-name>   # also resolve output to products/clients/<client-name>/
```

If `<path>` is omitted, ask for it (or confirm cwd). Prerequisite: the codebase already exists locally (clone/extract is a manual step, same as `code-takeover-estimation`).

## Read first

- `../../shared/methodology.md` - the four lenses (SAST/SCA/DAST/secrets), trust-the-running-system, and the 2026 AI-codegen + supply-chain research. The "why".
- `../../shared/recurring-holes.md` - the mandatory PASS/FAIL/N-A checklist + the AI-codegen footgun pass + 2026 supply-chain hardening. The "known killers to confirm".
- `../../shared/dimensions.md` - the CORE + stack PACK hunt matrix (drives the finder fan-out, incl. C9 AI-codegen footguns).
- `../../shared/engine.md` - the detect → fan-out → synthesize → Codex procedure
- `../../shared/severity-map.md` - P↔client severity + A-F grading rubric
- `../../shared/dimensions-scorecard.md` - the 10 dimensions (drives the DOCX scorecard grading)
- `templates/full-audit-template.md` - the deliverable structure

## Workflow

### Step 1 - Resolve target & metrics
Confirm the codebase path, app name, framework + versions (from `package.json`/`pubspec.yaml`/etc.). Gather header metrics: LOC, dependency count, test-file count, last-commit date. Pick the output location:
- target under `products/clients/<name>/` → write to that client folder as `<AppName>-Code-Audit.md` + `.docx`
- otherwise → `<target>/audit/YYYY-MM-DD-code-audit.md` + `.docx`

Establish the run dir (`<output-dir>/.audit-run-<ts>/_partials/`) per the engine.

### Step 2 - Build/health check
Run the stack's build/typecheck: `tsc --noEmit`, `expo export`, `flutter analyze`, etc. Capture results as Build Health (dimension 1) evidence. A failing build is itself a BLOCKER. **A green build is a baseline, not proof** (`methodology.md`) - it does not clear schema drift, runtime auth holes, or money-math bugs. Detect the stack here and select the recurring-holes checklist + stack PACK for the fanout.

### Step 3 - Full parallel fanout (engine Stage 2)
Dispatch all stack-warranted reviewer scopes in a SINGLE message (parallel, read-only) so all 10 dimensions are covered. Each writes a severity-tagged partial to `_partials/`. Use the reviewer prompt shape from `.claude/agents/reviewers/security-fanout.md`.

On top of the engine's scopes, ALWAYS add:
- **An AI-codegen footgun reviewer (C9)** per `recurring-holes.md` section A: hunt XSS (CWE-80) + log/output injection (CWE-117) - the two classes 2026 models are worst at - plus "feature implemented, ownership/role guard forgotten" access-control gaps. This reviewer MUST trace dataflow ACROSS files (taint in route A, sink in service B), not scan one file at a time - that is the exact gap the models leave.
- **The recurring-holes checklist as an explicit scope**: every item in `recurring-holes.md` (the detected stack pack + supply-chain list) gets a PASS / FAIL / N-A verdict with evidence. Run `capabilities/supply_chain_guard` (age gate + OSV) and grep workflows for `pull_request_target`, shared publish caches, and dependency `pre/postinstall` scripts.

### Step 4 - Codex cross-check (engine Stage 4)
Run `codex exec review` as the independent second model. Follow the engine's rules EXACTLY: direct-redirect never pipe, `--base` only (no custom prompt), Monitor until-loop wait, verify the `^codex$` verdict, silent-exit guard. For a non-git codebase, `git init` a throwaway baseline. If Codex is unavailable/unauthenticated, record it under Residual Risk and proceed Claude-only - do NOT block the audit.

### Step 5 - Verify high-severity findings against source of truth (opportunistic)
Pre-takeover we usually lack live DB/Stripe access, so this is best-effort, not mandatory. For each BLOCKER/HIGH finding, verify against ground truth WHERE ACCESS EXISTS - never against the build:
- **Schema drift / DB**: referenced columns vs live `information_schema.columns` (Supabase MCP `execute_sql`); RLS/security lints via `get_advisors`.
- **Performance**: real slow queries from `pg_stat_statements`, not guessed.
- **Runtime / auth / XSS**: a live browser walk (Chrome MCP) of the flows the finding touches.
- **Payments**: Stripe TEST mode only if test keys are available.

Drop any finding the source of truth refutes. For every tool that was unavailable (no DB access, no Stripe keys, no running app), record the gap under Residual Risk in Step 6 - keep those findings static and labelled, never imply coverage we did not have. If NO live access exists at all, say so plainly and proceed Claude+Codex static.

### Step 6 - Synthesize (engine Stage 3 + scorecard)
Dedupe, severity-order, map P→client severity, reconcile Codex vs Claude findings. Grade all 10 dimensions A-F + risk. Write the master markdown following `templates/full-audit-template.md`:
- Verdict (use a verbatim verdict string from severity-map so DOCX colors render)
- Code Review Scorecard (10 dims)
- **Recurring-holes checklist results** - the `recurring-holes.md` items as a PASS / FAIL / N-A table with evidence (this is what proves we checked the known killers)
- Findings Summary (severity-tiered table; add a "verified against" column - source-of-truth / static / browser)
- Critical Findings narrative (one paragraph per BLOCKER)
- What's Working Well (always - balance earns trust)
- Recommended Path - phases by focus, **Hours/Weeks left as TBD** (UNPRICED) + the pricing-handoff blockquote
- Assumptions, Engagement Options, Residual Risk (incl. Codex status + which source-of-truth tools were unavailable)

Delete `_partials/` after writing.

### Step 7 - Generate branded DOCX
```bash
cd <skill-dir>/../../shared
uv venv --python 3.13 2>/dev/null; uv pip install python-docx -q
uv run python generate_audit_docx.py <path-to-master.md> <path-to-output.docx>
```
The generator renders the scorecard (grade-colored) and a full-audit footer. Open/spot-check the DOCX.

### Step 8 - Present & hand off
Show the client the verdict + scorecard summary. Then:
> "Audit complete. To price the remediation, run `code-takeover-estimation` against this audit - it consumes the findings as its review input and emits the priced phased CSV + estimation DOCX."

## Cost / runtime
~15-25 min (fanout + Codex + DOCX). Heavier than `/light-code-review` by design - this is the paid comprehensive tier.

## Common mistakes
| Mistake | Fix |
|---------|-----|
| Piping Codex output | Direct-redirect `>` only (`feedback_codex_headless_no_pipe_buffering`) |
| Passing a custom prompt with `--base` | Mutually exclusive; `--base` only |
| Blocking the audit when Codex is unauthenticated | Note it in Residual Risk, ship Claude-only findings |
| Putting hours/prices in the audit | UNPRICED - pricing is `code-takeover-estimation`'s job |
| Reviewer agents editing code | Read-only analysis only |
| Trusting a Codex "no findings" on a huge diff | Verify the file ends with a real verdict (silent-exit guard) |
| Inventing findings in synthesis | Synthesis stays in lane - no finding a reviewer didn't flag |
| Treating a green build as proof | Baseline only; verify high-sev against source of truth where access exists (Step 5) |
| Skipping the recurring-holes checklist | Mandatory PASS/FAIL/N-A per `recurring-holes.md`; it is what proves we checked the known killers |
| Single-file XSS/injection review | C9 finder MUST trace dataflow across files - that is where 2026 models fail |
| Implying coverage we lacked | Log every unavailable source-of-truth tool under Residual Risk; never imply a live check we couldn't run |
