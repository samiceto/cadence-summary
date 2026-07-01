# Plan — Part 9: Scalability Rules (Cadence)

**Objectives:** Scale workers + comms + AI without loss/duplication; meet latency/freshness targets.

**Architecture Decisions:** Queue-depth autoscaling; delta sync (cursors+webhooks); per-tenant sending identities + throttling + failover; batched AI + rate-limited LLM; DLQ + circuit breakers; indexing/archival.

**Dependencies:** Architecture (Part 5), AI (Part 6), comms providers (Part 4).

**Milestones:** M1 delta sync + webhooks; M2 worker autoscaling + DLQ; M3 sending throttle + failover; M4 AI batching/rate-limit; M5 load tests + indexes/archival.

**Timeline:** ~2 weeks (overlaps + post-MVP hardening).

**Risks:** month-end spikes; provider rate limits.

**Mitigation:** backpressure; idempotent handlers; load test at 10×.

**Resource Estimates:** 1 BE, ~2 weeks.
