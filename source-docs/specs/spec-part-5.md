# Spec — Part 5: Architecture Standards (Cadence)

**Functional Requirements**
- Modular monolith (Next.js + NestJS) + worker tier; connector adapter layer; event-driven sequence engine.
- Managed Neon Postgres + Redis; Clerk auth; Stripe billing; Twilio + email providers behind adapters; LLM providers behind adapter.

**Non-functional Requirements**
- No vendor lock-in at provider layer; 12-factor config; secrets in vault; horizontal scalability of workers.

**User Stories**
- As an engineer, I want swappable connector/comms/LLM adapters so providers can change without rewrites.

**Acceptance Criteria**
- Adding a new accounting connector requires only an adapter implementing the interface.
- Switching email provider requires no domain logic change.

**Technical Constraints**
- Durable, resumable, idempotent sequence steps; RLS multi-tenancy; idempotent accounting writes; webhook ingestion.

**Edge Cases**
- Provider outage → failover/degrade; partial connector capability → capability flags.

**Success Metrics**
- New connector integration < 2 weeks; zero cross-tenant leakage; p95 dashboard < 1.5s.
