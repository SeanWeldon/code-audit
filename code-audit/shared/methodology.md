# Audit methodology - the four lenses + verify-against-truth

The "why" behind the whole `code-audit` suite (`light` / `full` / `deep`). Each
tier applies this at its own depth: `light` reads it for framing, `full` adds
Codex + opportunistic source-of-truth verification, `deep` runs the full
verification battery. Sources current as of 2026-06-08.

## Defense in depth: four complementary lenses

No single lens finds everything; each catches what the others miss. A real audit runs all four ([ZeonEdge](https://zeonedge.com/hy/blog/cicd-pipeline-security-sast-dast-secrets-scanning-dependency-audit), [Oligo 2025](https://www.oligo.security/academy/application-security-testing-in-2025-techniques-best-practices), [SentinelOne](https://www.sentinelone.com/cybersecurity-101/cybersecurity/code-security-audit/)):

1. **SAST (static / white-box)** - read source for injection, auth flaws, logic and money-math bugs, error handling. The bulk of the fan-out. Use delta-awareness on huge repos (focus newly-changed + high-risk surfaces) to cut noise.
2. **SCA (software composition analysis)** - known CVEs in direct AND transitive deps, plus license risk. We already own `capabilities/supply_chain_guard` (age gate + OSV); run it.
3. **DAST (dynamic / black-box)** - exercise the RUNNING app for runtime, integration, and misconfig flaws that source review cannot see (auth flows, race conditions, env-specific 5xx). For us: live browser walk (Chrome MCP) + provider test-mode probing.
4. **Secrets scanning** - credentials in source AND git history; env-var scoping (public-prefixed secrets).

## The governing principle: trust the running system, not the green build

The most important rule, learned repeatedly on real client code: **a passing `tsc`/build is a baseline, not proof.** Type systems give false confidence. Concrete failure classes that compile clean and only break at runtime:
- Supabase-js `.select('a,b,c')` / `.or()` / `.eq()` column strings are not tsc-checked against the schema, so DB column drift 500s only at runtime (see `feedback_supabase_select_runtime_column_drift`).
- Timezone-naive date parsing passes types, wrong at runtime.
- Stale generated DB types that no longer match prod.

Therefore every surviving finding is verified against ground truth (live `information_schema`, DB advisors, `pg_stat_statements`, provider test mode, a live browser walk) - never re-confirmed by the build. Retract any finding the source of truth refutes, fast.

## Verify-before-claiming (applies to the audit itself)

Same discipline we apply to bug calls: do not report a finding from a single weak signal. Require two independent signals (e.g. static match + live repro, or code grep + `information_schema` mismatch). State coverage gaps honestly (no Stripe keys -> payment findings are static + browser-DAST only). Never imply a tool ran that did not.

## The recurring-holes principle for THIS stack

For Next.js + Supabase + Stripe, the same handful of holes are "the default output of AI codegen, found in live paying apps within hours" ([Stripe-on-Supabase holes](https://aicourses.com/ai-for-developers/stripe-payments-nextjs-supabase-security/), [Next.js+Supabase hardening](https://checkvibe.dev/blog/secure-nextjs-supabase-app)). They become a MANDATORY checklist (see `recurring-holes.md`), not a hope-we-catch-it. Treat any net-new stack the same way: build its recurring-holes checklist before the fan-out.

## 2026 research: assume AI-generated code, audit for what the models miss

Most codebases we audit are AI-built (vibe-coded / agent-built). The 2026 data
says that changes the threat model - audit for where the models are *measurably*
weak, not for an evenly-distributed bug field.

**AI code is less secure than human code, by a wide margin.** A 534-sample study
across six LLMs found 25.1% carried a confirmed OWASP Top-10 vulnerability;
CVSS 7.0+ flaws appear ~2.5x more often and overall vuln density is ~2.7x human
code ([AppSec Santa / paperclipped 2026](https://www.paperclipped.de/en/blog/ai-generated-code-security-vulnerabilities/), [Veracode Spring 2026](https://www.veracode.com/blog/spring-2026-genai-code-security/)). "Nearly half of all AI-generated code contains
known vulnerabilities when no security guidance is explicitly provided."

**The models are worst at a predictable shortlist - hunt these first:**
- **XSS (CWE-80): ~15% security pass rate. Log injection (CWE-117): ~13%.** The
  two worst categories, and *getting worse* per Veracode. Every audit of an
  AI-built app runs an explicit XSS + log/output-injection sweep.
- **Cross-file / multi-line dataflow holes.** Models pattern-match a single
  file; they fail when a vuln spans files (tainted input in route A, sink in
  service B). Finder prompts MUST trace dataflow *across* files, not scan one
  file at a time - this is the exact gap.
- **"Implements the feature, forgets the guard."** Design-level flaws are up
  153%: auth bypass, broken session management, and access-control gaps where
  the update logic exists but the role/ownership check was never added. Pair
  every mutating route with "is the ownership + role check present?"
- Models are comparatively *strong* at SQLi (CWE-89, ~82%) and weak-crypto
  (CWE-327, ~86%) - still check, but they are not where the yield is.

This is the **AI-codegen footgun pass** (see `recurring-holes.md`): a fixed
reviewer scope every tier runs on top of the stack pack.

## 2026 research: supply chain went wormable and CI-native

SCA is no longer "scan for known CVEs in deps." The 2026 npm campaigns added
attack classes the audit's CI/CD + SCA dimensions must hunt directly:
- **Wormable credential-stealing packages** (Shai-Hulud family): malicious
  postinstall harvests npm tokens + GitHub PATs to auto-republish, and AWS /
  Vault / CI secrets from the host ([Microsoft, May 2026](https://www.microsoft.com/en-us/security/blog/2026/05/28/typosquatted-npm-packages-used-steal-cloud-ci-cd-secrets/)).
- **CI-native persistence with valid provenance** (Miasma / Red Hat npm): bypass
  code review entirely, steal GitHub Actions OIDC tokens, ship trojaned packages
  carrying *valid* SLSA provenance ([Microsoft, Jun 2026](https://www.microsoft.com/en-us/security/blog/2026/06/02/preinstall-persistence-inside-red-hat-npm-miasma-credential-stealing-campaign/)).
- **`pull_request_target` Pwn Request + Actions cache poisoning** across the
  fork-base trust boundary ([TanStack postmortem, May 2026](https://tanstack.com/blog/npm-supply-chain-compromise-postmortem)).

Audit consequences: run `capabilities/supply_chain_guard` (age gate + OSV);
grep workflows for `pull_request_target`, shared caches in publish jobs, and
unpinned actions; flag any `preinstall`/`postinstall` script in a dependency;
treat a public-prefixed secret next to a CI token as a P0. These cross-reference
the rules in `~/.claude/rules/security.md`.
