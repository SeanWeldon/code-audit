# code-audit

Whole-codebase audit suite for Bolder Apps / WISE. Four tiers, one shared
research engine, one entry point (`/audit`). Self-contained and drop-in for any
repo.

| Tier | Audience | Output | When |
|---|---|---|---|
| `light` | us | top 5-8 findings + readiness verdict (~5 min) | fast takeover gut-check |
| `full` | client / buyer | branded UNPRICED DOCX scorecard + Codex cross-check | paid client audit / for-sale decision |
| `deep` | maintaining team | severity-ranked findings report -> triage -> PRs | a codebase we OWN; find everything |
| `takeover` | us, deciding to acquire | feasibility verdict + scorecard + phased fix roadmap + takeover-vs-rebuild + strategic advice | deciding whether to ACQUIRE + own a codebase |

## Usage

```
/audit                       # tier table + decision tree, recommends one tier
/audit light <path>          # fast gut-check
/audit full <path> [client]  # paid client deliverable
/audit deep <path> [project] # internal find-everything audit
/audit takeover <path> [name]# acquire-or-rebuild feasibility decision
```

## What makes it high quality

One shared knowledge base (`shared/`) that every tier reads at its own depth:

- **methodology.md** - the four lenses (SAST / SCA / DAST / secrets),
  "trust the running system not the green build", and the 2026 research:
  AI-generated code carries ~2.7x the vuln density of human code and is worst at
  XSS (CWE-80) and log injection (CWE-117); supply chain went wormable and
  CI-native (Shai-Hulud, Miasma, TanStack).
- **recurring-holes.md** - the mandatory PASS/FAIL/N-A checklist: the AI-codegen
  footgun pass (every AI-built app), the Next+Supabase+Stripe stack pack, and
  the 2026 supply-chain hardening list.
- **dimensions.md** - the CORE + stack PACK hunt matrix (incl. C9 AI-codegen
  footguns) that drives the security finder fan-out.
- **ownership-dimensions.md** - the four ownership finders (O1 architecture,
  O2 feature-completeness, O3 operational-readiness, O4 cost-to-own) + the
  takeover-vs-rebuild test + strategic-advice scope. Drives the `takeover` tier.
- **dimensions-scorecard.md** - the 10-dim client scorecard that drives the DOCX.
- **engine.md** - detect -> fan-out -> synthesize -> (full) Codex cross-check.
- **severity-map.md** - P0-P3 <-> client severity + A-F grading.
- **workflow-skeleton.md** - the deep tier's cost-disciplined tiered Workflow.
- **generate_audit_docx.py** - branded DOCX generator (full tier).

## Boundaries

Owns whole-codebase auditing only. Diff / pre-ship review lives in
`bolder-quality` (`/review`, `/precheck`); pricing lives in
`code-takeover-estimation`; the internal threat-model security gate is the
`security-scan` skill. `bolder-quality`'s `audit`/`light` review modes route into
this plugin.

## Layout

```
code-audit/
|-- .claude-plugin/plugin.json
|-- commands/audit.md          # the /audit router
|-- skills/                    # light-code-review, full-code-audit, deep-code-audit, takeover-audit
`-- shared/                    # the audit knowledge base (all tiers read it)
```
