# Constitution — Part 7: Security Requirements (Cadence)

**Principles:** least privilege, defense in depth, tenant isolation, auditability.

**Identity & access:** Clerk auth; SSO (post-MVP) for Pro; RBAC (Owner, Finance, Viewer, Approver); per-action authorization checks server-side.

**Tenant isolation:** Postgres RLS on every tenant table; no cross-tenant queries; per-tenant comms identity (sending domain/number) and accounting credentials scoped separately.

**Accounting credentials:** OAuth tokens (QuickBooks/Xero/Stripe) stored encrypted (KMS/vault), least-privilege scopes, rotated; never logged; revocation honored immediately.

**Secrets:** vault/KMS; no secrets in code/env files in repo; short-lived tokens for workers.

**Data in transit/at rest:** TLS 1.2+; AES-256 at rest (Neon/R2); field-level encryption for tokens and PII contact data.

**Comms abuse prevention:** rate limits, blocklist enforcement, consent ledger checks before every SMS, suppression of bounced/complained addresses.

**Audit:** immutable audit log of every agent action, message sent, policy change, approval, accounting write-back; exportable.

**App security:** OWASP ASVS baseline; input validation; CSRF/XSS protections; dependency scanning (Dependabot); secret scanning in CI.

**Incident response:** Sentry alerting; documented IR runbook; breach notification process; backups + tested restore.

**Compliance posture:** SOC 2 Type II roadmap (Pro/enterprise trust); GDPR/UK-GDPR DPA; TCPA/consent for SMS; CAN-SPAM for email.

**Consistency check:** protects data (Part 8), enables trust-led sales (Part 11), upholds quality (Part 10).
