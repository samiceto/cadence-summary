---
id: 0006
title: Metrics Dashboard and Onboarding
stage: green
date: 2026-06-20
surface: agent
model: claude-opus-4-8
feature: part-12-success-metrics
branch: main
user: samiullahmalik7474@gmail.com
command: /sp.implement
labels: ["t12.1","t12.2","t12.4","t12.5","t1.2","t4.7","t2.1","t2.2","t2.3","t2.4","metrics","onboarding"]
links:
  spec: specs/spec-part-12.md
  ticket: null
  adr: null
  pr: null
files:
  - packages/metrics/* (new: events, north-star, aging, business, report, lift)
  - packages/domain/src/onboarding.ts + presets.ts (vertical presets)
  - packages/db/drizzle/0003_partner_access.sql + client.ts (withUser)
  - apps/api/src/{metrics,onboarding,tenants}/* + common/auth.ts + app.module.ts
  - apps/web/app/dashboard/page.tsx + components/aging.tsx
  - tasks/part-{1,2,4,12}-tasks.md
tests:
  - packages/metrics: 2 passed (holdout, lift)
  - packages/domain: 16 passed (+ onboarding 3) + 2 DB-gated skipped
  - all packages: 35 pass / 0 fail / 5 skipped
---

## Prompt

"complete them all, first complete coding task" (continuing the remaining tasks)

## Response snapshot

- **Part 12 metrics** — new `@cadence/metrics`: `events.ts` (taxonomy + track),
  `north-star.ts` (recovered cash + DSO, null-for-no-history), `aging.ts`
  (buckets via SQL now()), `business.ts` (per-tenant MRR/ARR), `report.ts`
  (monthly), `lift.ts` (deterministic ~10% holdout + recovery lift). API
  `GET /metrics/{north-star,aging,lift,report}`.
- **T1.2/T4.7 dashboard** — web dashboard fetches north-star + aging; `NorthStar`
  + `Aging` components render reconciled data (placeholders when none).
- **Part 2 onboarding** — `completeOnboarding` + `assessFit` (anti-persona) +
  vertical presets (wholesale/services/default); partner switcher via migration
  0003 (self-scoped membership/tenant RLS) + `withUser` + `GET /me/tenants`.

Verified: 10 packages build; all 3 apps typecheck; 35 unit tests pass.
**Live on Neon:** north-star $4,200 recovered, aging $13,400 bucketed correctly,
lift +33%.

## Outcome

- ✅ Impact: The product now *shows its value* (recovered cash, DSO, aging) and onboards tenants with tailored presets + fit checks + partner switching.
- 🧪 Tests: 35 unit pass; metrics SQL validated live against Neon.
- 📁 Files: new `@cadence/metrics` package + onboarding/partner API + dashboard UI.
- 🔁 Next prompts: T7.4 audit export, T8.5 retention/erasure, T3.5 clawback, T10.4 deliverability auto-pause, T6.5 eval gates, T1.3 CI tone gate, T4.1 QB/Xero, T6.4 negotiation.
- 🧠 Reflection: Partner switching needed cross-tenant reads — solved with a self-scoped RLS policy on `app.current_user_id` rather than weakening isolation.

## Evaluation notes (flywheel)

- Failure modes observed: float-equality test flake (lift); binding a JS Date into a SQL filter failed — switched aging to SQL now().
- Graders run and results (PASS/FAIL): build PASS; typecheck PASS (all); unit PASS (35/35 run); live Neon metrics PASS.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): schedule the monthly report + nightly score/reconcile via cron (repeatable worker triggers).
