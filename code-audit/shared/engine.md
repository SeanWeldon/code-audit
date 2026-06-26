# Code Audit Engine (shared)

Canonical procedure for client-facing code review. Both `/light-code-review` and `/full-code-audit` read this. It is NOT a skill (no `SKILL.md`, not registered) - it is shared reference material.

The engine is four stages: **detect → fan out → synthesize → (full only) Codex cross-check**. Light runs stages 1-3 with a narrow reviewer set; full runs all four across all 10 dimensions.

---

## Run directory

Pick a run dir up front so partials don't collide:

```
<RUN>=<output-dir>/.audit-run-<YYYY-MM-DD-HHMM>
<RUN>/_partials/        # one file per reviewer scope
```

Delete `_partials/` after synthesis (keep the archive clean). `<output-dir>` resolution is defined in each SKILL.md.

---

## Stage 1 - Stack detection

Read the target codebase root and classify the stack before dispatching reviewers. Cheap signals:

| Signal file | Stack | Reviewer scopes to enable |
|---|---|---|
| `package.json` w/ `expo`/`react-native` | Expo / RN mobile | Security, Mobile/Frontend, Backend/Data, Infra |
| `package.json` w/ `next`/`react`/`vite` | Web frontend | Security, Frontend, Backend/Data, Infra |
| `pubspec.yaml` | Flutter | Security, Mobile/Frontend, Backend/Data, Infra |
| `requirements.txt` / `pyproject.toml` | Python backend | Security, Backend/Data, Infra, Architecture |
| `supabase/` dir, edge functions | Supabase backend | Security (RLS/IDOR), Backend/Data |
| `go.mod` / `Cargo.toml` / other | generic | Security, Backend/Data, Architecture, Infra |

Capture for the deliverable header: app name, framework + versions, LOC (rough `git ls-files | xargs wc -l` or `find`-based), dependency count, test-file count, last-commit date.

**Always enable Security.** Add the rest by detection. Architecture/Quality is always reviewed (it's stack-agnostic).

---

## Stage 2 - Reviewer fan-out (parallel, single message)

Dispatch read-only review subagents in ONE message so they run in parallel and share the prompt cache. Each subagent owns ONE scope, reviews ALL relevant files, writes a partial, and **invents nothing outside its scope**. Reviewer prompt shape mirrors `.claude/agents/reviewers/security-fanout.md`:

- Priority-ordered "what to flag" list for the scope
- Explicit "out of scope for you - other reviewers cover those"
- Output contract: write to `<RUN>/_partials/<scope>.md`, severity-tagged findings only, `No <scope> findings.` if clean
- **Read-only**: never edit code.

Scopes map onto the 10 dimensions in `dimensions-scorecard.md`:

| Scope | Dimensions covered |
|---|---|
| Security | 3 (auth/RLS), 7 (security) |
| Frontend/Mobile | 4 (navigation), 5 (feature completeness) |
| Backend/Data | 3 (backend/db) |
| Architecture/Quality | 2 (architecture), 6 (code quality) |
| Infra/Build | 1 (build health), 8 (missing infra), 9 (dep bloat) |
| Bugs/Robustness | 10 (critical bugs) |

**Light review:** dispatch only Security + the primary-stack scope (Frontend/Mobile OR Backend/Data) + Bugs/Robustness - 2-3 agents, ~5 min.
**Full audit:** dispatch all scopes that the detected stack warrants - typically 4-6 agents - so every one of the 10 dimensions is covered.

Each reviewer emits findings tagged `[P0]`/`[P1]`/`[P2]`/`[P3]` with `file:line` evidence (matches the internal reviewer contract; the synthesizer maps to client severity).

---

## Stage 3 - Synthesis

One synthesizer (you, the main agent, or a dedicated subagent) consumes all partials. Mirrors `.claude/agents/reviewers/synthesize.md`:

1. **Dedupe** - same `file:line` + same root cause → keep the more specific, note the second scope in parens.
2. **Severity-order** - BLOCKER → HIGH → MEDIUM → LOW. Do NOT re-grade.
3. **No invention** - if no reviewer flagged it, don't add it.
4. **Map P→client severity** per `severity-map.md` (P0→BLOCKER, P1→HIGH, P2→MEDIUM, P3→LOW).
5. **Empty case** - all partials clean → "No findings." and a clean readiness verdict.
6. **Missing partial** - note the missing scope under Residual Risk / Assumptions.

For a **full audit**, also assign each of the 10 dimensions an A-F grade + risk from the findings (rubric in `severity-map.md`) to populate the scorecard.

---

## Stage 4 - Codex cross-check (FULL AUDIT ONLY)

Run Codex as an independent second model to catch hallucinated findings (both directions) before the deliverable goes to a client. Pattern is the validated `precheck` / `.githooks/pre-push` invocation:

```bash
TS=$(date +%Y-%m-%dT%H-%M-%S)
OUT="<RUN>/codex-crosscheck-${TS}.md"
nohup codex exec review --base <BASE_SHA> \
  --skip-git-repo-check \
  --dangerously-bypass-approvals-and-sandbox \
  --title "full-code-audit: <AppName>" \
  > "$OUT" 2>&1 &
PID=$!
```

Rules (all validated, do not deviate):
- **Direct-redirect with `>`, NEVER pipe** - pipes buffer codex output indefinitely (`feedback_codex_headless_no_pipe_buffering`).
- `--base <sha>` and a custom `[PROMPT]` are **mutually exclusive** - `codex exec review` is purpose-built; pass `--base` only.
- Wait via the **Monitor tool until-loop** (`until ! ps -p <PID>...`), not `sleep` polling. Timeout 600000ms.
- Extract the verdict after the `^codex$` sentinel: `awk '/^codex$/{flag=1; next} flag' "$OUT" | head -120`.
- **Silent-exit guard:** if the diff spans >50 files / >5000 LOC, scope tighter; verify the output file ends with a real verdict, not just exploration logs (`feedback_codex_cli_silent_exit_workaround`).

**Non-git client codebase:** `git init` a throwaway baseline commit so `--base` has something to diff against, OR run Codex per-subdirectory. **Codex missing/unauthenticated:** do NOT block the audit - record "Codex cross-check unavailable" under Residual Risk and ship the Claude-only findings.

**Reconciliation:** compare Codex findings to the synthesized Claude findings.
- Both flag it → confidence ↑, keep.
- Only Codex → add it (verify the `file:line` exists first - `pattern_subagent_file_line_claims_verify`).
- Only Claude, Codex silent on a file it reviewed → keep but consider demoting severity one notch if it was a judgment call.
- Never let Codex *remove* a verified secret/RLS/data-loss finding.

---

## Output

Each SKILL.md defines its own deliverable template and output path. The engine's job ends at a synthesized, severity-mapped, (optionally Codex-reconciled) finding set + dimension grades.
