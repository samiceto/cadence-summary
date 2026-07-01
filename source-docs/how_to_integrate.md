# How to Integrate Cadence for a Client

A practical guide to deploying and configuring Cadence for a paying client — connecting **their** accounting system, **their** sending identity, and **their** billing so the product works for their business. Written for you (the implementer) selling/onboarding a client.

---

## 1. Deployment model — pick one

| Model | What it means | Best for |
|-------|---------------|----------|
| **Single-tenant per client** *(recommended for selling)* | Each client gets their own deploy (own API/worker/web/DB) | Clean isolation, simple billing, white-label per client |
| **Multi-tenant (shared)** | One deploy serves many clients via built-in row-level-security tenancy | Running it yourself as a SaaS with many small customers |

The app supports both (it's multi-tenant internally with RLS). For **selling to individual clients**, single-tenant-per-client is simplest to reason about and bill.

---

## 2. Accounts the CLIENT must provide (or you create on their behalf)

| Service | Purpose | Who signs up |
|---------|---------|--------------|
| **Accounting** — QuickBooks Online or Xero | Source of invoices/customers/payments | Client (you connect via OAuth) |
| **Stripe** | Collect payments + (optionally) bill the client | Client's Stripe account |
| **Resend** (or any email provider) | Send email reminders | You or client; needs their domain verified |
| **Twilio** | Send SMS reminders | You or client; needs a number + A2P 10DLC registration |
| **OpenAI** | AI message drafting | Yours or client's API key |
| **Clerk** | Authentication for the client's team logging into the dashboard | Yours (per-client app) |
| **Neon** (Postgres) + **Render** | Database + hosting | Yours (you operate it) |

> **Sending identity matters.** Reminders should come from the *client's* domain (email) and *client's* number (SMS) for deliverability and trust. Plan to verify their domain in Resend and provision/register their Twilio number early — these have lead times.

---

## 3. Infrastructure setup (one-time, per client)

1. **Fork/clone the repo** for the client (or use a shared repo with per-client config).
2. **Create the database** — a Neon project. Copy the **pooled** and **unpooled** connection strings.
3. **Run migrations** against the new DB:
   ```bash
   pnpm --filter @cadence/db db:migrate
   ```
4. **Deploy** — push to GitHub and create a Render Blueprint from `render.production.yaml` (paid, always-on — required for reliable Stripe webhooks). See `docs/runbooks/deployment.md`.
5. **Fill secrets** in the Render `cadence-shared` env group + per-service vars (full list in §5).

---

## 4. Connect the client's services

### Accounting (QuickBooks / Xero) — the core integration
1. Register a developer app in the **Intuit** and/or **Xero** developer portal (one-time, yours).
2. Set the OAuth redirect to `https://<client-app>/api/connectors/callback`.
3. Put the client ID/secret in env (`QUICKBOOKS_CLIENT_ID/SECRET`, `XERO_CLIENT_ID/SECRET`).
4. The client clicks **Connect** in the app → completes OAuth → Cadence stores their token **encrypted** and begins syncing invoices.
5. **To go live publicly**, the accounting apps need marketplace **app-review** approval (see `docs/integrations/marketplace-submission.md`). For a single named client you can often use the app in development/private mode without full review.

### Email (Resend)
- Verify the client's sending **domain** in Resend (SPF/DKIM DNS records).
- Set `RESEND_API_KEY` and `EMAIL_FROM` (e.g. `billing@theirdomain.com`).
- The app supports **per-tenant verified sending identities** — only verified domains/numbers are ever used.

### SMS (Twilio) — full setup

Cadence sends SMS through a pluggable adapter: when `TWILIO_ACCOUNT_SID`,
`TWILIO_AUTH_TOKEN`, and `TWILIO_FROM_NUMBER` are set it uses the **real Twilio**
adapter; if any is missing it silently falls back to an in-memory **fake** (no
delivery). So "wiring up real SMS" = providing those three values on a properly
registered Twilio number. Step by step:

1. **Create / upgrade a Twilio account.** A *trial* account can only text
   **verified** numbers and prepends a "sent from a trial account" notice —
   fine for a smoke test, not for production. **Upgrade to a paid account** before
   go-live (Console → top banner → Upgrade).
2. **Buy an SMS-capable number** (Console → Phone Numbers → Buy a number; ensure
   the **SMS** capability is checked). Use a local long code or toll-free number;
   for higher volume consider a messaging service.
3. **Register for the carriers (US):** complete **A2P 10DLC** — register a
   **Brand** (the client's business) and a **Campaign** (use case: "account
   notifications / debt collection"). This has **lead time (days)** and is
   **mandatory** for US long-code traffic; unregistered traffic is filtered.
   - Toll-free numbers need **toll-free verification** instead.
   - **International** (e.g. UK/EU/PK): check Twilio **Geo-permissions**
     (Messaging → Settings → Geo permissions) for each destination country, and
     register a **Sender ID** / comply with local rules where required.
4. **Set the env vars** (Render `cadence-shared` group):
   `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_FROM_NUMBER` (E.164, e.g.
   `+14155551234`). For per-tenant numbers, also add a **verified sending
   identity** in-app (only verified numbers are ever used — deliverability gate).
5. **Wire the inbound webhook for replies & opt-outs.** In the number's
   Messaging config, set the inbound webhook to
   `https://<client-api>/webhooks/twilio/<tenantId>`. Cadence classifies inbound
   replies and routes them: **STOP/opt-out → unsubscribe → the contact is
   suppressed automatically**, while promise-to-pay / disputes pause or escalate
   to a human. (Delivery-status receipts are a separate Twilio status callback
   if you want them.)
6. **Consent is required before Cadence will text anyone.** This is enforced in
   code, not optional: the policy gate returns **`sms_requires_optin`** and
   **suppresses** any SMS to a customer who hasn't granted SMS consent (email is
   allowed as transactional; SMS is not — TCPA). The client must capture opt-in
   and record it via `recordConsent(tx, tenantId, { customerId, channel: 'sms',
   state: 'granted', source })` — e.g. a checkbox on their invoice/portal, or an
   inbound **START**. Without this, SMS steps will never send (by design).
7. **Other gates that apply automatically:** **quiet hours** (SMS only sends
   08:00–20:00 local; otherwise it *defers* and reschedules), the **per-channel
   frequency cap**, **VIP approval**, and **suppression/opt-out**. Nothing
   bypasses these.
8. **Test delivery end-to-end** (two helper scripts ship in the repo):
   ```bash
   # raw provider send — confirms the number/account actually deliver:
   pnpm --filter @cadence/domain exec \
     node --env-file=.env --import tsx scripts/test-sms.ts +<client-test-number>
   # full pipeline — seeds an overdue invoice and drives sequence → draft →
   # policy gate → real Twilio on the SMS escalation step:
   pnpm --filter @cadence/domain exec \
     node --env-file=.env --import tsx scripts/test-sms-pipeline.ts +<client-test-number>
   ```
   On a trial account, first add the test number under **Verified Caller IDs**.
   Check **Console → Monitor → Messaging logs** for the delivery status / error
   code (e.g. `30007` carrier filtering, `21608` unverified, `30008` unknown).

**SMS go-live checklist:** paid account · SMS-capable number · A2P 10DLC (or
toll-free / geo-permissions) **approved** · env vars set · inbound webhook wired ·
an opt-in capture path in place · both test scripts deliver to a real handset.

### Payments + billing (Stripe)
- Use the client's **Stripe secret key** (`STRIPE_SECRET_KEY`).
- Create the subscription **products/prices** if you're billing the client through the app, and set `STRIPE_PRICE_STARTER/GROWTH/PRO`.
- Register a webhook at `https://<client-api>/webhooks/stripe/<tenantId>` and set `STRIPE_WEBHOOK_SECRET`.

### Auth (Clerk)
- Create a Clerk application for the client; set `CLERK_SECRET_KEY` + `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`.
- Invite the client's finance team as users. The dashboard authenticates them; their role (owner/finance/viewer/approver) controls what they can do.

---

## 5. Environment variables (the full checklist)

Set these in the Render `cadence-shared` group (and the two web-service vars). `.env.example` documents every key.

**Required to run:** `DATABASE_URL`, `DATABASE_URL_UNPOOLED`, `REDIS_URL` (auto-wired), `CADENCE_ENCRYPTION_KEY` (`openssl rand -base64 32`), `APP_URL`, `NODE_ENV=production`.
**Auth:** `CLERK_SECRET_KEY`, `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`.
**Billing:** `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_PRICE_{STARTER,GROWTH,PRO}`.
**Comms:** `RESEND_API_KEY`, `EMAIL_FROM`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_FROM_NUMBER`.
**AI:** `OPENAI_API_KEY` (models default to `gpt-4o-mini` for drafting, `gpt-4o` for synthesis).
**Accounting OAuth:** `ACCOUNTING_OAUTH_REDIRECT_URL`, `QUICKBOOKS_CLIENT_ID/SECRET`, `XERO_CLIENT_ID/SECRET`.
**Monitoring (optional):** `NEXT_PUBLIC_SENTRY_DSN`, `NEXT_PUBLIC_POSTHOG_KEY`, `SENTRY_*`.
**Web:** `API_URL`, `NEXT_PUBLIC_API_URL` (both = the API's URL).

> ⚠️ Never commit `.env`. Secrets are entered once in the Render dashboard (the Blueprint marks them `sync: false`).

---

## 6. White-labeling for the client

- **Branding:** update the web app's copy/colors (it uses CSS variables + inline styles, no heavy framework) and the mission page.
- **Sending identity:** their domain + number (above) — this is what customers see.
- **Tone & cadence:** dunning presets are per-industry; the AI drafts in their voice. Tune the preset for their vertical during onboarding.
- **Domain:** point a custom domain (e.g. `collections.theirco.com`) at the Render web service.

---

## 7. Go-live checklist

- [ ] Production deploy (always-on, not free tier) is up; `GET /health` returns ok.
- [ ] Migrations applied to the client's DB.
- [ ] Accounting connected (OAuth) and first sync completed — invoices visible.
- [ ] Email domain verified; a test reminder delivers to inbox (not spam).
- [ ] SMS number registered (A2P) and a test message delivers.
- [ ] Stripe webhook registered and a test event returns 200.
- [ ] Clerk users invited; the client can log in and see their dashboard.
- [ ] Quiet hours, consent, and suppression confirmed for their region.
- [ ] Approval gates set for VIP accounts per the client's preference.
- [ ] Backups/PITR enabled on Neon; monitoring alerts wired (optional but recommended).
- [ ] **Legal:** the client has a Privacy Policy + Terms, and consent to contact their customers.

---

## 8. What you operate vs. what the client owns

- **You operate:** the deploy (Render), the database (Neon), the developer apps (Intuit/Xero), monitoring, updates.
- **The client owns:** their data, their Stripe account, their sending domain/number, their customer relationships — and can export or offboard at any time (no lock-in).

---

## 9. Ongoing

- **Risk model** improves as real payment history accrues — retrain with `pnpm --filter @cadence/domain train:risk:real` once there's enough data.
- **Scaling:** the production Blueprint autoscales the API and worker; a queue-depth autoscaler is included for bursty load.
- **Support:** runbooks for deployment, incident response, alerting, and disaster recovery live in `docs/runbooks/`.
