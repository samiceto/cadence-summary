# Constitution — Part 4: Product Requirements (Cadence)

**Core capabilities (MVP):**
1. **Accounting sync:** QuickBooks/Xero (+ Stripe Invoicing) connectors → invoices, customers, payment status (bi-directional).
2. **Late-payment prediction:** score each invoice's risk/timing of going late from history + customer behavior.
3. **Dunning agent:** personalized multi-step sequences across email + SMS; tone/cadence by customer segment; auto-escalation.
4. **Payment plans & negotiation:** agent can offer pre-approved plans / partial payments within policy.
5. **Payment capture:** embedded pay link (Stripe) for one-click payment.
6. **Cash application & reconciliation:** match incoming payments to invoices; write back to accounting.
7. **Policy & approval controls:** human-set rules, tone limits, escalation approvals, blocklist (VIP customers).
8. **Dashboard:** DSO, aging, recovered cash, sequence performance.

**Post-MVP:** more ERPs (NetSuite), cash-flow forecasting, customer self-serve portal, voice channel.

**Functional musts:** every outbound message policy-bounded + logged; nothing sent to blocklisted/VIP without approval; agent actions reversible/auditable.

**Non-functional musts:** email deliverability (SPF/DKIM/DMARC, warmup); SMS compliance (TCPA/consent); idempotent accounting writes.

**Out of scope (v1):** legal debt collection; consumer collections; non-supported ERPs.

**Consistency check:** implements mission (Part 1); bounded by security (Part 7), data (Part 8), quality (Part 10).
