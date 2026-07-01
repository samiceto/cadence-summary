# Spec — Part 11: Growth Strategy (Cadence)

**Functional Requirements**
- Marketplace app listings (QuickBooks/Xero/Stripe) with OAuth install flow.
- Partner/reseller portal for accountants (multi-client management, referral attribution).
- In-product ROI calculator + recovered-cash reporting; light branding on payment requests.
- Freemium read-only insights; activation = paid conversion path.

**Non-functional Requirements**
- Install→activation friction minimized (<10 min TTV); referral attribution accurate.

**User Stories**
- As an accountant, I want to onboard many clients and earn referral revenue.
- As a buyer, I want to see projected ROI before paying.

**Acceptance Criteria**
- Marketplace install completes OAuth + first insight without manual setup.
- Partner referrals tracked + attributed to billing.

**Technical Constraints**
- Marketplace OAuth/app review compliance; multi-tenant partner access via RLS-scoped memberships.

**Edge Cases**
- Marketplace policy change → adapter-isolated; partner offboarding → client tenant continuity.

**Success Metrics**
- CAC payback < 5 mo; NRR > 115%; install→activation > 60%; partner-sourced share growing.
