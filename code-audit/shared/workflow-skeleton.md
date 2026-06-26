# Deep-audit Workflow skeleton (tiered, cost-disciplined)

The orchestration recipe for Step 4. Copy + adapt this into a `Workflow({script})` call. It is the canonical "find -> adversarially verify -> synthesize" pipeline with the model-tiering and verify-cap that keep a 250K-LOC audit from becoming a 2.5M-token incident.

## Rules baked into the shape
- **pipeline() not parallel()**: each dimension verifies its own findings the moment that dimension's finder returns - no barrier across dimensions.
- **TIER MODELS**: finders + verifiers are `model:'sonnet'`; the single synthesis is Opus (omit `model`). An all-Opus fan-out is BLOCKED by `pre-tool-workflow-budget.sh`.
- **CAP the verify layer**: only P0/P1 (BLOCKER/HIGH) findings get the adversarial verify fan-out. P2/P3 pass through unverified (noted as such). This stops the (findings x verifiers) multiplication.
- **READ-ONLY**: finder/verifier prompts forbid edits. The audit produces a report, not commits.
- **Announce the count first**: sum finder shards + (expected P0/P1 findings x verifiers) + 1. Tell the user before launching (Step 3 gate).

## Skeleton

```js
export const meta = {
  name: 'deep-code-audit',
  description: 'Deep multi-dimension audit of a mature codebase, verified against source of truth',
  phases: [{ title: 'Find' }, { title: 'Verify' }, { title: 'Synthesize' }],
}

// DIMENSIONS: built in Step 1 from dimensions.md (CORE + detected PACK).
// Shard big ones (security, payments) into sub-scopes so each finder owns a tractable surface.
const DIMENSIONS = args.dimensions  // [{key, prompt, files}] passed in via Workflow args

const FINDINGS_SCHEMA = { /* { findings: [{id, dimension, file, line, severity, title, evidence, rootCause, recommendedFix}] } */ }
const VERDICT_SCHEMA  = { /* { id, real: boolean, reason, sourceOfTruthChecked } */ }

const results = await pipeline(
  DIMENSIONS,
  // Stage 1 - FIND (sonnet, read-only)
  d => agent(
    `READ-ONLY audit of the ${d.key} dimension over: ${d.files}.\n${d.prompt}\n` +
    `Emit P0-P3 findings with file:line evidence. Invent nothing outside ${d.key}. ` +
    `For the recurring-holes checklist items in your scope, report PASS/FAIL/N-A explicitly.`,
    { label: `find:${d.key}`, phase: 'Find', model: 'sonnet', schema: FINDINGS_SCHEMA }
  ),
  // Stage 2 - VERIFY (sonnet) - only P0/P1, each adversarially.
  // CRITICAL: pipeline() returns ONLY the last stage. If this stage returns just
  // the verified highs, every P2/P3 from Stage 1 is SILENTLY DROPPED from the
  // report (real bug, archetypes audit 2026-06-08). So this stage MUST return an
  // object carrying BOTH the full finder output and the verified highs.
  async (found, d) => {
    const all = found?.findings || []
    const verdicts = await parallel(
      all.filter(f => f.severity === 'P0' || f.severity === 'P1')
        .map(f => () => agent(
          `Adversarially verify this ${d.key} finding. Try to REFUTE it. ` +
          `Check against the SOURCE OF TRUTH, not the build: ` +
          `for schema/DB use information_schema via Supabase MCP; for RLS use get_advisors; ` +
          `for perf use pg_stat_statements; for runtime use a browser walk. ` +
          `If you cannot confirm against ground truth, mark real=false. Finding: ${JSON.stringify(f)}`,
          { label: `verify:${f.id}`, phase: 'Verify', model: 'sonnet', schema: VERDICT_SCHEMA }
        ).then(v => ({ ...f, verdict: v })))
    )
    return { dimKey: d.key, allFindings: all, verifiedHigh: verdicts.filter(Boolean).filter(f => f.verdict?.real) }
  }
)

// Collect BOTH the verified highs AND every P2/P3 (carried unverified) for synthesis.
const perDim = results.filter(Boolean)
const allFindings = perDim.flatMap(r => r.allFindings || [])         // every finding, all severities
const confirmed = perDim.flatMap(r => r.verifiedHigh || [])          // P0/P1 that survived verification
const lowerSev = allFindings.filter(f => f.severity === 'P2' || f.severity === 'P3')

// Stage 3 - SYNTHESIZE (Opus, single)
const report = await agent(
  `Synthesize a severity-ranked deep-audit report. ` +
  `Dedupe by file:line+root-cause, order BLOCKER->HIGH->MEDIUM->LOW, do not re-grade, invent nothing. ` +
  `Mark any P0/P1 that FAILED verification as unconfirmed - do not drop it silently. ` +
  `Include the recurring-holes checklist results and a triage table. ` +
  `Verified highs: ${JSON.stringify(confirmed)} ; medium/low: ${JSON.stringify(lowerSev)} ; full finder output: ${JSON.stringify(allFindings)}`,
  { label: 'synthesize', phase: 'Synthesize', schema: REPORT_SCHEMA }  // no model => Opus
)
return report
```

## Sizing examples (calibrated against the 2026-06-06 Ora run)
- **~250K LOC, Next+Supabase+Stripe** (Ora, actual): 14 finders. The finders surfaced **35 P0/P1 findings**, each spawning a verifier, so the real total was **88 agents / ~5M tokens, NOT the ~35 I projected.** The verify layer is `finders + (P0/P1 count x refuters)`, and on a finding-rich legacy codebase the P0/P1 count is the dominant term and is UNKNOWN until the finders return.
- **Smaller repo (<50K LOC)**: do not shard; ~10 finders + a thin verify layer + 1 synthesis = ~15 agents.

## Announce a RANGE, and damp the verify term when finders are rich
The verify multiplication is the trap (real incident: a 2.5x overrun on Ora because I announced a point estimate of ~35 and the P0/P1 yield was 35, not the ~15 I guessed). Two rules:
1. **Announce a range, not a point**: "N finders + roughly (expected_high_sev x 1) verifiers + 1 synthesis; on a legacy codebase expect the high-sev count to run high, so the total could be 2-3x the finder count." For a 250K-LOC legacy app, say **"30-90 agents depending on how many blockers turn up."**
2. **Damp the verify term** when you expect a rich yield: cap verify to **P0 only** (not P0+P1), or batch findings (one verifier judges 3-5 related findings in a single call), or run a single verifier per dimension that re-checks all that dimension's high-sev findings at once. The schema/RLS findings verify cheaply against `information_schema` in bulk - one query confirms many.

## Source-of-truth tools the verifiers use
- `mcp__plugin_supabase_supabase__execute_sql` -> `information_schema.columns` (schema drift), `pg_stat_statements` (perf)
- `mcp__plugin_supabase_supabase__get_advisors` (RLS/security lints)
- Chrome MCP browser walk (runtime/DAST) - load tools via ToolSearch first
- `capabilities/supply_chain_guard` + OSV (SCA) - run outside the workflow, fold results into C6
- Stripe test mode ONLY if test keys are present; otherwise payment findings stay static + browser-DAST and the gap goes in Residual Risk
