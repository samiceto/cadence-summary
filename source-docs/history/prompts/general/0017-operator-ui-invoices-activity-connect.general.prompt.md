---
id: 0017
title: Operator UI — invoices, activity, connect + dev seed
stage: green
date: 2026-06-30
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: Sami Ullah
command: build missing UI
labels: ["web", "ui", "connectors", "dev-seed", "quickbooks"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
  - apps/web/app/layout.tsx
  - apps/web/app/dashboard/page.tsx
  - apps/web/app/invoices/page.tsx
  - apps/web/app/activity/page.tsx
  - apps/web/components/connections-panel.tsx
  - apps/web/components/approve-button.tsx
  - apps/api/src/dev/dev.controller.ts
  - apps/api/src/app.module.ts
  - apps/api/src/connections/connections.controller.ts
  - apps/api/src/messages/messages.controller.ts
  - apps/api/src/invoices/invoices.controller.ts
  - packages/domain/scripts/bootstrap-qbo-tenant.ts
  - docs/integrations/connect-quickbooks-later.md
tests:
  - manual: POST /dev/seed → 8 customers, 6 open + 2 settled invoices, 6 scored, 4 messages
  - manual: web smoke — /dashboard, /invoices, /activity render seeded data (HTTP 200, content asserts pass)
---

## Prompt

User flow: onboarding wizard → dashboard showed only a number + aging bar, no way to
connect an accounting system, no visibility into the engine. User asked what the
onboarding did, where the real working is, and how to see what's happening. Chose to
demo with fake data (Intuit OAuth parked), then: "Build the missing UI", "go ahead with
your recommendation and create a step-by-step guide on how to add Intuit later",
"complete all remainings".

## Response snapshot

Diagnosed: backend pipeline (sync→score→sequence→send→reconcile) is complete, but the
web app only had landing + onboarding + a read-only metrics dashboard — no connect
button, no invoice list, no activity view. WSL2→Neon cold connections were hanging
(pooler exhaustion from retries), so seeding via one-shot scripts failed; the running
API's warm pool worked.

Added (one API rebuild): `POST /dev/seed` (non-prod, runs on warm pool — seeds demo AR
book + grants dev memberships + runs the real pipeline with fake LLM/comms),
`GET /connections`, `GET /messages` (recent), enriched `GET /invoices` with customer +
risk score. Built web: nav, dashboard ConnectionsPanel (Connect QuickBooks/Xero + Sync),
/invoices table, /activity timeline with Approve button. Restarted API+web pointed at
tenant 00000000-0000-4000-8000-0000000000a1. Wrote Intuit setup guide.

## Outcome

- ✅ Impact: Operator can now see invoices (risk-scored), the dunning activity feed
  (sent / held-for-approval / suppressed), and connect/sync an accounting system from
  the dashboard. Demo data shows recovered $11,700, DSO 38d, populated aging.
- 🧪 Tests: manual API + web smoke tests pass; no automated tests added.
- 📁 Files: 6 web, 5 API, 1 script, 1 doc (see list).
- 🔁 Next prompts: wire approve/sync optimistic refresh; add a real GET /messages
  pagination; connect a live QuickBooks sandbox via the new button.
- 🧠 Reflection: cold DB connects hung on WSL2; routing the seed through the API's warm
  pool was the unlock. tsx didn't emit Nest DI metadata — compiled `dist` was required.

## Evaluation notes (flywheel)

- Failure modes observed: WSL2→Neon cold-connect hangs (no connect_timeout); tsx/esbuild
  not emitting decorator metadata → Reflector DI undefined; slow /mnt/d module loads.
- Graders run and results (PASS/FAIL): manual smoke PASS.
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): set postgres connect_timeout so cold scripts
  fail-fast and retry instead of hanging.
