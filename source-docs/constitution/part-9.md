# Constitution — Part 9: Scalability Rules (Cadence)

**Scaling model:** stateless API + horizontally scalable worker tier (BullMQ); managed Neon Postgres + Redis; scale workers by queue depth.

**Sequence engine:** durable state machines; steps scheduled via delayed jobs; resumable + idempotent; one in-flight step per invoice; backpressure on provider rate limits.

**Sync at scale:** incremental/delta sync with cursors + webhooks (not full pulls); batched; per-connection rate-limit awareness; nightly full reconciliation.

**Comms throughput:** per-tenant sending identities; warmup-aware throttling; provider failover (email Resend↔Postmark; SMS Twilio); respect carrier/ISP limits.

**AI scaling:** batch nightly risk scoring; cache templates/embeddings; frontier model only on live negotiation; queue + rate-limit LLM calls; degrade gracefully to templates if LLM unavailable.

**Data scaling:** indexed on tenant_id + status + due_date; partition/archival of old invoices/messages; read replicas for dashboards (post-MVP).

**Performance targets:** dashboard p95 < 1.5s; sequence step dispatch within 1 min of schedule; sync freshness < 15 min via webhooks.

**Reliability:** retries w/ exponential backoff + DLQ; idempotent handlers; circuit breakers on providers; no message loss/duplication.

**Cost-aware scaling:** serverless/managed first; scale-to-low off-hours; avoid premature sharding.

**Consistency check:** delivers NFRs (specs), preserves margin (Part 3), supports reliability (Part 10).
