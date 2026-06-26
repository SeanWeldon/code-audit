---
name: light-code-review
description: Fast client-facing triage of a whole codebase  -  surfaces the top 5-8 highest-risk findings + a readiness verdict in a concise markdown brief. No Codex, no DOCX, no full scorecard. Use when a client requests a light/quick code review, a takeover-viability gut-check, or the user says "light code review", "quick review of this app", "/light-code-review". For the paid comprehensive version use /full-code-audit; for OUR own pre-ship diffs use /review or the code-review skill.
---

# /light-code-review  -  Fast client code triage

The lightweight tier of the client code-review services. Same engine as `/full-code-audit` but narrowed for speed: a 2-3 agent fanout, no Codex cross-check, no DOCX, no 10-dimension scorecard. Produces a 2-3 page markdown brief a client can read in five minutes.

**Boundary:** reviews a WHOLE external/client codebase for a client deliverable. NOT for our own diffs (use `/review`/`precheck`).

## Usage

```
/light-code-review <path>                 # local path on disk
/light-code-review <github-url>           # clone first, then triage
/light-code-review <path> <client-name>   # force output under products/clients/<client-name>/
```

If `<path>` is omitted, ask for it (or confirm cwd). GitHub URL inputs are cloned to a tmp dir for review; the brief still lands in the monorepo (see Step 1).

## Read first

- `../../shared/engine.md`  -  detect → fan-out → synthesize (skip Stage 4 Codex)
- `../../shared/severity-map.md`  -  P↔client severity + verdict phrasing
- `templates/light-review-template.md`  -  the brief structure

## Workflow

### Step 1  -  Resolve target

If the argument is a GitHub URL, clone the repo first:

```
gh repo clone <owner>/<repo> /tmp/light-review/<repo>
```

For Bolder client repos use the `bolderappsdev` gh account (`gh auth switch -h github.com -u bolderappsdev` if SeanWeldon is active and the repo is private under that org).

Then confirm codebase path, app name, stack + versions. **Output location (in priority order):**

1. `<client-name>` arg given OR target lives under `products/clients/<name>/`  → `products/clients/<name>/audit/YYYY-MM-DD-light-review.md`
2. Target is a GitHub URL or any path OUTSIDE the WISE monorepo  → `reports/code-reviews/<repo-slug>/YYYY-MM-DD-light-review.md` in the monorepo (the team-shareable home; matches existing convention)
3. Target is somewhere else inside the monorepo  → `<target>/audit/YYYY-MM-DD-light-review.md`

Never default the brief into `/tmp` - briefs must land in a git-tracked, team-shareable location. Establish a run dir with `_partials/` next to the brief (or under `/tmp/light-review/.../audit/.audit-run-...` if the clone lives in /tmp).

### Step 2  -  Narrow fanout (engine Stage 2, reduced)
Dispatch **2-3** read-only reviewers in a SINGLE message:
1. **Security** (always)
2. **Primary-stack** scope  -  Frontend/Mobile OR Backend/Data, whichever the stack detection makes dominant
3. **Bugs/Robustness** (critical bugs, data-loss, crashes)

Each writes a severity-tagged partial to `_partials/`. Reviewers are read-only; they surface TOP risks, not an exhaustive list.

### Step 3  -  Synthesize the brief (engine Stage 3)
Dedupe, severity-order, map P→client severity. Write the brief from `templates/light-review-template.md`:
- Verdict (1-2 sentences + a verbatim verdict string from severity-map)
- Top Findings table  -  **5-8 rows max**, severity · area · file:line · fix direction
- What's Working Well  -  2-3 bullets
- Recommended Next Step  -  ONE clear recommendation (often: "commission a full audit" when the surface looks risky)

Delete `_partials/` after writing. No DOCX  -  the brief ships as markdown.

### Step 4  -  Present
Show the verdict + top findings. If the brief surfaces BLOCKER/HIGH density or security/data-layer risk, recommend `/full-code-audit` as the paid follow-on.

## Cost / runtime
~5 min. Deliberately cheap and fast  -  it's the triage tier. When in doubt about depth, this is the default; escalate to `/full-code-audit` only when the client is paying for the comprehensive picture.

## When to escalate to /full-code-audit
- Client is paying for a comprehensive audit / formal deliverable (branded DOCX, scorecard).
- The light pass surfaces ≥1 BLOCKER or multiple HIGH security/data findings.
- A takeover decision hinges on the full 10-dimension picture + priced remediation.
