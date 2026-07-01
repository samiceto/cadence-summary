---
id: 0011
title: Fix two environment warnings
stage: green
date: 2026-06-25
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: general
labels: ["env", "db-safety", "openai", "ci"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - packages/db/src/client.ts
 - packages/db/src/migrate.ts
 - .env
 - .env.example
 - REMAINING.md
tests:
 - pnpm --filter @cadence/db typecheck (pass)
 - pnpm --filter @cadence/db test (3 DB-gated tests skip locally, as designed)
 - tsx guard smoke test (4/4 scenarios pass)
 - curl OpenAI /v1/models + gpt-4o-mini completion (HTTP 200)
---

## Prompt

fix inverenment two warnings

## Response snapshot

Addressed the two environment warnings called out in REMAINING.md:

1. **OPENAI_API_KEY 401** — verified the key now authenticates (HTTP 200 on
   `/v1/models` and a live `gpt-4o-mini` chat completion). Warning was stale;
   real LLM drafting works. Updated REMAINING.md.

2. **DATABASE_URL points at production Neon** — the only guard was the
   `CADENCE_TEST_DB` flag, with `getDb()` silently falling back to the prod
   `DATABASE_URL`. Added a central, host-level refusal:
   - `assertNotProtectedDb()` throws when `CADENCE_TEST_DB=1` and the connection
     string targets `CADENCE_PROTECTED_DB_HOST` (set to the prod Neon endpoint
     `ep-restless-forest-atfn265a` in `.env`).
   - `resolveConnectionString()` prefers a throwaway `CADENCE_TEST_DATABASE_URL`
     under tests so they never lean on the app DATABASE_URL.
   - Wired the guard into both `getDb()` (client.ts) and the migration runner
     (migrate.ts). Documented both vars in `.env.example`.
   - CI is unaffected: no protected host configured, localhost throwaway DB.

## Outcome

- ✅ Impact: An accidental local `CADENCE_TEST_DB=1` now hard-errors instead of
  writing to production Neon; OpenAI drafting confirmed working.
- 🧪 Tests: db typecheck pass; db test suite skips DB-gated tests locally as
  designed; guard smoke test 4/4; OpenAI 200.
- 📁 Files: client.ts, migrate.ts, .env, .env.example, REMAINING.md.
- 🔁 Next prompts: optionally provision a real throwaway local Postgres and run
  the DB-gated suites with CADENCE_TEST_DATABASE_URL set.
- 🧠 Reflection: the prod-host denylist is a belt-and-suspenders layer on top of
  the existing CADENCE_TEST_DB gate — defense in depth without changing CI.

## Evaluation notes (flywheel)

- Failure modes observed: none; one warning (OpenAI 401) was already stale.
- Graders run and results (PASS/FAIL): guard smoke test PASS (4/4); typecheck PASS.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): add a unit test asserting
  assertNotProtectedDb throws on the protected host under CADENCE_TEST_DB=1.
