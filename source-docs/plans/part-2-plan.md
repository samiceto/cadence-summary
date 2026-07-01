# Plan — Part 2: Customer Profile (Cadence)

**Objectives:** Fast, tailored onboarding + partner (accountant) multi-tenant access.

**Architecture Decisions:** Vertical default presets (cadence/tone/channels); tenant-membership model for partners; RLS-scoped switching.

**Dependencies:** Auth/RBAC (Part 7), sequence presets (Part 4/6), connectors (Part 4).

**Milestones:** M1 onboarding wizard; M2 vertical presets; M3 partner multi-tenant switcher; M4 fit/anti-persona check.

**Timeline:** ~1.5 weeks.

**Risks:** onboarding friction; partner access complexity.

**Mitigation:** template presets; reuse membership model; <10 min TTV target tested.

**Resource Estimates:** 1 FE + 1 BE, ~1.5 weeks.
