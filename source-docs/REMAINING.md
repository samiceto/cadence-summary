# Cadence — Remaining Work

**Last updated:** 2026-06-26
**Status:** core + all code-completable tracks done (incl. deploy config,
autoscaling, monitoring-as-code + live telemetry wiring, load-test harness, DR
drill script, marketplace kit, content) · remaining is external app-review,
ML-with-data, and host execution (deploy/run-the-scripts) only

**2026-06-26 update — ops/infra/growth artifacts (deploy=Render, monitoring=Sentry+PostHog):**
- **T9.2 done (code)** — `render.yaml` Blueprint (api+worker+Redis, native
  autoscaling) + `scripts/queue-autoscaler.ts` (queue-depth scaling via Render API)
  + `docs/runbooks/deployment.md`.
- **T10.5 (code)** — `monitoring/` as-code (alerts.yaml + Sentry/PostHog rules &
  apply scripts), the **T12.1 event taxonomy wired** end-to-end, a worker
  queue-depth heartbeat, and **telemetry sinks actually connected** to Sentry +
  PostHog (`observability/providers.ts initTelemetry`, called in api+worker boot).
- **T9.5** — k6 load-test harness (`load-test/`).
- **T8.5** — automated restore drill (`scripts/dr-restore-drill.sh`).
- **T11.1** — submission kit (`docs/integrations/marketplace-submission.md`).
- **T11.5 done (draft)** — DSO/AR content + benchmarks (`content/`).

**2026-06-25 update — non-Docker code tracks landed (T9.3, T12.3, T11.1, T11.2, T11.4, T6.1):**
- **T9.3 done (code)** — per-tenant sending identities: `sending_identities`
  table (RLS, migration 0005) + verified-only resolver (`@cadence/domain
  sending-identity.ts`, prefers tenant identity over platform default) wired into
  `sendMessage` for email + SMS; `SendingIdentitiesController` (create/list/verify).
  Remaining: provider-side domain/number provisioning is host config.
- **T12.3 done (code)** — platform MRR/ARR/NRR + cohort rollups: read-only
  cross-tenant `withPlatformAdmin` GUC + policies (migration 0006) and pure
  rollups (`@cadence/metrics platform.ts`). Remaining: an HTTP/admin surface needs
  a platform-admin identity (RBAC has only tenant roles).
- **T11.1 done (code)** — QBO/Xero OAuth install flow: provider-agnostic
  authorize/exchange/refresh helpers (`@cadence/adapters oauth.ts`),
  HMAC-signed state (`@cadence/security oauth-state.ts`), authorize endpoint +
  Next→API callback that stores encrypted tokens via the vault, and worker wiring
  that decrypts + refreshes per-tenant tokens. Remaining (external): Intuit/Xero
  app-store listings + app-review submission.
- **T11.2 done** — partner portal: referral columns (migration 0007), pure
  attribution rollup (`@cadence/metrics partner.ts`), `GET /me/portal`, and the
  portal UI with a cookie-based client switcher wired into the dashboard.
- **T11.4 done** — freemium + activation funnel: gating + read-only insights
  (`@cadence/metrics freemium.ts`), funnel rollups (`funnel.ts`),
  `FreemiumController` (insights + activation-intent) emitting funnel events, and
  the free-insights page with an upgrade CTA.
- **T6.1 pipeline done (synthetic)** — pure-TS gradient-boosted trees
  (`@cadence/domain/ml/*`) + a `RiskScorer` that drops in behind
  `computeRiskScore`; the worker loads the trained `ml/model.json` (beats the
  heuristic on holdout) and falls back to the heuristic when absent. Remaining:
  retrain on real labelled payment history.

**2026-06-22 update (2) — test/eval category fully closed:**
- **T6.2 done** — broader draft-quality eval (`drafting-eval.test.ts`).
- **T10.1 done** — webhook routing/parsing contract tests; pure helpers
  extracted from the webhook controller (`webhooks/webhook-events.ts`). The API
  package now has a real test runner (`tsx --test`).
- **T10.2 done** — DB-gated end-to-end loop harness (`e2e.test.ts`) + Twilio &
  Resend transport contract tests. All of category 3 (Tests & evals) is now done.


**2026-06-22 update — code-completable tracks landed:**
- **T4.1 done** — QuickBooks Online + Xero `ConnectorAdapter` implementations
  (`packages/adapters/src/impls/{qbo,xero}-connector.ts`), wired into
  `makeConnector` and covered by pure-mapper contract tests. Per-tenant OAuth
  token wiring in the worker still waits on the install flow (T11.1).
- **Onboarding wizard UI done** — multi-step wizard at `apps/web/app/onboarding`
  driving the existing assess → roi → complete backend.
- **Tests/evals broadened** — intent-accuracy golden-set gate (≥90%, hermetic),
  negotiation-policy fuzz gate, DB-gated payment-application + reconciliation
  integration tests, QBO/Xero contract tests.
- **Ops docs + scanning** — incident-response, alerting, and DR runbooks under
  `docs/runbooks/`; dependency scanning (`pnpm audit` + dependency-review) added
  to CI alongside the existing gitleaks secret scan.

The **core agent loop is complete and validated live on Neon + Stripe**:
connect → sync → score → sequence → tone-safe draft → consent/policy-gated send →
reply handling + negotiation → payment application → reconciliation → metrics/
dashboard. Plus auth/RBAC, encrypted token vault, Stripe billing (checkout/tiers/
success-fee/metered/refunds), webhooks, onboarding, audit/erasure, deliverability
auto-pause, throttle/failover, LLM rate-limiting.

Per-task status lives in `tasks/part-N-tasks.md` — `[x]` done, `[~]` partial, no
marker = not started. This file is the consolidated "what's left".

---

## Legend
- 🔴 **not started**
- 🟡 **partial** (core done; noted remainder)
- Category tells you *why* it isn't just-write-code.

---

## Remaining by category

### 1. External integrations (need third-party OAuth / app review)
| Task | Status | What's needed |
|------|--------|---------------|
| **T4.1** QuickBooks/Xero connectors | ✅ done | `QboConnector` + `XeroConnector` implement `ConnectorAdapter` (sync + idempotent write-back) in `@cadence/adapters`, wired into `makeConnector`. Remaining to go *live*: QBO/Xero developer apps + the OAuth install flow that decrypts the per-tenant token and stores realmId/tenantId (that's T11.1). |
| **T11.1** Marketplace listings + install | 🟡 | **OAuth install flow done (code)**: authorize/exchange/refresh helpers (`@cadence/adapters oauth.ts`), signed-state CSRF (`@cadence/security oauth-state.ts`), authorize endpoint + Next→API callback storing encrypted tokens, worker decrypt+refresh wiring. Remaining (external, not code): QuickBooks/Xero/Stripe app-store listings + app-review submission — **submission kit + checklist + listing copy at `docs/integrations/marketplace-submission.md`** (scopes, redirect URIs, security-questionnaire answers, demo-video script). The disconnect/revoke endpoint Intuit's review requires is now **implemented**: `DELETE /connections/:id` + provider-side `revokeToken` (QBO/Xero), tokens cleared + status→revoked, audited. |

### 2. Machine learning (needs training data + pipeline)
| Task | Status | What's needed |
|------|--------|---------------|
| **T6.1** GBM late-payment model | 🟡 | **Pipeline + real-data loader done**: pure-TS gradient-boosted trees (`@cadence/domain/ml/{gbm,features,synthetic,scorer,train}.ts`) with a `RiskScorer` that drops in behind `computeRiskScore` (worker loads the trained model, falls back to heuristic). Synthetic artifact (`ml/model.json`) beats the heuristic on holdout (AUC 0.756 vs 0.726, logLoss 0.55 vs 1.04). **Real-data path now coded**: `ml/data.ts` `loadLabeledHistory` (non-leaky labelling — resolved invoices, paid-after-due/written-off ⇒ late; snapshot rows only while still-open so `days_overdue` is learned without leakage; prior-history features from earlier-issued invoices only) + `scripts/train-real.ts` (`pnpm --filter @cadence/domain train:risk:real`) which pools all tenants via `withPlatformAdmin`, splits by invoice, and **only overwrites `model.json` if it beats the heuristic on a disjoint holdout and has ≥200 resolved invoices** — so a thin dataset can't regress scoring. Remaining: run it once real payment history has accrued. |

### 3. Tests & evals (expand coverage)
| Task | Status | What's needed |
|------|--------|---------------|
| **T10.1** Unit + integration suites | ✅ done | DB-gated payment-application + reconciliation integration tests (`payments.test.ts`) + webhook routing/parsing contract tests (`apps/api/src/webhooks/webhook-events.test.ts`, with helpers extracted for testability). |
| **T10.2** e2e + contract tests | ✅ done | DB-gated full onboard→sync→score→sequence→pay→reconcile harness (`e2e.test.ts`); per-provider contract tests for Stripe/QBO/Xero mapping + Twilio/Resend transport (request shape + error mapping, `{twilio-sms,resend-email}.test.ts`). |
| **T10.3 / T6.5** AI eval gates | ✅ done | Tone-safety, intent-accuracy golden set (≥90%, `intent-eval.test.ts`), and negotiation-policy fuzz gate (`negotiation-eval.test.ts`) all gate CI. |
| **T6.2** Drafting tone eval | ✅ done | Broader golden-set draft-quality eval (`drafting-eval.test.ts`): facts/channel/pay-link contract across every preset × context, plus tone-safety holds for any (incl. adversarial) LLM output. |

### 4. Ops / infra (deploy & runbooks, not app code)
| Task | Status | What's needed |
|------|--------|---------------|
| **T7.5** Scanning + IR runbook | ✅ done | gitleaks + `pnpm audit` + dependency-review now in CI; incident-response runbook at `docs/runbooks/incident-response.md`. |
| **T8.5** Retention/erasure + backups | 🟡 | `eraseCustomer` (PII redaction, legal hold) done; DR/PITR procedure + **automated restore-drill script** (`scripts/dr-restore-drill.sh` — PITR branch, integrity queries, RTO timing, teardown) + runbook (`docs/runbooks/disaster-recovery.md`). Remaining (host): set Neon PITR retention + run the drill (`NEON_PROJECT_ID=… bash scripts/dr-restore-drill.sh`). |
| **T9.2** Worker autoscaling | ✅ done (code) | DLQ + backoff done; **autoscaling as-code**: `render.yaml` native CPU/mem scaling for the API + a **queue-depth autoscaler** (`scripts/queue-autoscaler.ts`, scales the worker on BullMQ backlog via Render's API, with hysteresis) + `docs/runbooks/deployment.md`. Remaining: deploy the Blueprint + run the autoscaler as a Render cron (host). |
| **T9.3** Per-tenant sending identities | ✅ done (code) | Throttle + failover done; `sending_identities` table (RLS, mig 0005) + verified-only `from` resolver wired into `sendMessage` (email + SMS), `SendingIdentitiesController`. Remaining: provider-side domain/number provisioning (host config). |
| **T9.5** Load test + indexing/archival | 🟡 | **Indexing pass done** (migration 0008: `idx_consent_latest`, `idx_invoices_tenant_customer`, `idx_risk_scores_invoice_recent`) + **k6 load-test harness** (`load-test/api-load.js` steady+spike scenarios with SLO thresholds; `load-test/README.md`). Remaining (host/Docker): run it against staging, confirm plans use the indexes, define the data-archival policy. |
| **T10.5** Ops dashboards + alerting + DR drill | 🟡 | **Monitoring-as-code done**: canonical `monitoring/alerts.yaml`, Sentry rules + apply script, PostHog dashboards/insights + apply script (`monitoring/`), the full **T12.1 event taxonomy wired** (onboarding/connector/sequence/payment/send/reply/queue-depth heartbeat) and the **telemetry sinks connected** to Sentry + PostHog in app bootstrap (`observability/providers.ts` `initTelemetry`). Remaining (host): run the two apply scripts with API tokens + attach Slack/PagerDuty channels + run the DR drill. |

### 5. Growth / content / frontend
| Task | Status | What's needed |
|------|--------|---------------|
| **T11.2** Partner portal + attribution | ✅ done | Referral columns (mig 0007), pure attribution rollup (`@cadence/metrics partner.ts`), `GET /me/portal`, and the portal UI with a cookie-based client switcher wired into the dashboard. |
| **T11.4** Freemium + activation funnel | ✅ done | Gating + read-only insights (`@cadence/metrics freemium.ts`), funnel rollups (`funnel.ts`), `FreemiumController` (insights + activation-intent) emitting funnel events, and the free-insights page with an upgrade CTA. |
| **T11.5** Content/SEO engine | ✅ done (draft) | Cornerstone content drafted under `content/`: "What is DSO & how to reduce it", "AR collections automation guide", and a "DSO benchmarks by industry" linkable asset (front-matter + internal links ready for the web app/CMS). Remaining (GTM): publish, OG images, sitemap, refresh benchmarks from first-party cohort data once it accrues. |
| **Onboarding wizard UI** | ✅ done | Multi-step wizard (`apps/web/app/onboarding` + `components/onboarding-wizard.tsx`) drives assess → ROI → complete; shows fit warning + projected recovered cash + applied preset. |

### 6. Platform analytics (warehouse)
| Task | Status | What's needed |
|------|--------|---------------|
| **T12.3** Business/cohort dashboards | ✅ done (code) | Per-tenant MRR/ARR done; platform-wide MRR/ARR/NRR + cohort rollups via read-only `withPlatformAdmin` GUC + policies (mig 0006) and pure rollups (`@cadence/metrics platform.ts`). Remaining: an HTTP/admin surface needs a platform-admin identity (RBAC has only tenant roles). |

---

## How to resume (quick start)
```bash
pnpm install
pnpm -r --filter "./packages/*" build   # build shared packages
pnpm --filter @cadence/api --filter @cadence/worker --filter @cadence/web typecheck
pnpm -r --filter "./packages/*" test      # all green; DB-gated tests skip locally (need CADENCE_TEST_DB=1 + throwaway Postgres)
pnpm --filter @cadence/api --filter @cadence/adapters --filter @cadence/domain test
```
Test landmarks: domain 53 pass / 9 DB-gated skipped (incl. `e2e.test.ts`,
`payments.test.ts`, `ml/ml.test.ts`, `ml/data.test.ts`); adapters 38 pass
(Stripe/QBO/Xero + Twilio/Resend + OAuth authorize/refresh/revoke contracts); security 11 pass (vault +
OAuth state); metrics 21 pass (incl. platform/partner/freemium rollups); api 5 pass.
- Risk model: `pnpm --filter @cadence/domain train:risk` retrains on synthetic data;
  `train:risk:real` retrains on real DB history (guards: ≥200 invoices, must beat
  heuristic on a disjoint holdout, else it leaves `model.json` untouched).
- DB migrations: `pnpm --filter @cadence/db db:migrate` — **0000–0008 all applied to
  Neon** (0005 sending-identities, 0006 platform-admin, 0007 referrals, 0008
  hot-path indexes applied 2026-06-25).
- ✅ Local `DATABASE_URL` still points at **production Neon**, but DB tests can no
  longer hit it: `getDb`/`migrate` hard-refuse any connection to
  `CADENCE_PROTECTED_DB_HOST` when `CADENCE_TEST_DB=1`, and prefer a throwaway
  `CADENCE_TEST_DATABASE_URL` when set (`packages/db/src/client.ts`). CI is
  unaffected (no protected host; localhost throwaway DB).
- ✅ `OPENAI_API_KEY` in `.env` now authenticates (verified 200 on
  `/v1/models` + a live `gpt-4o-mini` completion) — real LLM drafting works.
- Live demos: `packages/domain/scripts/{live-demo,verify-money}.ts`.

## Suggested next order
All code-completable tracks are done. What remains is **running** the artifacts
that now exist, plus genuinely external steps:
1. ✅ **Migrations 0005–0008 applied to Neon** (2026-06-25).
2. **Deploy** — push the `render.yaml` Blueprint, fill secrets, add the
   queue-autoscaler cron (`docs/runbooks/deployment.md`).
3. **Wire monitoring** — run `monitoring/{sentry,posthog}/apply-*.sh` with API
   tokens; set `NEXT_PUBLIC_SENTRY_DSN` / `NEXT_PUBLIC_POSTHOG_KEY` so
   `initTelemetry` lights up; attach Slack/PagerDuty.
4. **Run the drills/tests** — `scripts/dr-restore-drill.sh` (T8.5) and the k6
   `load-test/` against staging (T9.5); confirm plans use the 0008 indexes.
5. **T11.1** app-store listings + Intuit/Xero submission (external) — kit at
   `docs/integrations/marketplace-submission.md`; do the disconnect-endpoint
   follow-up it flags.
6. **T6.1** run `train:risk:real` once real labelled payment history has accrued.
7. **T11.5** publish the `content/` drafts (GTM).
