# Spec — Part 3: Business Model (Cadence)

**Functional Requirements**
- Stripe billing with 3 tiers (Starter $99 / Growth $349 / Pro $899), annual default, metered SMS bundles, opt-in success fee on recovered aged debt.
- Enforce tier limits (active invoices, channels, multi-entity, API) and upgrade prompts.
- Success-fee calculation from reconciled recovered debt >90 days (opt-in tenants only), invoiced monthly.

**Non-functional Requirements**
- Billing accuracy 100%; usage metering idempotent; gross margin > 85%.

**User Stories**
- As a buyer, I want pricing anchored to recovered cash so ROI is obvious.
- As finance, I want predictable annual billing plus transparent success-fee math.

**Acceptance Criteria**
- Exceeding invoice limit triggers soft block + upgrade CTA.
- Success fee only charged on reconciled, attributable recoveries; auditable statement provided.

**Technical Constraints**
- Stripe metered billing + webhooks; usage events idempotent; success-fee attribution tied to audit log.

**Edge Cases**
- Refund/clawback after success fee charged → reverse fee.
- Downgrade with over-limit invoices → grace + guided reduction.

**Success Metrics**
- CAC payback < 5 mo; NRR > 115%; ACV ~$4,200; COGS < 12% revenue.
