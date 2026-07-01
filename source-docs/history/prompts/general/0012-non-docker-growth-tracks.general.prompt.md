---
id: 0012
title: Complete non-Docker code tracks
stage: green
date: 2026-06-25
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: general
labels: ["t9.3", "t12.3", "t11.1", "t11.2", "t11.4", "oauth", "freemium", "partner-portal"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - packages/db/drizzle/0005_sending_identities.sql
 - packages/db/drizzle/0006_platform_admin.sql
 - packages/db/drizzle/0007_referrals.sql
 - packages/db/src/schema.ts
 - packages/db/src/client.ts
 - packages/domain/src/sending-identity.ts
 - packages/domain/src/send.ts
 - packages/metrics/src/platform.ts
 - packages/metrics/src/partner.ts
 - packages/metrics/src/funnel.ts
 - packages/metrics/src/freemium.ts
 - packages/adapters/src/oauth.ts
 - packages/security/src/oauth-state.ts
 - apps/api/src/connectors/connectors.controller.ts
 - apps/api/src/connectors/connector-callback.controller.ts
 - apps/api/src/sending-identities/sending-identities.controller.ts
 - apps/api/src/freemium/freemium.controller.ts
 - apps/api/src/tenants/tenants.controller.ts
 - apps/worker/src/connector-credentials.ts
 - apps/web/app/partners/page.tsx
 - apps/web/app/insights/page.tsx
 - apps/web/app/api/connectors/callback/route.ts
 - apps/web/app/api/freemium/activate-intent/route.ts
tests:
 - packages/domain/src/sending-identity.test.ts (5 pass)
 - packages/metrics/src/platform.test.ts (7 pass)
 - packages/metrics/src/partner.test.ts (3 pass)
 - packages/metrics/src/freemium.test.ts (8 pass)
 - packages/adapters/src/oauth.test.ts (6 pass)
 - packages/security/src/oauth-state.test.ts (6 pass)
 - "full sweep: domain 36 pass/9 DB-gated skipped, adapters 35, security 11, metrics 21; api/worker/web typecheck clean"
---

## Prompt

complete the task that do not required docker and heavy infra

## Response snapshot

Completed all five non-Docker, code-completable tracks (chosen by the user via a
scoping question), one at a time with tests:

- **T9.3 — per-tenant sending identities**: `sending_identities` table (RLS, mig
  0005) + verified-only `resolveFrom` resolver wired into `sendMessage` for email
  + SMS (prefers tenant identity, never sends from an unverified domain),
  `SendingIdentitiesController` (create/list/verify).
- **T12.3 — platform rollups**: read-only cross-tenant `withPlatformAdmin` GUC +
  policies (mig 0006); pure `rollupMrr` / `computeNrr` / `cohortRetention`
  (`@cadence/metrics platform.ts`).
- **T11.1 — OAuth install flow**: provider-agnostic authorize/exchange/refresh
  helpers (`@cadence/adapters oauth.ts`), HMAC-signed state CSRF
  (`@cadence/security oauth-state.ts`), authorize endpoint + Next→API callback
  storing encrypted tokens via the vault, and worker wiring that decrypts +
  refreshes per-tenant QBO/Xero tokens. (External app review remains.)
- **T11.2 — partner portal**: referral columns (mig 0007), pure attribution
  rollup, `GET /me/portal`, portal UI + cookie-based client switcher wired into
  the dashboard.
- **T11.4 — freemium funnel**: gating + read-only insights, funnel rollups,
  `FreemiumController` (insights + activation-intent) emitting funnel events, and
  the free-insights page with an upgrade CTA.

## Outcome

- ✅ Impact: 5 tracks landed; remaining work is now external app-review, ML
  (needs data), and host/ops execution only.
- 🧪 Tests: 35 new unit/contract tests, all green; full sweep clean; all apps +
  packages typecheck.
- 📁 Files: 3 migrations, schema + client changes, 4 new metrics modules, OAuth
  helper + signed state, 4 API controllers, worker credential wiring, 2 web pages
  + 2 web route handlers.
- 🔁 Next prompts: T11.1 app-store submission; T6.1 GBM model; ops execution
  (PITR/DR drill, dashboards, T9.5 load test).
- 🧠 Reflection: kept each feature's core logic pure + unit-tested; RLS-sensitive
  cross-tenant reads went through a dedicated read-only GUC rather than weakening
  isolation.

## Evaluation notes (flywheel)

- Failure modes observed: worker missing `@cadence/security` dep (added);
  `next lint` (deprecated) injected `allowJs` into web tsconfig (reverted).
- Graders run and results (PASS/FAIL): all new test suites PASS; typecheck PASS.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): add a platform-admin identity so the
  T12.3 rollups can be exposed over an admin HTTP endpoint.

📋 Architectural decision detected: read-only platform-admin RLS bypass (GUC +
policies) for cross-tenant analytics, and HMAC-signed OAuth state carrying tenant
through the provider redirect — Document reasoning and tradeoffs? Run
`/sp.adr platform-admin-and-oauth-state`
