# Plan — Part 12: Success Metrics (Cadence)

**Objectives:** Instrument north-star + KPIs; monthly recovered-cash reporting; lift measurement.

**Architecture Decisions:** PostHog product analytics + warehouse for finance/cohort metrics; control/holdout for lift; metrics from reconciled data tied to audit log.

**Dependencies:** Reconciliation (Part 8), billing (Part 3), AI (Part 6).

**Milestones:** M1 event taxonomy + instrumentation; M2 north-star dashboard; M3 business/cohort dashboards; M4 monthly tenant report; M5 lift/control measurement.

**Timeline:** ~1.5 weeks (overlaps).

**Risks:** metric mistrust; attribution error; sparse-data noise.

**Mitigation:** reconcile from accounting truth; audit-linked attribution; confidence bounds.

**Resource Estimates:** 1 BE/analyst, ~1.5 weeks.
