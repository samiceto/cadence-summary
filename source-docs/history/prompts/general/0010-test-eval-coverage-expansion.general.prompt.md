---
id: 0010
title: Test and eval coverage expansion
stage: general
date: 2026-06-22
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: (none — direct request)
labels: ["tests", "evals", "webhooks", "contract-tests", "e2e"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - packages/domain/src/drafting-eval.test.ts
 - packages/domain/src/e2e.test.ts
 - packages/adapters/src/impls/twilio-sms.test.ts
 - packages/adapters/src/impls/resend-email.test.ts
 - apps/api/src/webhooks/webhook-events.ts
 - apps/api/src/webhooks/webhook-events.test.ts
 - apps/api/src/webhooks/webhooks.controller.ts
 - apps/api/package.json
 - REMAINING.md
tests:
 - packages/domain: 31 pass / 9 DB-gated skipped (adds drafting-eval + e2e)
 - packages/adapters: 29 pass (adds Twilio + Resend transport contract tests)
 - apps/api: 5 pass (new tsx --test runner; webhook routing/parsing contract)
---

## Prompt

yes, take on the test/eval items next

## Response snapshot

Took on the three code-completable test/eval items left in REMAINING.md:

1. **T6.2** — `drafting-eval.test.ts`: golden-set draft-quality contract over
   every preset × context (tone-safe, carries invoice no. + amount, SMS has STOP
   + length budget, email has subject, pay link surfaced); plus a property that
   tone-safety holds for any LLM output including 5 adversarial rewrites.
2. **T10.1** — extracted the webhook controller's pure routing/parsing decisions
   into `webhooks/webhook-events.ts` (`classifyStripeEvent`, `paymentRef`,
   `resendEventType`, `resendRecipient`), refactored the controller to use them,
   and added `webhook-events.test.ts`. Wired a real `tsx --test` runner into the
   api package (was a placeholder echo).
3. **T10.2** — `e2e.test.ts`: DB-gated full onboard→sync→score→sequence→pay→
   reconcile harness against fakes; plus Twilio/Resend transport contract tests
   (`twilio-sms.test.ts`, `resend-email.test.ts`) that stub `fetch` and assert
   request shape + error mapping.

Verified: adapters 29/29, api 5/5, domain 31 pass / 9 DB-gated skipped, 0 fail.
Type-checked the test files explicitly (the package build excludes tests) — the
only diagnostic is the pre-existing `t.skip` typing quirk also present in
`sync.test.ts`, not a real error. Updated REMAINING.md: category 3 (Tests &
evals) fully closed; status 48/61.

## Outcome

- ✅ Impact: closed the entire Tests & evals category (T6.2, T10.1, T10.2, T10.3/T6.5); the proven loop now has an end-to-end regression guard.
- 🧪 Tests: adapters 29 pass; api 5 pass; domain 31 pass / 9 DB-gated skipped; 0 fail.
- 📁 Files: 4 new test files, 1 extracted helper module, controller refactor, api test runner, REMAINING.md.
- 🔁 Next prompts: T11.1 OAuth install flow; then ops execution (Neon PITR drill, dashboards) and the GBM model (T6.1).
- 🧠 Reflection: extracting pure helpers from the Nest controller made webhook logic testable without a Nest/DB harness — small refactor, real coverage.

## Evaluation notes (flywheel)

- Failure modes observed: package builds exclude `*.test.ts`, so DB-gated test branches aren't type-checked by default — caught by an explicit one-off tsc include.
- Graders run and results (PASS/FAIL): draft-quality contract PASS; tone-safety-under-adversarial-LLM PASS; webhook routing/idempotency-key PASS; comms contract PASS.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): add the api/worker test scripts to the root `pnpm test` fan-out and a CI step that type-checks test files.
