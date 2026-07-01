# Tasks — Part 5: Architecture Standards (Cadence)

> **Status (2026-06-20): Foundation milestone implemented.** Monorepo + all
> shared packages + 3 apps build & typecheck; unit tests green. RLS test is
> CI-gated (Postgres service). See root `README.md`.

- **[x] T5.1 Repo + CI/CD + envs** — Monorepo (Next.js+NestJS), pipelines, dev/staging/prod. Priority: P0. Estimate: 1.5d. Dependencies: none. DoD: deploy to staging via CI. → pnpm+Turborepo workspace, `.github/workflows/ci.yml`, zod-validated `@cadence/config`.
- **[x] T5.2 DB schema + RLS** — Neon schema with tenant RLS on all tables. Priority: P0. Estimate: 2d. Dependencies: T5.1. DoD: RLS tests prove isolation. → `@cadence/db` SQL migrations (0000_init, 0001_rls), `withTenant()`, `rls.test.ts`.
- **[x] T5.3 Worker tier + queue** — BullMQ + Redis worker infrastructure. Priority: P0. Estimate: 1.5d. Dependencies: T5.1. DoD: jobs scheduled/processed with retries. → `@cadence/queue` (backoff retries + DLQ), `apps/worker`.
- **[x] T5.4 Adapter interfaces** — Connector/comms/LLM adapter contracts. Priority: P0. Estimate: 1.5d. Dependencies: T5.1. DoD: interfaces + 1 impl each. → `@cadence/adapters` (Twilio/Resend/OpenAI/Stripe impls + fakes + factory).
- **[x] T5.5 Observability baseline** — Sentry + PostHog + structured logs. Priority: P1. Estimate: 1d. Dependencies: T5.1. DoD: errors + events captured. → `@cadence/observability` (redacting logger + telemetry seams).
