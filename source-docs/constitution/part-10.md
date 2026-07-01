# Constitution — Part 10: Quality Standards (Cadence)

**Engineering quality:** TypeScript strict; lint/format gates; PR review; trunk-based + CI; feature flags for risky changes.

**Testing:** unit (sequence state machine, reconciliation, consent checks); integration (connectors, webhooks, payment write-back); e2e (onboarding → sync → sequence → payment → reconcile); contract tests for accounting providers; load tests on worker tier.

**AI quality:** golden-set evals in CI for (a) tone-safety (never threatening/non-compliant), (b) reply intent classification accuracy, (c) draft quality, (d) negotiation policy adherence. Ship gated on thresholds; track collection lift vs control.

**Relationship-safety as a quality gate:** zero tolerance for sends to blocklist/VIP without approval, post-consent-withdrawal sends, or tone violations — these are P0 incidents.

**Financial correctness:** reconciliation accuracy is a release gate; never double-apply or misallocate payments; balance drift alerts.

**Deliverability quality:** monitor bounce/complaint/unsubscribe rates; auto-pause sending domain on threshold breach.

**Observability:** PostHog (product), Sentry (errors), structured logs, per-sequence + per-tenant dashboards; alerting on queue depth, deliverability, reconciliation drift.

**Reliability targets:** 99.9% API uptime; RPO ≤ 15 min, RTO ≤ 4 h; no duplicate/lost messages.

**Definition of Done:** tested + evaluated + documented + observable + policy-compliant + reversible.

**Consistency check:** enforces product musts (Part 4), AI guardrails (Part 6), security (Part 7).
