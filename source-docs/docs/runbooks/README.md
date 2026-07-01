# Cadence Operations Runbooks

Operational documentation for running Cadence in production. These runbooks
assume the architecture in the repo: a NestJS API (`apps/api`), a BullMQ worker
(`apps/worker`) over Redis, a Next.js dashboard (`apps/web`), and Neon Postgres
with row-level security (RLS) for tenant isolation.

| Runbook | Covers | Task |
|---------|--------|------|
| [incident-response.md](./incident-response.md) | Severity model, on-call flow, triage, comms, common incidents | T7.5 |
| [alerting.md](./alerting.md) | Dashboards (Sentry/PostHog), alert thresholds, on-call owners | T10.5 |
| [disaster-recovery.md](./disaster-recovery.md) | RPO/RTO targets, Neon PITR, restore drill, failover | T8.5 / T10.5 |

## Observability seams

Application code never imports a vendor SDK directly. Two provider-agnostic
seams in `@cadence/observability` carry signals (the SDKs are wired in the app
entrypoints):

- `captureError(err, context)` → Sentry (exceptions). Context is redacted of
  PII before it leaves the process (`redact()`).
- `captureEvent(event, props)` → PostHog (product/finance events; T12.1 taxonomy).
- `logger.{info,warn,error,debug}` → structured JSON logs.

## Service inventory

| Component | Tech | Scaling unit | State |
|-----------|------|--------------|-------|
| API | NestJS / Node 22 | stateless replicas | none (DB + Redis) |
| Worker | BullMQ consumer | replicas per queue depth | none |
| Queues | Redis 7 | managed instance | jobs + DLQ |
| Database | Neon Postgres 16 | managed (autoscaling compute) | **system of record** |
| Dashboard | Next.js | stateless replicas | none |

Queues (see `@cadence/queue`): `sync`, `scoring`, `sequence`, `send`,
`reconcile`. Each job carries a tenant context and runs RLS-scoped via
`withTenant`.
