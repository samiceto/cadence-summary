# Plan — Part 3: Business Model (Cadence)

**Objectives:** Tiered Stripe billing + metered SMS + opt-in success fee with auditable attribution.

**Architecture Decisions:** Stripe products/prices per tier; metered usage events (idempotent); success-fee engine reading reconciled recoveries + audit log.

**Dependencies:** Stripe (Part 5), reconciliation + audit (Parts 4/7/8), metrics (Part 12).

**Milestones:** M1 tiers + checkout; M2 limit enforcement + upgrade prompts; M3 metered SMS; M4 success-fee calc + statement; M5 clawback/refund handling.

**Timeline:** ~2 weeks.

**Risks:** success-fee disputes; metering errors; margin erosion.

**Mitigation:** transparent auditable statements; idempotent metering; COGS monitoring < 12%.

**Resource Estimates:** 1 BE + part-time FE, ~2 weeks.
