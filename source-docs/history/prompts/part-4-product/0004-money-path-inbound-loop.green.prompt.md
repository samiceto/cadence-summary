---
id: 0004
title: Money Path and Inbound Loop
stage: green
date: 2026-06-20
surface: agent
model: claude-opus-4-8
feature: part-4-product
branch: main
user: samiullahmalik7474@gmail.com
command: /sp.implement
labels: ["t4.4","t4.5","t4.6","t8.2","t8.3","t6.3","payments","reconciliation","reply-intent","approvals"]
links:
  spec: specs/spec-part-4.md
  ticket: null
  adr: null
  pr: null
files:
  - packages/db/drizzle/0002_paylink.sql + schema.ts (invoices.pay_url)
  - packages/adapters/src/connector.ts + impls/stripe-connector.ts (payUrl mapping)
  - packages/domain/src/payments.ts (applyPayment, reconcileInvoices)
  - packages/domain/src/replies.ts (+ replies.test.ts) (classifyReply, handleReply)
  - packages/domain/src/{send,sequences,sync}.ts (approved opt, pay link)
  - packages/queue/src/queues.ts (SendJob.approved)
  - apps/worker/src/workers.ts (reconcile wired; send approved)
  - apps/api/src/messages/* + customers/* (+ app.module) (approve, blocklist)
  - packages/domain/scripts/{live-demo,verify-money}.ts
  - tasks/part-{4,6,8}-tasks.md
tests:
  - packages/domain: 13 passed (incl. replies 4) + 2 DB-gated skipped
  - live validation on Neon: payment idempotency + reply→suppress
---

## Prompt

"continue with remaining tasks"

## Response snapshot

Built the money path + inbound loop:

- **T4.4** — Stripe `hosted_invoice_url` captured at sync (`pay_url`, migration
  0002) and embedded in drafts; SMS channel sends through the gated path.
- **T4.5/T8.2** — `applyPayment` (idempotent on payment external id; updates
  balance/status; closes paid sequences; idempotent connector write-back) +
  `reconcileInvoices` (T8.3: local-vs-accounting drift → audit + Sentry alert).
- **T6.3** — `classifyReply` (deterministic STOP/dispute/PTP + cheap-LLM for
  ambiguous, safe fallback) + `handleReply` (suppress on STOP, pause on
  dispute/PTP/question).
- **T4.6** — approval workflow (`GET /messages/pending-approval`,
  `POST /messages/:id/approve` → re-queue with `approved`) + `POST
  /customers/:id/blocklist` (suppress all channels); `approved` threaded through
  the send path.

Verified: 8 packages build; api/worker typecheck; domain 13/13 unit tests
(2 DB-gated skipped). **Live on Neon:** partial→`partially_paid`,
replay→`duplicate` (no double-apply), full→`paid`/0 balance, "STOP"→`unsubscribe`
(0.99)→suppression — all audited.

## Outcome

- ✅ Impact: The cash loop closes (pay → apply → reconcile) and inbound replies route safely (STOP/dispute) — the agent now reacts, not just sends.
- 🧪 Tests: 13 unit + live Neon validation of payments idempotency & reply suppression.
- 📁 Files: new `payments.ts`/`replies.ts` domain services + approval/blocklist API + pay-link plumbing + migration 0002.
- 🔁 Next prompts: T3.1 billing (Stripe tiers/checkout), T6.4 negotiation (OpenAI Agents), T9.1 delta sync + webhooks (real inbound payment/reply events), T12.1/T12.2 metrics + north-star data, T2.1 onboarding.
- 🧠 Reflection: applyPayment derives idempotency from the payment's external id so webhook replays are inherently safe — verified against the real DB, not just unit mocks.

## Evaluation notes (flywheel)

- Failure modes observed: left an unused `sql` import in payments.ts (removed); ensured the frequency-cap counts only dispatched messages (fixed earlier) so applied here cleanly.
- Graders run and results (PASS/FAIL): build PASS; typecheck PASS; unit PASS (13/13); live Neon validation PASS (idempotency + suppression).
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): add webhook endpoints (Stripe payment_intent.succeeded → applyPayment; inbound email/SMS → handleReply) so the loop runs on real events (T9.1).
