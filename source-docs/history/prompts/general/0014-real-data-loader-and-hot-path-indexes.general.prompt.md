---
id: 0014
title: Real-data risk loader + hot-path indexes
stage: green
date: 2026-06-25
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: general
labels: ["t6.1", "t9.5", "ml", "training-data", "indexing", "performance"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - packages/db/drizzle/0008_hot_path_indexes.sql
 - packages/domain/src/ml/data.ts
 - packages/domain/src/ml/data.test.ts
 - packages/domain/scripts/train-real.ts
 - packages/domain/src/index.ts
 - packages/domain/package.json
 - REMAINING.md
tests:
 - "packages/domain test suite: 53 pass / 9 DB-gated skipped / 0 fail (incl. 8 new data.test.ts cases)"
 - "packages/domain typecheck clean"
---

## Prompt

okey start remaining and tell me when need me

## Response snapshot

Worked through the two genuinely code-completable items in the remaining list;
everything else is hard-blocked on the user / third parties / host infra.

- **T9.5 (indexing half)** — migration `0008_hot_path_indexes.sql`: added the
  hot-path indexes the live request paths were missing — `idx_consent_latest`
  `(tenant_id, customer_id, channel, occurred_at DESC)` for the per-send consent
  gate, `idx_invoices_tenant_customer` for the per-customer history aggregation in
  `scoreOpenInvoices`/the risk loader, and `idx_risk_scores_invoice_recent` for
  latest-score reads. All tenant-prefixed (RLS-aligned), `IF NOT EXISTS`.
- **T6.1 (real-data path)** — `ml/data.ts` `loadLabeledHistory` + pure
  `assembleHistory`: non-leaky labelling (resolved invoices only; written-off or
  paid-after-due ⇒ late), survival-style snapshot rows emitted only while an
  invoice was still open (so `days_overdue` is learned without leaking the
  label), and prior-history features computed from earlier-issued invoices only.
  `scripts/train-real.ts` (`train:risk:real`) pools tenants via
  `withPlatformAdmin`, splits by invoice (snapshots of one invoice never straddle
  the split), and **only overwrites `model.json` when it beats the heuristic on a
  disjoint holdout and has ≥200 resolved invoices** — a thin dataset can't regress
  scoring. 8 new unit tests cover labelling, snapshot cutoffs, prior-history, and
  deterministic invoice-level splitting.

Did NOT run `train:risk:real` (it connects to prod Neon, and there's no labelled
history yet). After explicit user go-ahead, applied migrations **0005–0008 to
production Neon** via `db:migrate` (idempotent, transaction-per-file); verified all
four recorded in `_cadence_migrations`. Migrations 0000–0008 now all live.

## Outcome

- ✅ Impact: T6.1 retrain is now a single command once data accrues; T9.5's
  code-completable indexing is done and shipped as 0008.
- 🧪 Tests: domain 53 pass / 9 DB-gated skipped / 0 fail; typecheck clean.
- 📁 Files: new `ml/data.ts` + `ml/data.test.ts` + `scripts/train-real.ts`;
  migration 0008; index/package wiring; REMAINING.md statuses (T6.1, T9.5).
- 🔁 Next (needs user/host): apply migrations 0005–0008 to Neon; T11.1 app-review;
  ops hardening (PITR, dashboards, load test, autoscaling); T11.5 content.
- 🧠 Reflection: the leakage trap here is realised days-late as a feature —
  avoided by labelling on terminal outcome but featurising only "still open at
  offset N", which is exactly what's observable at scoring time.
