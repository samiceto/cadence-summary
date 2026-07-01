# Cadence — AR Collections on Autopilot

**An autonomous accounts-receivable agent that gets businesses paid faster — without damaging customer relationships.**

Cadence connects to a company's accounting system, watches their unpaid invoices, and runs intelligent, on-brand follow-up across email and SMS until the money lands in the bank — then reconciles it back to the ledger automatically. It replaces the manual, repetitive, easy-to-drop work of chasing late payers with a system that never forgets, stays polite, and proves its results in recovered cash.

---

## The problem it solves

Every business that invoices on terms has the same silent leak: **late payments**. Revenue looks healthy while cash quietly dries up. Chasing it by hand — remembering who owes what, sending reminders, following up, handling "I'll pay Friday," recording payments — is a full-time job, and it's the first thing that slips when the team is busy.

The result is a higher **DSO (Days Sales Outstanding)**, strained cash flow, and avoidable bad debt.

Cadence closes that leak with disciplined, automated, relationship-safe collections.

---

## What it does — the collections loop

1. **Connect & sync** — Links to QuickBooks Online, Xero, or Stripe and continuously syncs invoices, customers, and payments.
2. **Score risk** — A machine-learning model scores every open invoice for how likely it is to go late, so effort goes where it pays off — not just oldest-first.
3. **Sequence** — Runs a multi-step dunning cadence per invoice (pre-due nudge → due-date reminder → escalating follow-ups), tuned to the client's industry.
4. **Draft (AI)** — Writes each message in the client's voice using AI — and runs every draft through a **tone-safety gate** that blocks anything threatening, shaming, or legally risky. This is enforced in code, not left to chance.
5. **Send** — Delivers across **email and SMS**, automatically respecting **quiet hours, consent, opt-outs, and sending limits**.
6. **Handle replies** — Classifies inbound replies (promise-to-pay, dispute, unsubscribe, question), **negotiates payment plans**, and pauses or escalates appropriately — routing real disputes to a human.
7. **Apply & reconcile** — Records payments and reconciles them back to the accounting ledger, so a paid invoice instantly stops getting chased.
8. **Measure** — Reports the only numbers that matter: **recovered cash** and **DSO**, computed from reconciled accounting data — not vanity metrics.

---

## What makes it different

- **Relationship-safe by design.** Every message is polite and on-brand. The tone-safety gate is a release blocker, not a guideline — the agent can't threaten or shame a customer.
- **You stay in control.** VIP and sensitive accounts can require human approval. A customer who replies or opts out is paused or suppressed instantly.
- **Always auditable & reversible.** Every send, approval, and payment write-back is logged immutably and can be reviewed or exported at any time.
- **Risk-prioritized.** A trained model focuses collections effort on the invoices most likely to slip — more recovered cash per hour of effort.
- **Compliance built in.** Consent tracking, quiet-hours, suppression lists, and per-channel opt-outs are enforced automatically (designed around TCPA / CAN-SPAM / GDPR principles).
- **No lock-in.** The client's data and ledger stay theirs; integrations are provider-agnostic.

---

## Proven, working capabilities

This is a complete, functioning system — not a prototype:

- ✅ **End-to-end collections loop** validated live (connect → sync → score → sequence → tone-safe draft → consent/policy-gated send → reply handling → payment application → reconciliation → metrics).
- ✅ **Operator console** — a live dashboard surfacing recovered cash + DSO (with a trend arrow showing whether DSO is improving) and AR aging, a risk-scored **invoice book** (with the model's recommended channel per invoice), an **activity feed** showing every drafted message with the policy gate's decision (sent / held-for-approval / suppressed) and which AI model tier wrote it, and a one-click **Connect & Sync** panel for accounting systems with an in-UI **approve** action for held messages.
- ✅ **Self-serve onboarding wizard** — a guided business-profile → **ROI projection** → commit flow that runs a live *fit check* (warns honestly when a prospect is a poor fit), projects 90-day and annualized **recovered cash**, **DSO reduction**, **payback period**, and applies an industry-tuned cadence preset automatically.
- ✅ **Freemium insights page** — a free, read-only preview that shows a prospect the cash Cadence would recover for them (recoverable cash, at-risk invoices, DSO) with the paid features clearly locked behind a single **"Activate dunning"** CTA.
- ✅ **Partner portal for accountants** — a multi-client book of business with each client's MRR, a referral attribution + **commission** rollup, and a one-click **active-client switcher** that re-scopes the whole app without re-login.
- ✅ **In-app mission & principles page** — the relationship-safety promise (versioned) surfaced in-product, so tenants can trust the agent to act autonomously.
- ✅ **Accounting connectors** for QuickBooks Online & Xero (OAuth install, sync, idempotent write-back) plus Stripe.
- ✅ **Risk model** — a gradient-boosted late-payment model (with a transparent heuristic fallback for cold start).
- ✅ **AI drafting** with a hard tone-safety gate and an intent-classification accuracy gate.
- ✅ **Multi-channel delivery** (email via Resend, SMS via Twilio) with deliverability auto-pause, throttling, and provider failover.
- ✅ **Billing** — subscription tiers, usage metering, success-fee model, refunds, Stripe webhooks.
- ✅ **Multi-tenant & secure** — database row-level security isolates every tenant; OAuth tokens encrypted at rest (AES-256-GCM); full audit log; GDPR erasure.
- ✅ **Go-to-market surfaces wired end-to-end** — landing page, onboarding/ROI wizard, freemium insights + upgrade intent tracking, and the partner/referral portal all run against the real API (not static mockups).

---

## Under the hood (for the technically curious)

A modern, production-grade architecture:

- **Monorepo** (TypeScript, strict) split into a NestJS **API**, a BullMQ **worker** for background processing, a Next.js **web** dashboard, and ~12 shared domain packages.
- **PostgreSQL (Neon)** with **row-level security** for hard multi-tenant isolation; hand-authored, RLS-complete migrations.
- **Redis + BullMQ** for reliable background jobs with retries and a dead-letter queue.
- **Pluggable adapters** for every external service (accounting, email, SMS, LLM, payments) — swap providers without touching business logic.
- **Observability** seams for Sentry (errors) and PostHog (product analytics), with a queue-depth heartbeat and as-code alert rules.
- **Deploys** as a one-click Render Blueprint (API + worker + web + Redis), with both a free/portfolio profile and a production profile (autoscaling).

---

## Pricing model (illustrative)

A tiered SaaS subscription, optionally plus a small success fee on recovered cash:

| Tier | Price / mo | For |
|------|-----------|-----|
| **Starter** | $99 | Email-only dunning, core automation |
| **Growth** | $349 | Email + SMS + payment plans |
| **Pro** | $899 | Higher volume, advanced controls |

---

## See it live — guided walkthrough

A populated demo is deployed and open — **no sign-up or login required**. It runs in a
scoped demo mode against a sample tenant pre-loaded with a realistic, **synthetic**
accounts-receivable book (fake customers and invoices — no real data). Everything you
see was produced by the real engine, not mocked screenshots.

**▶ Open it:** **https://cadence-web-y3yh.onrender.com**

> First load may take ~30s — the demo runs on a free hosting tier that sleeps when idle.

Walk through it in this order:

1. **Dashboard** — `…/dashboard`
   The north-star view: **recovered cash** and **DSO (Days Sales Outstanding)**, computed
   from reconciled payments — plus the **AR aging** bar (how the outstanding balance is
   spread across 1–30 / 31–60 / 61–90 / 90+ days overdue). Below it is the **Accounting
   connections** panel.

2. **Invoices** — `…/invoices`
   The book the agent is working: each open/overdue invoice with its customer, balance,
   days overdue, and the model's **risk score (0–100)** + recommended channel. Higher
   score = more likely to pay late = worked harder. This is the "effort goes where it
   pays off" idea, made visible.

3. **Activity** — `…/activity`
   The heart of the demo: every message the agent **drafted**, with the **policy gate's
   decision** shown as a status:
   - **● Sent** — passed all safety checks and went out (to a fake inbox in the demo).
   - **● Needs approval** — a **VIP** account held for a human; click **Approve & send**
     to release it (the action is real — it re-queues the send).
   - **● Suppressed** — the customer **opted out**, so the agent refused to contact them.

   That one screen demonstrates the whole promise: persistent follow-up that stays
   relationship-safe, with humans in control of sensitive accounts.

4. **Connect an accounting system** (optional) — on the dashboard, **+ Connect
   QuickBooks** starts the real OAuth install flow. It's the genuine integration entry
   point (it will send you to Intuit's consent screen); in the demo you can stop there —
   the seeded data already shows what a synced book looks like.

### Also worth exploring (direct links)

These surfaces aren't in the top nav but are live in the same demo — open them directly:

5. **Onboarding & ROI wizard** — `…/onboarding`
   Answer a few questions about a business and Cadence runs a **live fit check** and
   projects your **recovered cash, DSO reduction, and payback period** before you commit —
   then applies an industry-tuned cadence preset. This is the self-serve front door.

6. **Freemium insights** — `…/insights`
   The "show, don't tell" upgrade page: a free, read-only preview of the cash Cadence
   would recover (recoverable cash, at-risk invoices, DSO), with paid features locked
   behind a single **Activate dunning** button.

7. **Partner portal** — `…/partners`
   For accountants and bookkeepers managing many clients: a book of business with each
   client's recurring revenue, **referral commission** earned, and a one-click switch of
   the active client that re-scopes the whole app.

8. **Mission & principles** — `…/mission`
   The relationship-safety promise, in-product and versioned — the rules the agent is
   contractually held to.

**What to take away:** in a few clicks you can see a real autonomous collections agent
score invoices, draft on-brand messages, and enforce safety/consent rules — with the
results measured in recovered cash and DSO, not vanity metrics.

> Note for reviewers: the demo deliberately uses synthetic data and fake message
> delivery (nothing is actually emailed/texted). Connecting a real QuickBooks **sandbox**
> is supported and documented in `docs/integrations/connect-quickbooks-later.md`.

---

## The bottom line

Cadence turns accounts-receivable collections from a manual chore into an autonomous, measurable, relationship-safe system. Businesses recover cash faster and cut DSO **without adding headcount** — and they keep full control and a complete audit trail the whole way.
