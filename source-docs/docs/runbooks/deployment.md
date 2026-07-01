# Runbook — Deployment & Scaling (Render)

Covers first-time setup, routine deploys, autoscaling (T9.2), and rollback for
the Cadence API + worker on Render, with Neon (Postgres) and Render Key Value
(Redis) as backing services.

## Two deploy profiles

| Profile | Blueprint | Services | Cost | Use |
|---------|-----------|----------|------|-----|
| **Portfolio / free** *(active)* | [`render.yaml`](../../render.yaml) | api (**workers embedded**) + web + redis, free plans, no autoscaling | ~$0/mo | demo, idles when unused |
| **Production** | [`render.production.yaml`](../../render.production.yaml) | api + **separate worker** + web + redis, Standard plans, autoscaling | ~$120/mo | GA |

The active blueprint Render auto-detects is **`render.yaml`** — currently the
**free** profile. The only code difference is one env flag:
**`RUN_WORKERS_IN_API=true`** makes the API host the BullMQ workers in-process
(see `apps/api/src/main.ts`), so the free profile needs no separate paid worker.
Production leaves the flag off and runs `cadence-worker` as its own service. To
switch to production, swap the filenames:
```bash
git mv render.yaml render.portfolio.yaml && git mv render.production.yaml render.yaml
```

## Topology (production)

| Service | Render type | Scales on | Health |
|---------|-------------|-----------|--------|
| `cadence-api` | web | CPU/mem 70% (native) | `GET /health` |
| `cadence-worker` | worker | queue depth (see below) + CPU | n/a |
| `cadence-web` | web | CPU/mem 70% | (port check) |
| `cadence-redis` | redis (Key Value) | — | `noeviction` required |
| Postgres | external (Neon) | — | PITR on (see disaster-recovery.md) |

All declared in [`render.yaml`](../../render.yaml).

## First-time setup

1. **Create the Blueprint.** Render → New + → Blueprint → select this repo. It
   reads `render.yaml` and provisions all three services + the `cadence-shared`
   env group.
2. **Fill secrets.** Every `sync: false` var in the env group prompts on first
   deploy. Paste the production values (Neon URLs, Clerk, Stripe, Twilio, Resend,
   OpenAI, OAuth client IDs/secrets, `CADENCE_ENCRYPTION_KEY`, Sentry/PostHog).
   - `DATABASE_URL` = Neon **pooled**; `DATABASE_URL_UNPOOLED` = Neon **direct**
     (migrations need the direct session).
   - `ACCOUNTING_OAUTH_REDIRECT_URL` must match the callback registered in the
     Intuit/Xero apps (T11.1), e.g. `https://app.cadence.app/api/connectors/callback`.
3. **Run migrations** (once, against Neon — separate from the app deploy):
   ```bash
   pnpm --filter @cadence/db db:migrate    # idempotent; applies 0000–0008
   ```
4. **Verify.** `curl https://<api>.onrender.com/health` → `{"status":"ok"}`.

## Routine deploy

`autoDeploy: true` ships every push to `main`. Render builds the monorepo
(`pnpm install --frozen-lockfile && pnpm build`) and rolls instances one at a
time; with `minInstances: 2` the API never drops to zero.

**Migrations + deploy ordering:** the migration runner is additive and
idempotent, so run `db:migrate` **before** the app deploy when a release adds
schema. Never ship code that reads a column the live DB lacks.

## Autoscaling (T9.2)

- **API** — native Render autoscaling on CPU/memory (`scaling:` block in
  `render.yaml`): 2→10 instances at 70% targets.
- **Worker** — dunning load is I/O-bound (waiting on Stripe/Twilio/Resend/LLM),
  so CPU lags the real backlog. Scale on **BullMQ queue depth** with the
  [`queue-autoscaler`](../../scripts/queue-autoscaler.ts) cron job:
  1. Render → New + → Cron Job, schedule `* * * * *` (every minute).
  2. Command: `node --import tsx scripts/queue-autoscaler.ts`
  3. Env: `REDIS_URL` (from the shared group), `RENDER_API_KEY`,
     `RENDER_WORKER_SERVICE_ID` (the `srv-…` id of cadence-worker). Optional:
     `QAS_MIN`, `QAS_MAX`, `QAS_JOBS_PER_INSTANCE` (default 500).
  - Scales **up immediately** under backlog, **down one step/min** (hysteresis)
    so a brief lull can't drop capacity mid-burst.
  - Keep the worker's `render.yaml` `maxInstances` ≥ `QAS_MAX`.

## Rollback

1. Render → cadence-api → **Events/Deploys** → pick the last-good deploy →
   **Rollback**. Repeat for cadence-worker.
2. If the bad release ran a migration: migrations here are **additive only**
   (new tables/columns/indexes, never destructive), so a code rollback is safe
   without a DB rollback. If a future migration is destructive, it must ship its
   own documented down-step — do not assume rollback safety.
3. Confirm `/health` and a smoke check of `/metrics/north-star` with valid auth.

## Capacity validation

Before a GA / marketing push, run the load test in
[`load-test/`](../../load-test/README.md) against the staging URL and confirm the
p95 thresholds hold and the query plans use the 0008 hot-path indexes.
