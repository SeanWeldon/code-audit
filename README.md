# code-audit

A Claude Code plugin marketplace containing the **code-audit** plugin: a whole-codebase
audit suite driven by a single `/audit` entry point.

## What's inside

`code-audit` ships four audit depths plus one shared research engine:

| Skill | Use it for |
|-------|-----------|
| `light-code-review` | Fast gut-check of a repo or a diff |
| `full-code-audit` | Client-deliverable audit (Codex cross-check + branded DOCX report) |
| `deep-code-audit` | Internal find-everything pass |
| `takeover-audit` | Acquisition / takeover feasibility (security battery + ownership finders + acquire/stabilize/rebuild verdict) |

The shared engine covers a four-lens sweep (SAST / SCA / DAST / secrets),
verify-against-source-of-truth, the 2026 AI-codegen footgun checklist, and
supply-chain research.

## Install (Claude Code)

```
/plugin marketplace add SeanWeldon/code-audit
/plugin install code-audit@code-audit
```

After an update is published:

```
/plugin marketplace update code-audit
```

## Run

From any repo you want to audit:

```
/audit
```

The `/audit` router picks the right depth, or name one explicitly (e.g. "light review",
"full audit", "takeover audit").
