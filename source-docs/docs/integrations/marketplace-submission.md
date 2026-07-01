# Marketplace submission kit (T11.1)

Everything needed to list Cadence on the **QuickBooks (Intuit) App Store** and the
**Xero App Store**, plus Stripe. The OAuth install flow is already built (see
`packages/adapters/src/oauth.ts`, `packages/security/src/oauth-state.ts`, and the
`connectors` controllers); this is the external submission checklist + the exact
values reviewers ask for. Anything marked **[you]** needs a human with the
developer account.

## 0. Pre-reqs

- Production deploy reachable over HTTPS (see `docs/runbooks/deployment.md`).
- A published **Privacy Policy** and **Terms** URL (`https://cadence.app/privacy`,
  `/terms`). **[you]** — both are required by Intuit and Xero review.
- A support email + marketing site.

## 1. OAuth configuration (both providers)

| Field | Value |
|-------|-------|
| Redirect URI (prod) | `https://app.cadence.app/api/connectors/callback` |
| Redirect URI (staging) | `https://app-staging.cadence.app/api/connectors/callback` |
| Flow | Authorization Code (with rotating refresh tokens) |
| State | HMAC-SHA256 signed, 10-min expiry (CSRF-safe) — already implemented |
| Token storage | AES-256-GCM at rest in the vault; never logged |

Set these env vars on the deploy (already in `render.yaml`):
`ACCOUNTING_OAUTH_REDIRECT_URL`, `QUICKBOOKS_CLIENT_ID/SECRET`,
`XERO_CLIENT_ID/SECRET`.

## 2. QuickBooks Online (Intuit)

**Scopes requested:** `com.intuit.quickbooks.accounting` (read invoices/customers
+ write payment records), `openid`, `profile`, `email`.

**Submission checklist** (Intuit Developer → your app → Production):
- [ ] **[you]** Create the production app + get prod keys; paste into Render secrets.
- [ ] **[you]** Complete the app profile: name, description (see §5 copy), logo,
      categories (Invoicing / Payments), support + privacy + EULA URLs.
- [ ] **[you]** Fill the **security questionnaire** (Intuit requires it): describe
      token encryption at rest (AES-256-GCM), TLS in transit, tenant isolation
      (Postgres RLS), least-privilege scopes, breach process (link
      `docs/runbooks/incident-response.md`).
- [ ] **[you]** Record the **end-to-end demo video** (connect → sync invoices →
      send a reminder → record a payment). Reviewers require it.
- [ ] **[you]** Pass Intuit's automated **OAuth + disconnect** test: the app must
      handle token revocation and the "Disconnect" flow. (Our callback stores
      tokens; confirm a disconnect path exists or add one — see §6.)
- [ ] **[you]** Submit for review; expect 1–3 weeks + a round of feedback.

## 3. Xero

**Scopes requested:** `offline_access accounting.transactions
accounting.contacts.read accounting.settings.read`.

**Submission checklist** (Xero Developer → My Apps → Submit for review):
- [ ] **[you]** Create the production app; set the redirect URI; paste keys into
      Render secrets.
- [ ] **[you]** Note: Xero returns the tenant via `GET /connections` — already
      handled (`fetchXeroConnections`). Confirm multi-org handling in the demo.
- [ ] **[you]** App listing: name, summary, description (§5), screenshots, logo,
      pricing, support + privacy URLs, regions.
- [ ] **[you]** Complete Xero's **security assessment** (similar to Intuit's).
- [ ] **[you]** Record the demo; submit. Xero review is typically 2–6 weeks.

## 4. Stripe

Stripe is a direct API integration (no marketplace review needed to go live), but
if listing on the **Stripe App Marketplace**:
- [ ] **[you]** Register the app, set the OAuth redirect, complete the listing.
- [ ] Webhooks already implemented (`apps/api/src/webhooks`); set
      `STRIPE_WEBHOOK_SECRET` in prod.

## 5. Listing copy (reuse across stores)

**Name:** Cadence — AR Collections on Autopilot

**Short summary (≤140 chars):** Cadence chases overdue invoices for you — polite,
on-brand reminders across email & SMS that get you paid faster, safely.

**Description:**
> Cadence connects to QuickBooks/Xero, watches your accounts receivable, and runs
> smart, tone-safe dunning sequences so you stop chasing late payers by hand. It
> scores which invoices are likely to go late, picks the right channel and timing,
> drafts on-brand messages, respects quiet hours and consent, handles replies and
> payment plans, and reconciles payments back to your ledger automatically. You
> keep control: every escalation is auditable and approval-gated where you want it.
> Teams using Cadence recover cash faster and cut DSO without adding headcount.

**Categories:** Invoicing, Payments, Cash Flow.

## 6. Known code follow-ups before submitting

- ✅ **Disconnect/revoke endpoint** — implemented: `DELETE /connections/:id`
  (`connections.controller.ts`) revokes the token **upstream** at the provider
  (`revokeToken` → QBO/Xero revoke endpoints) then clears the stored tokens and
  flips the connection to `revoked` locally (access stops immediately even if the
  upstream call fails). Owner/finance only; audited as `connection.disconnected`.
- **Token refresh on 401** — `connector-credentials.ts` refreshes on expiry;
  confirm it also handles a hard 401 (revoked) by marking the connection
  `revoked` rather than retrying forever. (Small follow-up.)
