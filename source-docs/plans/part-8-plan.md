# Plan — Part 8: Data Standards (Cadence)

**Objectives:** Canonical model, idempotent write-back, reconciliation, consent/suppression, retention.

**Architecture Decisions:** Accounting = system of record; external IDs + idempotency keys on all mutations; nightly reconciliation + drift alerts; data classification + retention/erasure jobs.

**Dependencies:** Connectors (Part 4), security (Part 7).

**Milestones:** M1 schema + classification; M2 idempotent write-back; M3 reconciliation + drift; M4 consent ledger/suppression; M5 retention/erasure + backups/restore tests.

**Timeline:** ~2 weeks (overlaps Part 4).

**Risks:** double-applied payments; data drift; erasure vs legal hold conflict.

**Mitigation:** idempotency keys; reconciliation gate; legal-hold rules in erasure flow.

**Resource Estimates:** 1 BE, ~2 weeks.
