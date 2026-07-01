# Cadence

Autonomous, relationship-safe accounts-receivable agent for B2B SMBs. Connects
to your accounting system, predicts late invoices, and runs polite-but-persistent
dunning across email + SMS — every send policy-bounded, reversible, and auditable.

> **Status:** Foundation milestone (Part 5 — Architecture + canonical data
> schema) implemented. Connectors, sequence engine, AI, and billing are
> scaffolded as adapter seams / queues and filled in by later parts.

## Monorepo layout

```
apps/
  web/         Next.js dashboard (north-star widget: recovered cash + DSO)
  api/         NestJS API (stateless; tenant-context middleware → RLS)
  worker/      BullMQ worker tier (sync, scoring, sequence, send, reconcile)
packages/
  config/        12-factor, zod-validated env (single source of truth)
  core/          canonical enums, data-classification, Result, errors, TenantContext
  db/            Neon Postgres schema + RLS (SQL migrations) + Drizzle query layer
  observability/ redacting structured logger + Sentry/PostHog telemetry seams
  adapters/      connector / comms / LLM interfaces + impls + in-memory fakes
  queue/         BullMQ infra: queues, retries/backoff, dead-letter queue
```

Mapping to the SDD plan: `specs/`, `plans/`, `tasks/` part-N → the parts above.

## Architecture decisions (foundation)

- **Modular monolith + worker tier** (Part 5): three deployables sharing typed
  packages. Stateless API scales horizontally; workers scale by queue depth.
- **Tenant isolation via Postgres RLS** (Part 5/7): every query runs inside
  `withTenant(ctx, …)`, which sets `app.current_tenant_id`; policies (`FORCE ROW
  LEVEL SECURITY`) deny-by-default. The audit log is immutable (no UPDATE/DELETE
  policy). Proven by `packages/db/src/rls.test.ts`.
- **No provider lock-in** (Part 5): comms/LLM/accounting live behind
  `@cadence/adapters`. Missing credentials fall back to in-memory fakes so the
  app boots and the loop is exercisable without spend.
- **Idempotency everywhere** (Part 8): external IDs + idempotency keys on all
  mutations so connector replays / out-of-order webhooks never double-apply.
- **Secrets/PII never logged** (Part 7): `@cadence/observability` redacts by key
  pattern + the `@cadence/core` data-classification taxonomy.

## Develop

```bash
pnpm install
pnpm build          # build shared packages
pnpm typecheck      # all apps + packages
pnpm test           # unit tests (DB RLS test needs Postgres — see below)

# run a local stack (needs Postgres + Redis reachable via .env)
pnpm --filter @cadence/db db:migrate
pnpm dev            # web :3000, api :4000, worker
```

### Database / RLS test

Migrations are hand-authored SQL in `packages/db/drizzle/*.sql` (authoritative,
RLS-complete) applied by `pnpm db:migrate`. The RLS isolation test runs in CI
against the Postgres service in `.github/workflows/ci.yml`; locally it needs a
reachable `DATABASE_URL` (use a throwaway Postgres, not production Neon).

## Configuration

All config is validated once at boot by `@cadence/config`. See `.env.example`
for the full key list (Clerk, Neon, Redis, Stripe, Twilio, Resend, QuickBooks,
Xero, OpenAI, R2, Sentry, PostHog).
