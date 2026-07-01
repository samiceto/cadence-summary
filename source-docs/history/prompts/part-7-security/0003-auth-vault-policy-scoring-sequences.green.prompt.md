---
id: 0003
title: Auth Vault Policy Scoring Sequences
stage: green
date: 2026-06-20
surface: agent
model: claude-opus-4-8
feature: part-7-security
branch: main
user: samiullahmalik7474@gmail.com
command: /sp.implement
labels: ["t7.1","t7.2","t7.3","t1.1","t4.2","t4.3","clerk","rbac","policy","consent","dunning"]
links:
  spec: specs/spec-part-7.md
  ticket: null
  adr: null
  pr: null
files:
  - packages/security/* (new: crypto, tokens, consent, policy)
  - packages/domain/src/{scoring,sequences,drafting,send,presets}.ts (new)
  - apps/api/src/common/{tenant-context.middleware,roles.guard,roles.decorator,tenant.decorator}.ts
  - apps/api/src/{invoices,connections}/*.controller.ts + app.module.ts
  - apps/worker/src/workers.ts (scoring/sequence/send wired)
  - packages/config/src/index.ts (CADENCE_ENCRYPTION_KEY) + .env.example
  - tasks/part-{1,4,7,8}-tasks.md
tests:
  - packages/security: 5 passed (crypto round-trip/tamper, policy decisions)
  - packages/domain: 9 passed (scoring 4, drafting 5) + 2 DB-gated skipped
---

## Prompt

"go ahead and complete them" (the recommended critical path: T7.1, T7.2, T4.2, T4.3, T1.1/T7.3)

## Response snapshot

Built the critical-path slice in dependency order:

- **T7.2 token vault** — `@cadence/security/crypto.ts` (AES-256-GCM, IV||tag||ct)
  + `tokens.ts` (encrypt-on-write, decrypt-on-read, revocation). New
  `CADENCE_ENCRYPTION_KEY` config.
- **T7.1 auth + RBAC** — api `TenantContextMiddleware` now verifies the Clerk
  session JWT → userId, resolves the tenant membership/role (partner
  multi-tenant via `x-cadence-tenant`), dev-header fallback off-prod. `RolesGuard`
  + `@Roles` + sample protected controllers (`GET /invoices`,
  `POST /connections/:id/sync`).
- **T1.1 + T7.3 policy & consent gate** — `policy.ts` `evaluateSendPolicy`
  (VIP/escalation approval, frequency cap, SMS quiet-hours defer) and the
  `evaluateOutbound` chokepoint combining consent/suppression + policy;
  `consent.ts` ledger + STOP suppression.
- **T4.2 scoring** — `scoring.ts` transparent `heuristic-v1` (cold-start safe)
  + batch `scoreOpenInvoices`; worker `scoring` queue; API surface.
- **T4.3 sequence engine** — `sequences.ts` durable, idempotent (per-step key),
  resumable state machine; `drafting.ts` template + cheap-LLM polish with
  graceful fallback; gated `send.ts` (consent+policy → dispatch → audit → SMS
  metering). Worker `sequence`+`send` queues wired, with quiet-hours deferral.

Verified: 9 packages build; api/web/worker typecheck; security 5/5 + domain 9/9
unit tests pass (DB-gated tests skip locally, run in CI).

## Outcome

- ✅ Impact: Real auth/RBAC, encrypted token vault, the relationship-safety + consent spine, risk scoring, and the end-to-end dunning loop (score → sequence → draft → gated send) are implemented.
- 🧪 Tests: 14 new unit tests pass; DB integration deferred to CI.
- 📁 Files: new `@cadence/security` package + 5 domain services + api auth/RBAC layer + worker wiring.
- 🔁 Next prompts: T4.4 Stripe pay link + SMS, T4.5 reconciliation/write-back, T3.1 billing, T6.3 reply intent + T6.4 negotiation (OpenAI Agents), T8.3 reconciliation.
- 🧠 Reflection: The send path has exactly one chokepoint (`evaluateOutbound`) so no message can bypass consent/policy — a deliberate safety invariant.

## Evaluation notes (flywheel)

- Failure modes observed: missing direct deps surfaced at build (`@cadence/config` in security, `drizzle-orm` in api/worker); `noUncheckedIndexedAccess` caught an unsafe destructure in policy count query.
- Graders run and results (PASS/FAIL): build PASS; typecheck PASS (all); unit tests PASS (security 5/5, domain 9/9); DB integration DEFERRED to CI.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): create an overdue Stripe test invoice, re-sync, then run scoring + start a sequence end-to-end against Neon to watch a gated email get drafted and "sent" (fake adapter) with full audit trail.
