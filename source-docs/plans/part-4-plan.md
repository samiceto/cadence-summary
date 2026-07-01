# Plan — Part 4: Product Requirements (Cadence)

**Objectives:** Ship core MVP: sync → score → sequence → pay → reconcile, with policy controls + dashboard.

**Architecture Decisions:** Connector adapter layer (QuickBooks/Xero/Stripe); BullMQ durable state-machine sequences; webhook-driven status; Stripe pay links; reconciliation engine with idempotent write-back.

**Dependencies:** Architecture (Part 5), AI (Part 6), data (Part 8), security (Part 7).

**Milestones:** M1 connectors + sync; M2 risk scoring; M3 sequence engine (email); M4 SMS + payment plans + pay link; M5 cash application/reconciliation; M6 policy/approval + blocklist; M7 dashboard.

**Timeline:** ~4 weeks (critical path, core of 7–9 wk MVP).

**Risks:** connector edge cases; reconciliation correctness; deliverability.

**Mitigation:** contract tests per provider; reconciliation as release gate; SPF/DKIM/DMARC + warmup.

**Resource Estimates:** 2 BE + 1 FE, ~4 weeks.
