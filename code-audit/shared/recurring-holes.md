# Recurring-holes checklist + AI-codegen footgun pass

The "default output of AI codegen, found in live paying apps within hours."
Every audit whose stack matches reports each item as **PASS / FAIL / N-A with
evidence** - never silently omitted. This is the part that turns an audit from
"we looked around" into "we checked the known killers." All three tiers run it
(`light` runs the stack checklist + footgun pass; `full` and `deep` add the
supply-chain list and verify against source of truth).

## A. AI-codegen footgun pass (EVERY AI-built app, stack-agnostic)

Run this on every audit unless you have positive evidence the code is
hand-written. Sourced from the 2026 data in `methodology.md` (Veracode Spring
2026, AppSec Santa). These are where the models measurably fail:

1. **XSS / unescaped output (CWE-80)** - ~15% model pass rate, the single worst
   class. Hunt every render of user/DB content: `dangerouslySetInnerHTML`,
   unescaped template injection, `v-html`, WebView `injectedJavaScript`,
   `expo-print`/PDF HTML, markdown renderers fed raw input. (P0/P1.)
2. **Log / output injection (CWE-117)** - ~13% pass rate, getting worse. User
   input logged or echoed without sanitization (CRLF into logs, ANSI into a
   terminal sink, unescaped into an error page). (P1/P2.)
3. **Cross-file dataflow holes** - the models pattern-match a single file and
   miss taint that crosses files (input validated in route A, used unsafely in
   service B). TRACE the dataflow across files; do not scan one file at a time.
   (severity per sink.)
4. **Feature implemented, guard forgotten** - design-level access control: a
   mutating route/action that does the write but never checks
   ownership/role (`user_id === auth.uid`, `is_admin`). Pair every mutation with
   "where is the guard?" Design-level flaws are up 153% in AI code. (P0/P1.)
5. **Don't over-index on SQLi/crypto** - models are comparatively strong there
   (~82% / ~86%). Still check, but the yield is in 1-4.

## B. Stack checklist: Next.js + Supabase + Stripe (canonical)

The pack we built for Ora. Each item PASS / FAIL / N-A with evidence
([Stripe-on-Supabase holes](https://aicourses.com/ai-for-developers/stripe-payments-nextjs-supabase-security/), [Next.js+Supabase hardening](https://checkvibe.dev/blog/secure-nextjs-supabase-app)):

1. **Service-role key never client-side** - no `NEXT_PUBLIC_` on the service-role
   key, no service-role client imported into a Client Component. (P0 if violated.)
2. **RLS enabled + column-level on every table** - especially tables holding
   entitlement/payment columns; row-level alone lets a user set
   `stripe_account_id`/`plan_tier` on their own row (IDOR). Prefer column
   `REVOKE` so only the webhook writes payment state. (P0/P1.)
3. **Stripe webhook signature verified** - HMAC-SHA256 check on every webhook
   handler; reject unsigned. (P0.)
4. **Webhook, not the success redirect, is the sole writer of payment/entitlement
   state** - the `?success=true` callback must not grant access. (P1.)
5. **Entitlements read fresh server-side, not from a stale JWT claim** - a
   cancelled user still holds an "active" claim until token expiry. (P1.)
6. **Secrets correctly scoped** - safe to `NEXT_PUBLIC_`: Supabase anon key,
   Supabase URL, Stripe publishable key. NEVER public: Stripe secret,
   service-role key, DB URLs, webhook secret, email/AI provider keys. (P0 if leaked.)
7. **Rate limiting** on login, signup, password reset, and any route hitting a
   paid external API. (P2.)

For a net-new stack, build the equivalent top-5 recurring-holes list BEFORE the
fan-out and run it the same way.

## C. Supply-chain hardening (2026 attack classes)

SCA is no longer just "known CVEs." Run `capabilities/supply_chain_guard` (age
gate + OSV), then check each ([Microsoft Shai-Hulud](https://www.microsoft.com/en-us/security/blog/2026/05/28/typosquatted-npm-packages-used-steal-cloud-ci-cd-secrets/), [Microsoft Miasma](https://www.microsoft.com/en-us/security/blog/2026/06/02/preinstall-persistence-inside-red-hat-npm-miasma-credential-stealing-campaign/), [TanStack postmortem](https://tanstack.com/blog/npm-supply-chain-compromise-postmortem)):

1. **No `pull_request_target` workflows** running untrusted PR code with secrets.
   (P0 - exfiltration vector. See `~/.claude/rules/security.md`.)
2. **No shared `actions/cache` in a publish/deploy job** - cache poisoning across
   the fork-base trust boundary. CI and publish workflows in separate files. (P1.)
3. **Dependency `preinstall`/`postinstall` scripts flagged** - the Shai-Hulud /
   Miasma persistence vector. List every dep lifecycle script. (P1/P2.)
4. **Actions pinned to a SHA, not a floating tag**; OIDC token scope minimized.
   (P2.)
5. **Public-prefixed secret next to a CI token = P0.** Wormable token theft
   (npm tokens, GitHub PATs) is the 2026 propagation mechanism.

## D. Net-new stacks
Build the equivalent A+B+C lists for the detected stack (its top-5 known
footguns + its CI footguns) BEFORE the fan-out, and run them the same way.
