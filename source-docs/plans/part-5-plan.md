# Plan — Part 5: Architecture Standards (Cadence)

**Objectives:** Stand up modular monolith + worker tier + adapter layers with no lock-in.

**Architecture Decisions:** Next.js + NestJS monolith; BullMQ workers; Neon + Redis; adapters for connectors/comms/LLM; RLS multi-tenancy; 12-factor + vault.

**Dependencies:** Hosting (Render/Fly), Cloudflare/R2, Clerk, Stripe.

**Milestones:** M1 repo + CI/CD + envs; M2 DB schema + RLS; M3 worker tier + queue; M4 adapter interfaces; M5 observability baseline.

**Timeline:** ~1.5 weeks (front-loaded, week 1).

**Risks:** premature complexity; lock-in.

**Mitigation:** monolith first; provider behind adapters; managed services.

**Resource Estimates:** 1–2 BE, ~1.5 weeks.
