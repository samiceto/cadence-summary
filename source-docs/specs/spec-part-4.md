# Spec — Part 4: Product Requirements (Cadence)

**Functional Requirements**
- Connectors: QuickBooks, Xero, Stripe Invoicing — bi-directional sync of customers, invoices, payment status.
- Late-payment risk scoring per open invoice.
- Dunning agent: multi-step email+SMS sequences, segment/tone-aware, auto-escalation.
- Payment plans/negotiation within policy; embedded Stripe pay link.
- Cash application & reconciliation with write-back.
- Policy/approval controls: rules, tone limits, escalation approvals, VIP/blocklist.
- Dashboard: DSO, aging, recovered cash, sequence performance.

**Non-functional Requirements**
- Email deliverability (SPF/DKIM/DMARC, warmup); SMS TCPA/consent; idempotent accounting writes; sync freshness < 15 min.

**User Stories**
- As finance, I want overdue invoices chased automatically in a safe tone.
- As an owner, I want to approve sensitive outreach before it sends.

**Acceptance Criteria**
- Connecting accounting imports invoices and computes DSO/aging within minutes.
- No send to blocklist/VIP without explicit approval; all sends logged + reversible.
- Incoming payment auto-matches and writes back without double-applying.

**Technical Constraints**
- BullMQ durable state-machine sequences; connector adapters; webhook-driven status.

**Edge Cases**
- Partial payment, overpayment, duplicate payment, disputed invoice, customer reply mid-sequence → sequence pauses/branches correctly.

**Success Metrics**
- DSO reduction 20–40% in 90 days; payment-link conversion tracked; recovery rate of aged debt.
