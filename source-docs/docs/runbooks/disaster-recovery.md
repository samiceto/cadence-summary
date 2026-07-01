# Disaster Recovery & Backups (T8.5 / T10.5)

Recovery targets and the procedures that meet them. The database is the only
stateful system of record; the API, worker, and dashboard are stateless and
recover by redeploying. Accounting (QuickBooks/Xero/Stripe) is the upstream
source of truth, so a full data loss is recoverable by re-syncing — but we do
not rely on that as the primary strategy.

## 1. Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| **RPO** (max data loss) | ≤ 5 min | Neon PITR is continuous; receivables/audit data must not be lost |
| **RTO** (time to restore) | ≤ 1 hour | DB restore + stateless redeploy |
| Backup retention | ≥ 30 days PITR window | Covers late-discovered corruption + erasure audit |
| Drill cadence | Quarterly restore drill | An untested backup is not a backup |

## 2. What is backed up

| Data | Mechanism | Notes |
|------|-----------|-------|
| Postgres (all tenant data) | **Neon PITR** (continuous WAL) | Restore to any point in the retention window |
| Encrypted OAuth tokens | In Postgres (`connections`, encrypted at rest, T7.2) | Restored with the DB |
| Object storage (R2) | R2 versioning / lifecycle | If/when used for attachments |
| Secrets / config | Secret manager (out of band) | Never in the DB or repo |
| Redis (queues) | **Not** backed up | Transient; jobs are idempotent and re-enqueueable |

Redis is intentionally disposable: every job carries an idempotency key and the
core operations (sync upsert, payment application, sends) are idempotent, so a
queue wipe causes redundant work, never incorrect work.

## 3. Procedures

### 3a. Point-in-time restore (Neon)
1. Declare the incident; apply send kill switch (see incident-response §5) so no
   sends fire against stale/restored state mid-recovery.
2. In Neon, create a branch restored to the target timestamp (just before the
   corruption/loss).
3. Validate the branch: row counts on `tenants`, `invoices`, `payments`,
   `audit_events`; spot-check a known tenant.
4. Repoint `DATABASE_URL` / `DATABASE_URL_UNPOOLED` to the restored branch and
   redeploy API + worker.
5. Re-run reconciliation (`reconcile` queue) per connection to realign local
   balances with accounting and surface any residual drift.
6. Lift the kill switch; resume sends.

### 3b. Region / provider failover
- Stateless tiers (API, worker, web) redeploy from image in an alternate region.
- Neon: promote a read replica / restore branch in the target region.
- DNS/ingress cut over once health checks pass.

### 3c. Full rebuild from accounting (last resort)
If PITR is unavailable, re-provision the schema (`db:migrate`) and re-sync each
connection. This recovers invoices/customers/payments but **not** Cadence-only
state (sequence history, message log, audit trail) — hence PITR is primary.

## 4. Quarterly restore drill (checklist)

> Run against a throwaway Neon branch; never against production traffic.
>
> **Automated:** `NEON_PROJECT_ID=… bash scripts/dr-restore-drill.sh` creates the
> PITR branch, runs the integrity queries, measures RTO, and tears the branch
> down. The manual checklist below is the superset to verify/extend it against.

- [ ] Pick a random recent timestamp; create a PITR branch.
- [ ] Record **time-to-restore** (start → app healthy on restored branch).
- [ ] Validate integrity: row counts within tolerance; a known tenant's
      invoices/payments reconcile; RLS still isolates tenants
      (cross-tenant query returns nothing — see `rls.test.ts`).
- [ ] Run the `reconcile` queue; confirm drift is within tolerance.
- [ ] Confirm measured RPO ≤ 5 min and RTO ≤ 1 h; file action items for any
      miss.
- [ ] Tear down the branch; log the drill result and date.

## 5. Dependencies & escalation

- Neon outage → follow provider status; PITR/branching unavailable means
  failover (3b) or wait. Keep the provider support path in the on-call doc.
- The drill owner is the on-call eng-lead; results are reviewed at the quarterly
  ops review alongside the alert-threshold tuning ([alerting.md](./alerting.md)).
