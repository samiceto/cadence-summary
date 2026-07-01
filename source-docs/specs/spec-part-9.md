# Spec — Part 9: Scalability Rules (Cadence)

**Functional Requirements**
- Stateless API + horizontally scalable BullMQ workers (scale by queue depth).
- Incremental/delta sync (cursors + webhooks) + nightly full reconciliation.
- Per-tenant sending identities with warmup-aware throttling; provider failover.
- Batch nightly AI scoring; queued + rate-limited LLM calls; graceful template fallback.

**Non-functional Requirements**
- Dashboard p95 < 1.5s; step dispatch within 1 min of schedule; sync freshness < 15 min; no message loss/duplication.

**User Stories**
- As a growing tenant, I want performance to hold as my invoice volume scales.

**Acceptance Criteria**
- Worker tier scales under load with steps dispatched on time.
- Provider rate limits respected without dropped/duplicated sends.

**Technical Constraints**
- Retries w/ backoff + DLQ; idempotent handlers; circuit breakers; indexes on tenant_id/status/due_date; archival of old data.

**Edge Cases**
- Spike in overdue invoices (month-end) → backpressure, no overload; provider outage → failover/degrade.

**Success Metrics**
- Meets latency/freshness targets at 10× baseline volume; 0 lost/duplicate messages.
