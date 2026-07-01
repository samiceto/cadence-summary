# Plan — Part 10: Quality Standards (Cadence)

**Objectives:** Comprehensive testing + AI evals + deliverability + reliability gates.

**Architecture Decisions:** Test pyramid (unit/integration/e2e/contract/load); AI golden-set eval harness in CI; deliverability auto-pause; observability stack (PostHog/Sentry/logs).

**Dependencies:** All build parts; AI (Part 6); comms (Part 4).

**Milestones:** M1 unit+integration; M2 e2e + contract tests; M3 AI eval gates; M4 deliverability monitor/auto-pause; M5 dashboards + alerting + DR drill.

**Timeline:** ~2 weeks (continuous, front-loaded weeks 2–4).

**Risks:** untested edge cases; eval gaps; deliverability decay.

**Mitigation:** relationship-safety + reconciliation as release gates; golden sets; auto-pause thresholds.

**Resource Estimates:** shared across team + 0.5 QA, ongoing.
