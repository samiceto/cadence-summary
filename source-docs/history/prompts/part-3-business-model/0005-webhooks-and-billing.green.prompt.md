---
id: 0005
title: Webhooks and Billing
stage: green
date: 2026-06-20
surface: agent
model: claude-opus-4-8
feature: part-3-business-model
branch: main
user: samiullahmalik7474@gmail.com
command: /sp.implement
labels: ["t9.1","webhooks","t3.1","t3.2","t3.3","t3.4","billing","stripe","subscriptions"]
links:
  spec: specs/spec-part-3.md
  ticket: null
  adr: null
  pr: null
files:
  - packages/domain/src/ingest.ts (shared upsert helpers) + sync.ts refactor
  - apps/api/src/webhooks/webhooks.controller.ts (Stripe + Twilio) + main.ts rawBody
  - packages/billing/* (new: tiers, checkout, subscription, limits, usage, success-fee)
  - apps/api/src/billing/billing.controller.ts + app.module.ts
  - tasks/part-3-tasks.md, tasks/part-9-tasks.md
tests:
  - packages/billing: 3 passed (tier gating)
  - full sweep: 30 unit pass, 2 DB-gated skipped; all apps typecheck
---

## Prompt

"keep going with webhooks then billing"

## Response snapshot

- **T9.1 webhooks** — signature-verified `POST /webhooks/stripe/:tenantId`
  (invoice events → `ingestInvoice`; `invoice.paid` → `applyPayment`;
  `customer.subscription.*` → subscription sync) + `POST /webhooks/twilio/:tenantId`
  (inbound SMS → `handleReply`). Enabled NestFactory `rawBody` for Stripe
  signature checks. Extracted `@cadence/domain/ingest.ts` upsert helpers shared
  by full sync + webhooks (refactored `sync.ts` onto them — no behavior change).
- **Part 3 billing** — new `@cadence/billing`: `tiers.ts` (TIER_LIMITS + price
  mapping + SMS channel gating), `checkout.ts` (`createCheckoutSession`),
  `subscription.ts` (Stripe→`subscriptions`/`tenants.tier`), `limits.ts`
  (`checkActiveInvoiceLimit`/`isChannelAllowed`), `usage.ts` (metered SMS
  summary), `success-fee.ts` (opt-in fee on >90d recoveries, itemized).
  API: `POST /billing/checkout`, `GET /billing/status`.

Verified: 9 packages build; all 3 apps typecheck; 30 unit tests pass (billing 3,
security 5, adapters 7, observability 2, domain 13) + 2 DB-gated skipped.

## Outcome

- ✅ Impact: The loop is now event-driven (real Stripe/inbound events) and the product is monetizable (subscriptions, tier gating, success fee).
- 🧪 Tests: 30 unit pass; webhook signature path + billing logic covered by typecheck + tier tests.
- 📁 Files: new `@cadence/billing` package + webhook controller + shared ingest helpers.
- 🔁 Next prompts: T3.3 finish (Stripe metered push), T3.5 clawback, T12.1/T12.2 metrics + north-star data, T4.7 dashboard, T2.1 onboarding, T10.x test/eval gates.
- 🧠 Reflection: Extracting ingest helpers means full-sync and webhook paths can never drift in idempotency/version-guard behavior — one implementation, two triggers.

## Evaluation notes (flywheel)

- Failure modes observed: recurring `noUncheckedIndexedAccess` destructure issue in count queries (fixed in limits/usage); missing direct deps (`@cadence/adapters`, `@cadence/billing`) surfaced at api typecheck.
- Graders run and results (PASS/FAIL): build PASS; typecheck PASS (all); unit PASS (30/30 run, 2 skipped).
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): drive a Stripe test-mode checkout + `customer.subscription.created` webhook against Neon to confirm tier lands on the tenant, then exercise a tier limit soft-block.
