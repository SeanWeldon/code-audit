---
name: takeover-audit
description: Takeover-feasibility deep audit of a codebase we are deciding whether to ACQUIRE and own. Runs the full deep security fan-out (SAST/SCA/DAST/secrets, C9 AI-codegen footguns, adversarial-verify) PLUS four ownership finders (architecture/maintainability, feature-completeness, operational-readiness, cost-to-own), then synthesizes a takeover-decision document: feasibility verdict, 10-dim scorecard, phased remediation roadmap with effort + sequencing, takeover-vs-rebuild call, and strategic advice. UNPRICED - hands pricing to code-takeover-estimation. Use when "should we take this over", "takeover audit", "/audit takeover", or assessing acquisition feasibility. For pure find-everything on code we already own use /deep-code-audit; for the lighter client for-sale DOCX use /full-code-audit.
---

# /takeover-audit - Is this code worth acquiring, and what does owning it cost?

The `deep` tier answers "is this code dangerous?" This tier answers the takeover
questions: **is a takeover feasible, what is needed to fix it, and what is the
strategic call (take it over, stabilize-then-own, or rebuild)?** It runs the full
deep security battery AND the ownership lens, then ends with a decision and
advice, not just a findings list.

## Boundary - pick the right tool

| Skill | Question | Output |
|---|---|---|
| **takeover-audit** (this) | should we ACQUIRE + own this, and what does that cost? | takeover-decision doc: verdict + scorecard + phased fix roadmap + rebuild-vs-takeover + strategic advice |
| `deep-code-audit` | find every defect in code we ALREADY own | severity-ranked engineering findings -> triage -> PRs |
| `full-code-audit` | client-facing paid for-sale audit (lighter parallel fanout + Codex + DOCX) | branded UNPRICED scorecard DOCX |
| `light-code-review` | fast 5-min gut-check before committing to deeper work | top-risks triage |
| `code-takeover-estimation` | price the remediation | priced phased CSV + estimation DOCX |

If the takeover decision is already MADE and you just want every bug, use
`deep-code-audit`. If you need the *price*, this audit hands off to
`code-takeover-estimation`.

## Read first

- `../../shared/methodology.md` - the four lenses + verify-against-source-of-truth + 2026 AI-codegen research. The "why".
- `../../shared/dimensions.md` - the security fan-out matrix (CORE + stack PACK + C9). The exploitability hunt.
- `../../shared/ownership-dimensions.md` - **the four ownership finders (O1-O4) + the takeover-vs-rebuild test + the strategic-advice scope. This is what makes this tier a takeover audit, not a security audit.**
- `../../shared/recurring-holes.md` - mandatory PASS/FAIL/N-A checklist + AI-codegen footgun pass + supply-chain hardening.
- `../../shared/workflow-skeleton.md` - the tiered Workflow recipe (find -> adversarial-verify -> synthesize) with model-tiering + agent-count discipline.
- `../../shared/dimensions-scorecard.md` - the 10-dim A-F scorecard the deliverable grades against.
- `../../shared/severity-map.md` - P0-P3 <-> BLOCKER/HIGH/MEDIUM/LOW, A-F rubric, and the three VERBATIM verdict strings.
- `templates/takeover-feasibility-report.md` - the deliverable structure.

## Workflow

### Step 1 - Scope, shape, and product intent
Resolve the path (codebase must be cloned locally). Capture header metrics (LOC, deps, test count, last commit, commit count, author spread). Detect the stack and select the security PACK (`dimensions.md`). **Also capture the product's INTENT** (README, spec, landing copy, the client's own description) - O2 feature-completeness is meaningless without "complete relative to WHAT". Exclude non-shipping dirs (marketing/docs/assets) from the deep read; announce them.

### Step 2 - Build/health + ownership baseline
Run the stack build/typecheck (`tsc --noEmit`, `expo export`, `flutter analyze`, `node -c`). A green build is a BASELINE not proof (`methodology.md`). Capture cheap ownership signals up front: file-size distribution (god-files), test-file count + whether tests run, CI presence (`.github/workflows`), migrations-in-repo, declared-vs-imported dep delta, `//TODO`/`//FIXME`/commented-integration counts. These seed the O1-O4 finders.

### Step 3 - Size the run and GET EXPLICIT GO (mandatory gate)
Same discipline as `deep` (the Workflow tool spends real tokens fast). The fan-out is **security finders (from `dimensions.md`) + 4 ownership finders (O1-O4) + a CAPPED adversarial-verify layer (security P0/P1 only - ownership finders are NOT verified, they are assessments) + 1 synthesis.** The verify layer is `security_finders + (P0/P1 count x refuters)` and the P0/P1 count is unknown until finders return. **Announce a RANGE** (a finding-rich legacy app runs high) and **TIER MODELS**: all finders + verifiers `model:'sonnet'`, ONLY the final synthesis Opus (the budget hook BLOCKS an all-Opus fanout). **Announce the projected agent count and wait for an explicit go.** Never auto-launch.

### Step 4 - Run the audit workflow
Build the script from `../../shared/workflow-skeleton.md`, extended: a **pipeline by dimension** for the security finders (each finds, then its P0/P1 are adversarially verified) AND a **parallel block of the 4 ownership finders** (O1-O4, sonnet, no verify layer - they emit graded assessments + evidence, not exploit claims). Each ownership finder reads `ownership-dimensions.md` for its scope and returns: a grade (A-F), risk, the key evidence (`file:line` / metrics), and a one-paragraph maintainability/completeness/readiness/cost read. All reviewers are READ-ONLY.

### Step 5 - Verify security findings against source of truth (the differentiator)
For each surviving security BLOCKER/HIGH, confirm against ground truth where access exists (NOT against the build): live `information_schema` for schema/DB, `get_advisors` for RLS, `pg_stat_statements` for perf, Stripe TEST mode only if keys present, a live browser/curl walk for runtime + auth. If the app is self-contained (e.g. SQLite), boot it locally and run a DAST walk with a CONTROL request to a known-gated endpoint (a passing 401/403 control proves the auth boundary works, making every other unauth 200 a confirmed bypass). Drop findings the source of truth refutes. Log every unavailable tool under Residual Risk - never imply coverage we lacked. Ownership findings need no live verification (they are structural reads).

### Step 6 - Synthesize the takeover-decision document
ONE Opus synthesis agent writes the deliverable per `templates/takeover-feasibility-report.md` to `<repo>/audit/YYYY-MM-DD-takeover-audit.md`. It must reach a DECISION, not just list findings:
1. **Verdict** (verbatim string from `severity-map.md`) + the headline why, applying the takeover-vs-rebuild test in `ownership-dimensions.md`.
2. **By-the-numbers** + the **10-dim A-F scorecard** (security dims from the fan-out, O1-O4 mapped per `ownership-dimensions.md`).
3. **Security findings** severity-ranked (the deep report, with "verified against" column) - the exploitability appendix.
4. **Phased remediation roadmap**: Phase 0 (security + stabilization, the BLOCKER/HIGH set) -> Phase 1 (core workflow) -> Phase 2 (feature gaps from O2) -> Phase 3 (production readiness from O3). Each phase: focus, the findings it closes, **S/M/L effort + a rough week range**, and **sequencing/dependencies**. Hours/Weeks columns left TBD (UNPRICED).
5. **Takeover vs rebuild** framing (the two-path table when the verdict is borderline or NOT VIABLE).
6. **Strategic advice** (the 6-point scope in `ownership-dimensions.md`): the call, Phase 0 definition, sequencing, access/info to demand before signing, hidden ownership costs, effort sizing.
7. **What's solid** + **Residual risk & coverage gaps**.
Mirror a dated entry to Obsidian `Projects/<name>/Progress Log` if it is a tracked project.

### Step 7 - Generate DOCX (optional, on request) + hand off
The report is grade-aware but image-light; render with the grade-free generator when a shareable artifact is wanted:
```bash
cd <skill-dir>/../../shared
uv run --with python-docx python generate_report_docx.py <report.md> <output.docx>
```
For a client-facing version, strip pricing (Law 4) and never print live credentials. Then hand off:
> "Takeover audit complete. To price the remediation, run `code-takeover-estimation` against this audit - it consumes the phased roadmap + findings and emits the priced task-level CSV + estimation DOCX."

## Cost / runtime
Heaviest tier (security battery + ownership finders + synthesis). ~20-35 min + real token spend. That is why the Step 3 gate exists. The payoff: a defensible acquire/stabilize/rebuild decision backed by source-of-truth-verified findings.

## Common mistakes
| Mistake | Fix |
|---------|-----|
| Auto-launching the workflow | Step 3 gate: announce agent count, get explicit go |
| All-Opus fan-out | sonnet finders/verifiers/ownership, Opus only for synthesis (budget hook blocks otherwise) |
| Adversarially verifying the ownership finders | O1-O4 are assessments, not exploit claims - no verify layer; only security P0/P1 get refuters |
| Grading O2 feature-completeness with no product intent | Step 1 captures the README/spec first - "complete vs WHAT" |
| Producing a findings list with no verdict | This tier MUST reach take-over / stabilize / rebuild + the strategic advice |
| Putting dollar figures in the report | UNPRICED - hand pricing to `code-takeover-estimation` (Law 4) |
| Trusting the green build | Verify security high-sev against source of truth (Step 5) |
| Reviewers editing code | Read-only; this is an assessment, not a remediation |
| Calling numerous-but-localized defects a rebuild | Apply the takeover-vs-rebuild test: contained + mechanical = takeover, structural + compounding = rebuild |
