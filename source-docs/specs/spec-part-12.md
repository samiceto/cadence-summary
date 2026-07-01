# Spec — Part 12: Success Metrics (Cadence)

**Functional Requirements**
- Instrument + surface north-star (recovered cash, DSO reduction) per tenant.
- Track product KPIs (activation, reply/promise-to-pay/payment-link conversion, AI quality), business KPIs (MRR/ARR, ACV, NRR, margin, CAC payback), reliability KPIs.
- Monthly recovered-cash report per tenant; control-group measurement of collection lift.

**Non-functional Requirements**
- Metrics accuracy from reconciled data; dashboards real-time-ish (< 15 min lag).

**User Stories**
- As a founder, I want a monthly "cash recovered" report proving ROI.
- As the team, we want cohort retention + lift dashboards.

**Acceptance Criteria**
- Dashboard shows DSO trend, recovered cash, aging from first activation.
- Collection lift measured vs holdout/control.

**Technical Constraints**
- PostHog product analytics + warehouse for cohort/finance metrics; attribution tied to audit log.

**Edge Cases**
- Sparse-data tenants → confidence-bounded metrics; seasonality → trailing-window normalization.

**Success Metrics**
- DSO −20–40% in 90 days; MRR $30–70k early cohorts; NRR > 115%; gross margin > 85%; lift > 15%.
