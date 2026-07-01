---
id: 0009
title: Complete remaining code-completable tracks
stage: general
date: 2026-06-22
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: (none — direct request)
labels: ["connectors", "onboarding", "tests", "evals", "ops", "runbooks"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - packages/adapters/src/impls/qbo-connector.ts
 - packages/adapters/src/impls/xero-connector.ts
 - packages/adapters/src/impls/qbo-connector.test.ts
 - packages/adapters/src/impls/xero-connector.test.ts
 - packages/adapters/src/factory.ts
 - packages/adapters/src/index.ts
 - apps/web/app/onboarding/page.tsx
 - apps/web/components/onboarding-wizard.tsx
 - apps/web/app/page.tsx
 - packages/domain/src/intent-eval.test.ts
 - packages/domain/src/negotiation-eval.test.ts
 - packages/domain/src/payments.test.ts
 - .github/workflows/ci.yml
 - docs/runbooks/README.md
 - docs/runbooks/incident-response.md
 - docs/runbooks/alerting.md
 - docs/runbooks/disaster-recovery.md
 - REMAINING.md
tests:
 - packages/adapters: 20 pass (Stripe + QBO + Xero mapper contract tests)
 - packages/domain: 28 pass / 8 DB-gated skipped (intent-eval, negotiation-eval, payments integration)
 - apps/api + apps/worker: typecheck clean
 - apps/web: typecheck clean
---

## Prompt

check remaining.md file and complete the project

## Response snapshot

REMAINING.md listed 18 open items spanning external integrations, ML, tests,
ops, and growth — much of it not code-completable here (app-store review, ML
training data, deploy-host config, GTM). Confirmed scope with the user, who
selected four code tracks: QBO/Xero adapters, onboarding wizard UI, ops
runbooks, and tests/evals. Delivered all four:

1. **T4.1** — `QboConnector` + `XeroConnector` (`ConnectorAdapter`, fetch-based,
   no new SDK deps): cursor/page sync of customers + invoices, status
   normalization, idempotent write-back. Extended `makeConnector` to take
   `string | ConnectorCredentials` (realmId/tenantId) without breaking Stripe
   callers. Pure-mapper contract tests for both.
2. **Onboarding wizard UI** — multi-step client wizard (Business → ROI → Review
   → Done) driving the existing assess/roi/complete endpoints; linked from home.
3. **Tests/evals** — intent-accuracy golden-set gate (≥90%, hermetic),
   negotiation-policy fuzz gate (2000 randomized inputs), DB-gated
   payment-application + reconciliation integration tests.
4. **Ops** — incident-response / alerting / disaster-recovery runbooks under
   `docs/runbooks/`; added `pnpm audit` + dependency-review to CI.

Updated REMAINING.md to mark T4.1, T10.3/T6.5, T7.5, and the onboarding wizard
done; T10.1/T10.2/T8.5/T10.5 moved to partial with the precise remainder noted.

## Outcome

- ✅ Impact: 4 of the 5 fully-code-completable remaining tracks landed; live-go
  items (OAuth install/app review, host wiring, ML data) flagged as out-of-band.
- 🧪 Tests: adapters 20 pass; domain 28 pass / 8 DB-gated skipped; api/worker/web typecheck clean.
- 📁 Files: 2 new connectors + tests, wizard page+component, 3 eval/integration test files, 4 runbooks, CI scanning, REMAINING.md.
- 🔁 Next prompts: T11.1 OAuth install flow (unblocks QBO/Xero live); T10.2 full e2e harness.
- 🧠 Reflection: keeping connectors fetch-based and exposing pure mappers kept the diff small and the tests hermetic/CI-friendly.

## Evaluation notes (flywheel)

- Failure modes observed: none in CI-runnable tests; DB-gated tests unverifiable locally by design (prod Neon).
- Graders run and results (PASS/FAIL): intent-accuracy gate PASS (≥90%); negotiation-policy invariants PASS (2000 cases); tone-safety gate PASS.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): add Twilio/Resend contract tests and the DB-gated onboard→pay→reconcile e2e.
