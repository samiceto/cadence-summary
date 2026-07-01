# Plan — Part 7: Security Requirements (Cadence)

**Objectives:** Tenant isolation, credential protection, consent enforcement, full audit.

**Architecture Decisions:** Postgres RLS everywhere; vault/KMS token encryption; consent ledger + suppression checks in send path; immutable audit log; RBAC.

**Dependencies:** Auth (Clerk), DB (Part 5), data model (Part 8).

**Milestones:** M1 RLS + RBAC; M2 token vault + scopes/revocation; M3 consent + suppression gate; M4 audit log + export; M5 scanning + IR runbook.

**Timeline:** ~2 weeks (overlaps Parts 4/8).

**Risks:** cross-tenant leak; credential exposure; non-compliant sends.

**Mitigation:** RLS tests; secret scanning; consent checks as hard gate; pen test pre-GA.

**Resource Estimates:** 1 BE (security focus), ~2 weeks; SOC 2 roadmap ongoing.
