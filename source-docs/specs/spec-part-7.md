# Spec — Part 7: Security Requirements (Cadence)

**Functional Requirements**
- Clerk auth + RBAC (Owner/Finance/Viewer/Approver); server-side authorization on every action.
- Encrypted storage of OAuth tokens (KMS/vault), least-privilege scopes, immediate revocation handling.
- Consent ledger check + suppression enforcement before every send.
- Immutable audit log of every agent action, message, approval, write-back.

**Non-functional Requirements**
- TLS 1.2+; AES-256 at rest; field-level encryption of tokens/PII; SOC 2 Type II roadmap; GDPR/TCPA/CAN-SPAM compliance.

**User Stories**
- As a buyer, I need confidence my accounting credentials and customer data are safe.
- As an admin, I want granular roles and a full audit trail.

**Acceptance Criteria**
- No cross-tenant access possible (RLS verified by tests).
- Tokens never appear in logs; revocation stops all access immediately.
- Every outbound + write-back action appears in exportable audit log.

**Technical Constraints**
- Postgres RLS; vault/KMS; OWASP ASVS baseline; dependency + secret scanning in CI.

**Edge Cases**
- Token revoked mid-sequence → pause + notify reconnect; compromised account → session revoke + alert.

**Success Metrics**
- 0 cross-tenant incidents; 100% sensitive actions audited; SOC 2 readiness milestones met.
