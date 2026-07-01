# Plan — Part 6: AI Strategy (Cadence)

**Objectives:** Build cost-routed AI: prediction, drafting, intent classification, negotiation — all tone-safe.

**Architecture Decisions:** GBM/small model for scoring (batched); cheap model + templates for drafting; cheap model for intent; frontier only on live negotiation; LLM adapter + queue + cache; eval harness in CI.

**Dependencies:** Data/features (Part 8), sequence engine (Part 4), policy engine (Part 1/10).

**Milestones:** M1 feature pipeline + scoring; M2 drafting w/ templates + tone policy; M3 intent classifier; M4 negotiation reasoning; M5 eval suite + gates; M6 fallback/degradation.

**Timeline:** ~2.5 weeks (overlaps Part 4).

**Risks:** tone violations; cost overrun; cold-start accuracy.

**Mitigation:** tone-safety gate 100%; cheap-model routing + caching; segment priors for cold start.

**Resource Estimates:** 1 ML/BE + support, ~2.5 weeks.
