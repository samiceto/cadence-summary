# Spec — Part 10: Quality Standards (Cadence)

**Functional Requirements**
- Test suites: unit (state machine, reconciliation, consent), integration (connectors, webhooks, write-back), e2e (onboard→sync→sequence→pay→reconcile), contract tests per provider, load tests.
- AI golden-set evals in CI: tone-safety, intent accuracy, draft quality, negotiation policy adherence.
- Deliverability monitoring with auto-pause on threshold breach.

**Non-functional Requirements**
- 99.9% uptime; RPO ≤ 15 min, RTO ≤ 4h; relationship-safety + financial-correctness as release gates.

**User Stories**
- As a customer, I trust that updates never introduce tone violations or payment errors.

**Acceptance Criteria**
- CI blocks release on failing tone-safety, intent, or reconciliation tests.
- Sends to blocklist/VIP/post-consent-withdrawal = P0, blocked pre-prod.

**Technical Constraints**
- PostHog + Sentry + structured logs; per-tenant/per-sequence dashboards; alerting on queue depth/deliverability/drift.

**Edge Cases**
- Deliverability degradation → auto-pause domain + alert; eval threshold regression → block deploy.

**Success Metrics**
- Tone violations = 0; intent accuracy > 90%; reconciliation > 99.9%; uptime 99.9%.
