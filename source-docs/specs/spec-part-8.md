# Spec — Part 8: Data Standards (Cadence)

**Functional Requirements**
- Canonical entities (Tenant, Customer, Invoice, Payment, RiskScore, Sequence, Message, Reply, ConsentRecord, AuditEvent, etc.).
- Accounting system as system of record; idempotent write-back via external IDs + idempotency keys.
- Dedupe on sync; nightly reconciliation vs accounting balances with drift alerts.
- Consent ledger + suppression list per channel/contact.

**Non-functional Requirements**
- Reconciliation accuracy > 99.9%; data classified (PII/financial/secret) and access-controlled; retention + erasure policies enforced.

**User Stories**
- As finance, I want balances that always match my accounting system.
- As a DPO, I want erasure-on-request honored with legal-hold exceptions.

**Acceptance Criteria**
- No payment double-applied; drift between Cadence and accounting flagged within 24h.
- Erasure request removes PII (except legal-hold financial records).

**Technical Constraints**
- Idempotency keys on all mutations/write-backs; PITR backups + restore tests; data export/delete per tenant.

**Edge Cases**
- Conflicting edits in accounting vs Cadence → accounting wins; out-of-order webhooks → reconciled by timestamp/version.

**Success Metrics**
- Reconciliation accuracy > 99.9%; 100% mutations carry idempotency keys; successful restore tests.
