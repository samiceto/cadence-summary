# Spec — Part 2: Customer Profile (Cadence)

**Functional Requirements**
- Onboarding captures business type, AR volume, billing model, verticals, geo to tailor defaults (cadence, tone, channels).
- Role model for buyer/user/influencer: Owner, Finance, Viewer, Approver; accountant partner (multi-client) access.

**Non-functional Requirements**
- Onboarding to first value < 10 minutes; English-first with i18n-ready strings.

**User Stories**
- As a controller, I want sensible defaults for my industry so I don't configure from scratch.
- As an external accountant, I want to manage multiple client tenants from one login.

**Acceptance Criteria**
- Selecting a vertical loads a tailored default sequence/tone preset.
- Accountant can switch between client tenants without re-auth.

**Technical Constraints**
- Multi-tenant access for partners via tenant memberships; RLS enforced.

**Edge Cases**
- Anti-persona (consumer/B2C, cash-only) → flagged as poor fit during onboarding.

**Success Metrics**
- ≥ 60% activation; ≥ 30% of pipeline partner-sourced over time.
