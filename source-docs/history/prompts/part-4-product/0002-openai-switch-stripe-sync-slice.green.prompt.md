---
id: 0002
title: OpenAI Switch and Stripe Sync Slice
stage: green
date: 2026-06-20
surface: agent
model: claude-opus-4-8
feature: part-4-product
branch: main
user: samiullahmalik7474@gmail.com
command: /sp.implement
labels: ["openai","stripe","connector","sync","t4.1","idempotency","rls"]
links:
  spec: specs/spec-part-4.md
  ticket: null
  adr: null
  pr: null
files:
  - packages/adapters/src/impls/openai-llm.ts (new; replaces anthropic-llm.ts)
  - packages/adapters/src/impls/stripe-connector.ts (real Stripe SDK impl + mapping)
  - packages/adapters/src/impls/stripe-connector.test.ts (4 mapping contract tests)
  - packages/adapters/src/{factory,fakes,index}.ts (OpenAI + makeConnector + FakeConnector)
  - packages/config/src/index.ts (OpenAI model defaults; ANTHROPIC removed)
  - packages/domain/* (new package: syncConnection + idempotent/version-guarded upsert)
  - apps/worker/src/workers.ts (sync queue wired to syncConnection)
  - .github/workflows/ci.yml (CADENCE_TEST_DB=1 gate)
  - packages/db/src/rls.test.ts (DB-gated guard)
  - .env.example, README.md, tasks/part-4-tasks.md, tasks/part-5-tasks.md
tests:
  - packages/adapters: 7 passed (3 fakes + 4 Stripe mapping)
  - packages/domain: sync.test.ts (2, DB-gated → CI)
  - packages/db: rls.test.ts (3, DB-gated → CI)
---

## Prompt

1. "Migrate my Neon DB now" + "First vertical slice (T4.1)" (AskUserQuestion answers)
2. "i removed anthropic varealbe use openaikey only and openai agents"

## Response snapshot

- **Neon migration:** Applied 0000_init + 0001_rls to the real Neon DB. Verified
  17 tables, 16 RLS policies, FORCE RLS on scoped tables (users correctly global).
- **OpenAI switch:** Replaced the Anthropic adapter with `OpenAiLlmAdapter`
  (official `openai` SDK, tier→model routing gpt-4o-mini/gpt-4o); factory now
  keys off OPENAI_API_KEY; config defaults + .env.example updated; Anthropic refs
  removed. Noted the Agents SDK layers on this key for Part 6 negotiation.
- **T4.1 slice:** Real `StripeConnector` (cursor pagination, status normalization
  incl. overdue derivation, idempotent write-back). New `@cadence/domain` package
  with `syncConnection()` — tenant-scoped upserts keyed on (tenant,system,
  external_id), version-guarded invoice updates (stale pages can't clobber).
  Worker `sync` queue wired end-to-end. `FakeConnector` added for tests.
- **Test safety:** DB-gated integration tests (RLS, sync) run only when
  CADENCE_TEST_DB=1 (CI), so `pnpm test` on a dev machine never writes to prod Neon.

Verified: 8 packages build; api/web/worker typecheck; adapters 7/7 pass; DB-gated
suites skip cleanly locally (5 skipped, 0 fail).

## Outcome

- ✅ Impact: Core data loop (connect → sync → persist) implemented for Stripe; LLM provider switched to OpenAI.
- 🧪 Tests: 7 adapter unit tests pass; 5 DB-gated integration tests deferred to CI.
- 📁 Files: new `@cadence/domain` package + Stripe/OpenAI impls + worker wiring + CI gate.
- 🔁 Next prompts: QuickBooks/Xero adapters, T4.2 risk scoring on synced invoices, T4.3 sequence engine, T7.2 per-tenant token vault (replace platform Stripe key in sync).
- 🧠 Reflection: Gated DB tests behind an explicit flag after realizing local DATABASE_URL points at prod Neon — prevents accidental test-data pollution.

## Evaluation notes (flywheel)

- Failure modes observed: shell glob expansion dropped a top-level test file (`src/**/*.test.ts`) — fixed by quoting so Node globs recursively.
- Graders run and results (PASS/FAIL): build PASS; typecheck PASS (all apps+packages); adapters tests PASS (7/7); DB integration DEFERRED to CI.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): run the sync against a throwaway Postgres (or a Stripe test-mode key into Neon) to confirm end-to-end import before building T4.2 on top.
