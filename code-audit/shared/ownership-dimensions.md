# Ownership dimensions - the takeover lens

The `deep` tier hunts *exploitability* (is this code dangerous?). A **takeover**
decision also needs *ownership cost* (what is it like to OWN, fix, and extend
this code?). These four dimensions are what `deep`'s security-weighted
`dimensions.md` matrix does not surface, and they are the difference between
"here are 60 bugs" and "here is whether you should take this on, what it costs to
fix, and what to do instead."

The `takeover` tier runs the **full security fan-out** (`dimensions.md` CORE +
stack PACK + C9) **PLUS these four ownership finders**, then synthesizes a
takeover-decision document, not just a findings list. Security findings still get
the adversarial-verify layer; ownership finders are assessments (no verify
layer - one finder per dimension, sonnet).

## The four ownership dimensions

| # | Dimension | The question it answers | Hunts for | Reads against |
|---|---|---|---|---|
| O1 | **Architecture & Maintainability** | How painful is this to extend and own? | god-files / 1000+ line modules, monolithic state, circular deps, copy-paste logic, no abstraction/adapter seams, inconsistent patterns, framework misuse, "every change ripples everywhere" coupling | file sizes, import graph (codegraph), pattern consistency across modules |
| O2 | **Feature Completeness** | How much of the product is actually real? | real vs scaffolded vs stubbed vs missing, mapped to the product's intent; `//TODO`/`//FIXME`/commented-out integrations; dead screens; "UI-complete, backend-stubbed" (the gigpost + archetypes pattern); hardcoded/mock data standing in for real flows | the README/spec/product intent vs what the code actually wires up |
| O3 | **Operational Readiness** | Can we run and ship this safely on day one? | test coverage (framework + count + critical-path %), CI/CD presence, error monitoring (Sentry etc.), structured logging, deploy automation + rollback, env/secrets management, migrations-in-repo vs black-box DB, runbook/docs, observability | presence/absence in repo; CI config; test runner output |
| O4 | **Cost-to-Own** | What is the ongoing carrying cost? | dependency bloat (declared vs imported), dead code, abandoned/EOL deps, build fragility + speed, native-build overhead, oversized assets in-repo, upgrade debt (pinned-ancient frameworks), bus-factor signals (one-author, no docs) | dep manifest vs import grep, build run, git author spread |

## How these map to the existing 10-dim scorecard

The takeover deliverable grades the same A-F scorecard the `full` tier uses
(`dimensions-scorecard.md`), so the two tiers speak the same language. The
mapping:
- O1 -> Architecture Quality (+ Navigation/Routing where relevant)
- O2 -> Feature Completeness
- O3 -> Missing Infrastructure (+ Backend/Database visibility)
- O4 -> Dependency Bloat (+ Build Health)
- The security fan-out (`dimensions.md`) -> Security + Critical Bugs + Code Quality + Backend integration

## The takeover decision the synthesis must reach

Ownership findings + security findings combine into ONE verdict (verbatim
strings from `severity-map.md` so DOCX colors render):

- **TAKEOVER VIABLE** - sound foundation; defects are contained and mechanical. A
  "finish and secure" job. Most code-takeover candidates that are worth taking.
- **VIABLE - STABILIZATION REQUIRED** - takeover makes sense but a mandatory
  Phase 0 (security + stabilization) must land before any feature work. The
  default verdict for a live app with real BLOCKER/HIGH security findings on an
  otherwise competent architecture (gigpost, archetypes both land here).
- **TAKEOVER NOT VIABLE** - fundamental issues compound; remediation approaches or
  exceeds a rebuild. Recommend greenfield using the existing app as the spec.

### The takeover-vs-rebuild test (drives the verdict)
Lean **rebuild / NOT VIABLE** when several compound:
- Architecture is fundamentally wrong (O1 grade F): no seams, total coupling,
  every fix risks regressions -> remediation is unpredictable.
- Feature-completeness is low AND what exists is unsound (O2 + security both bad):
  you are paying to fix code you will mostly replace anyway.
- Operational readiness is near-zero (O3): no tests + no migrations + black-box DB
  means you cannot safely change anything and cannot reproduce prod.
- Cost-to-own is high and structural (O4): EOL framework, native-build hell, one
  undocumented author.
Lean **takeover / VIABLE** when the architecture is competent and the defects,
however numerous, are localized and mechanical (delete-this-endpoint, add-this-
guard, wire-this-stub) - numerous-but-contained is a takeover, not a rebuild.

## Strategic-advice scope (what the deliverable owes beyond findings)

The synthesis must end with advice a decision-maker can act on, not just a bug
list:
1. **The call**: take it over, stabilize-then-own, or rebuild - and the one-line why.
2. **Phase 0 definition**: the non-negotiable security + stabilization work that
   must land before any feature work (the BLOCKER/HIGH set, sequenced).
3. **Sequencing + dependencies**: what must come first (e.g. migrations into repo
   before any schema change; secret rotation before granting team access).
4. **Access / info to demand before signing**: prod credentials (rotate-first),
   a test account, the live DB/schema, the original spec, deploy access, domain/
   provider ownership. Tie each to a finding that stays unverifiable without it.
5. **Hidden ownership costs**: the carrying costs that will not show up as a bug
   (EOL upgrades coming due, native-build fragility, single-author bus factor).
6. **Effort sizing (UNPRICED)**: per-phase S/M/L effort + rough week ranges, with
   Hours/Weeks left for `code-takeover-estimation`. NEVER put dollar figures here
   (Law 4) - hand pricing to the estimation skill.
