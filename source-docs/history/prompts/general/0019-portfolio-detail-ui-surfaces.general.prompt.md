---
id: 0019
title: Portfolio detail — add missing UI surfaces
stage: general
date: 2026-06-30
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: Sami Ullah
command: general
labels: ["portfolio", "docs", "ui", "marketing"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - portfolio_detail.md
tests:
 - none
---

## Prompt

read D:\projects\projects\Cadence\portfolio_detail.md, and check if ui has some features missing, add in this file. and if you think of this file needs to be updated, because i will paste this to my portfolio project details page, to make client introduce everything about project

## Response snapshot

Audited the actual Next.js web app (apps/web/app + components) against the portfolio
doc. The doc described only the operator console (Dashboard, Invoices, Activity) but the
app ships four more user-facing surfaces that were undocumented or only mentioned in a
single throwaway line:

- `/onboarding` — business-profile → ROI projection → commit wizard with a live fit/anti-persona check (recovered cash, DSO reduction, payback).
- `/insights` — freemium read-only preview with locked features + "Activate dunning" CTA.
- `/partners` — accountant partner portal: book of business, MRR, referral commission, one-click active-client switcher.
- `/mission` — versioned in-app relationship-safety principles.
- `/` — landing page entry to onboarding.

Edits made to `portfolio_detail.md`:
1. Enriched the "Operator console" capability bullet (DSO trend arrow, recommended
   channel, model tier) and added four new capability bullets for the surfaces above.
2. Replaced the generic "Partner / freemium / onboarding flows" line with a concrete,
   wired-end-to-end description.
3. Extended the live walkthrough with steps 5–8 (direct links) covering onboarding,
   insights, partners, and mission.

## Outcome

- ✅ Impact: Portfolio doc now reflects the full shipped UI; client-facing intro is complete.
- 🧪 Tests: none (documentation only).
- 📁 Files: portfolio_detail.md
- 🔁 Next prompts: optionally add screenshots/GIFs of each surface for the portfolio page.
- 🧠 Reflection: source of truth was the code, not the prior doc — verified routes/components directly.

## Evaluation notes (flywheel)

- Failure modes observed: none.
- Graders run and results (PASS/FAIL): n/a.
- Prompt variant (if applicable): n/a.
- Next experiment (smallest change to try): include thumbnail captures per route.
