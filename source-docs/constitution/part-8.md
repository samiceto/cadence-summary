# Constitution — Part 8: Data Standards (Cadence)

**Core entities:** Tenant, User, Connection (accounting/Stripe), Customer, Invoice, Payment, RiskScore, Sequence, SequenceStep, Message, Reply, Approval, PaymentPlan, ConsentRecord, AuditEvent.

**Sources of truth:** accounting system is system of record for invoices/payments; Cadence mirrors + annotates (risk, sequence state, comms). Writes back idempotently (external IDs + idempotency keys).

**Data classification:** PII (contact names, emails, phones), financial (invoice/payment amounts), credentials (OAuth tokens — secret). Each classified and access-controlled.

**Quality:** dedupe customers/invoices on sync; reconcile balances vs accounting nightly; flag drift; never double-apply payments.

**Consent & suppression:** consent ledger per channel/contact; suppression list for bounces/complaints/unsubscribes/STOP; checked before send.

**Retention:** invoices/comms retained per tenant policy (default 7y financial records); PII deletion on request (right to erasure) with legal-hold exceptions; tokens deleted on disconnect.

**Privacy:** data minimization; no selling data; no training shared models on tenant data without explicit opt-in; per-tenant data export + delete (portability).

**Lineage & idempotency:** every sync + write-back carries external ref + idempotency key; full audit of mutations.

**Backups:** automated Neon backups; PITR; periodic restore tests.

**Consistency check:** supports reconciliation (Part 4), security (Part 7), metrics integrity (Part 12).
