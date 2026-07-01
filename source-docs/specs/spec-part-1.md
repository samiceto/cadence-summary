# Spec — Part 1: Vision & Mission (Cadence)

**Functional Requirements**
- System must articulate and enforce the "relationship-safe autonomous AR" promise across every feature (every send policy-bounded, reversible, auditable).
- Provide a tenant-visible statement of mission/principles in-app and surface the north-star (recovered cash / DSO) on the primary dashboard.

**Non-functional Requirements**
- Vision constraints (relationship safety, affordability, margin) are testable gates in CI, not aspirational text.

**User Stories**
- As a founder, I want assurance the tool won't damage client relationships, so I trust it to act autonomously.
- As an operator, I want to see recovered cash so the product proves its value.

**Acceptance Criteria**
- Dashboard shows recovered cash + DSO trend on first load post-activation.
- Any feature violating relationship-safety principle is blocked by policy engine.

**Technical Constraints**
- Principles encoded as enforceable policy + eval gates (Parts 6/10), not just docs.

**Edge Cases**
- New tenant with no payment history → show insight placeholders, no misleading metrics.

**Success Metrics**
- 100% of outbound actions policy-checked; recovered-cash metric visible to 100% of activated tenants.
