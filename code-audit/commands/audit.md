---
description: Whole-codebase audit router. Dispatches to the right tier - light (fast gut-check), full (paid client deliverable + Codex + DOCX), deep (internal find-everything on a codebase we own), or takeover (acquire-or-rebuild feasibility decision). No verb prints a decision tree and recommends one tier from cheap signals.
---

# /code-audit:audit

Single entry point for whole-codebase auditing. Routes a tier verb to the bundled
skill and forwards remaining args. Pure pass-through: does NOT re-implement
anything. Invocable as `/audit` or `/code-audit:audit`.

All three tiers read one shared research engine (`../shared/`): the four-lens
methodology (SAST/SCA/DAST/secrets), the verify-against-source-of-truth
principle, the mandatory recurring-holes checklist, the 2026 AI-codegen footgun
pass, and the supply-chain hardening list.

## Usage

- `/audit` - print the tier table + decision tree, no action
- `/audit <tier> <path> [name]` - dispatch to the tier's skill, forwarding args

## Tiers

| Tier | Dispatches to | Audience | Output | When |
|---|---|---|---|---|
| `light` | `light-code-review` skill | us | top 5-8 findings + readiness verdict, ~5 min | fast takeover gut-check before committing to deeper work |
| `full` | `full-code-audit` skill | client / buyer | branded UNPRICED DOCX scorecard + Codex cross-check | a paid client audit / pre-takeover for-sale decision |
| `deep` | `deep-code-audit` skill | us, the maintaining team | severity-ranked findings report -> triage -> PRs | a large codebase we OWN; "find everything wrong" |
| `takeover` | `takeover-audit` skill | us, deciding to acquire | feasibility verdict + 10-dim scorecard + phased fix roadmap (effort) + takeover-vs-rebuild + strategic advice -> hands off to estimation | deciding whether to ACQUIRE + own a codebase; "should we take this over and what does it cost" |

## Decision tree (printed by the no-arg form)

1. Deciding whether to ACQUIRE + own a codebase (feasibility + fix roadmap + rebuild-vs-takeover + strategy)? -> `takeover`
2. Codebase we already OWN and maintain, want every defect found + fixed? -> `deep`
3. Client paying for a for-sale audit / lighter branded DOCX scorecard? -> `full`
4. Just need a fast read on whether code is worth deeper work? -> `light`

> `takeover` vs `full`: both inform an acquisition, but `full` is the lighter client-facing for-sale DOCX (parallel fanout + Codex), while `takeover` is the heavy internal decision tool (deep verified security battery + ownership finders + a take/stabilize/rebuild verdict + strategic advice). `takeover` vs `deep`: `deep` finds every bug in code we ALREADY own; `takeover` adds the ownership lens + decision for code we are deciding to own.

## Process

1. **Parse tier**: read the first positional arg.
2. **No tier given**: print the tier table + decision tree above, then recommend
   one from cheap signals (do NOT auto-run):
   - user said "take over"/"acquire"/"should we own this"/"takeover" or it is a code-takeover candidate clone -> recommend `takeover`
   - target path under `products/clients/` or an external clone, client paying for a for-sale audit DOCX -> recommend `full`
   - a repo we own/maintain (under this monorepo's `products/` we built, or the user says "we own this") -> recommend `deep`
   - user said "quick"/"gut-check"/"is it worth it" -> recommend `light`
   Stop. Do not invoke any skill.
3. **Unknown tier**: print the tier table, append `did you mean <closest>?`, exit
   without invoking anything.
4. **Known tier**: invoke the corresponding skill by name via the Skill tool,
   passing through every arg after the tier verbatim (path, client/project name).
5. **Hand off**: once dispatched, the router is done. The skill owns output,
   artifacts, and exit code. Do not post-process.

## Plugin boundaries

This plugin owns WHOLE-CODEBASE auditing only.

OUT of scope (handled elsewhere):
- Our own staged-diff / pre-ship review -> `bolder-quality` (`/review`, `/precheck`, `/bolder-review`)
- Internal whole-repo security gate vs a threat model -> `security-scan` skill
- Pricing the remediation -> `code-takeover-estimation` skill (consumes a `full` audit's findings)
- Confirming a single change works -> `verify` skill

`bolder-quality`'s `/review audit` and `/review light` modes route INTO this
plugin. When a `full` audit completes, recommend `code-takeover-estimation` for
pricing; do not call it directly.

## Errors

- Unknown tier -> print this help, exit non-zero.
- Tier given but no `<path>` -> ask for the path (or confirm cwd), do not guess.
