# Constitution — Part 5: Architecture Standards (Cadence)

**Principles:** modular monolith first (Next.js + NestJS) + worker tier; event-driven sequences; stateless services + managed state; provider/connector adapters; no lock-in.

**Reference stack:** Next.js + React + TS + Tailwind + shadcn/ui; NestJS API; Neon Postgres + Redis; Clerk auth; Stripe billing + pay links; BullMQ for sequence scheduling; Twilio (SMS) + email (Resend/Postmark) for comms; Cloudflare + R2; Render/Fly; PostHog + Sentry; Anthropic + OpenAI behind adapter, cheap/open models for drafting.

**Standards:**
- Sequences modeled as durable state machines (resumable, idempotent steps).
- All accounting writes idempotent + reconciled; webhooks for inbound payment/status.
- Multi-tenant with Postgres RLS; per-tenant comms identity (sending domains/numbers).
- 12-factor config; secrets in vault; least-privilege accounting scopes.

**Consistency check:** enables scalability (Part 9), security (Part 7), cost (Part 3).
