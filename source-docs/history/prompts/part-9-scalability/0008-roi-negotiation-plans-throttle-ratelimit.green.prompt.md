---
id: 0008
title: ROI Negotiation Plans Throttle RateLimit
stage: green
date: 2026-06-20
surface: agent
model: claude-opus-4-8
feature: part-9-scalability
branch: main
user: samiullahmalik7474@gmail.com
command: /sp.implement
labels: ["t11.3","t6.4","t4.4","t9.3","t9.4","t3.3","roi","negotiation","payment-plans","throttle"]
links:
  spec: specs/spec-part-9.md
  ticket: null
  adr: null
  pr: null
files:
  - packages/metrics/src/roi.ts (projectRoi)
  - packages/domain/src/negotiation.ts (+ test) + db migration 0004 payment_plans
  - packages/adapters/src/resilience.ts (Failover*, RateLimitedLlmAdapter)
  - packages/queue/src/throttle.ts (Redis token bucket) + worker wiring
  - packages/billing/src/usage.ts (reportSmsUsageToStripe)
  - apps/api: onboarding(roi), invoices(plan)
  - tasks/part-{3,4,6,9,11}-tasks.md
tests:
  - packages/metrics: 3 passed (incl. ROI)
  - packages/domain: 23 passed (incl. negotiation 4) + 2 DB-gated skipped
  - all packages: 43 pass / 0 fail / 5 skipped
---

## Prompt

"keep going with the remaining app-code tasks"

## Response snapshot

- **T11.3 ROI calculator** — `projectRoi` (conservative recovered-cash + DSO
  reduction + payback) + `POST /onboarding/roi`.
- **T6.4 negotiation + T4.4 payment plans** — `payment_plans` table (migration
  0004, RLS) + `negotiatePlan`: installment numbers are deterministic +
  policy-clamped; the frontier model ONLY phrases the (tone-safe) message.
  `POST /invoices/:id/plan`.
- **T9.3 throttle/failover** — Redis token-bucket `throttle` (per-tenant/channel
  warmup caps) in the send worker + `FailoverEmail/SmsAdapter`.
- **T9.4 AI rate-limit** — `RateLimitedLlmAdapter` (concurrency + min-interval)
  wraps the worker LLM; batch scoring already in place.
- **T3.3 metered SMS push** — `reportSmsUsageToStripe` (Stripe Billing Meters,
  idempotent identifier). Part 3 now fully complete.

Verified: 11 packages build; all 3 apps typecheck; 43 unit tests pass (negotiation
+ ROI added); migration 0004 applied to Neon.

## Outcome

- ✅ Impact: Negotiation/payment-plans, sending throttle + provider failover, LLM rate limiting, ROI pitch, and metered SMS billing — Part 3 done; scale + AI-cost controls in place.
- 🧪 Tests: 43 unit pass; negotiation plan math + ROI bounds covered.
- 📁 Files: new negotiation + resilience + throttle modules + migration 0004.
- 🔁 Next prompts: T4.1 QB/Xero connectors, T6.1 GBM model, T10.2 e2e/contract tests, T11.2/T11.4 growth, T12.3 platform rollups, T7.5/T8.5/T9.5/T10.5 ops.
- 🧠 Reflection: Negotiation keeps the LLM strictly advisory — it phrases, but deterministic policy code sets every number, so a plan can never breach policy.

## Evaluation notes (flywheel)

- Failure modes observed: Promise-resolve type mismatch in the LLM rate-limiter queue (fixed with Promise<void> + () => resolve()).
- Graders run and results (PASS/FAIL): build PASS; typecheck PASS (all); unit PASS (43/43 run); migration PASS on Neon.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): call POST /invoices/:id/plan on the demo VIP invoice and confirm a policy-clamped 3-installment plan persists + a tone-safe offer is returned.
