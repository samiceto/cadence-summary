# Connecting QuickBooks (Intuit) — step-by-step

This guide takes you from nothing to a **live QuickBooks sync** in Cadence. It
covers the free **sandbox** (fake company, for testing) first, then what changes
for **production** (a real company).

Cadence already has the full QuickBooks connector built
(`packages/adapters/src/impls/qbo-connector.ts`) and the OAuth endpoints wired
(`/connectors/quickbooks/authorize` → consent → `/connectors/callback`). All you
provide is an Intuit app + its credentials.

---

## What Intuit is for

QuickBooks is the accounting system Cadence reads invoices from. **Intuit** (its
parent) runs the OAuth login — like "Sign in with Google", it proves you own the
QuickBooks company and hands Cadence a token to read its invoices. Nothing else.

---

## Part A — Sandbox (test data, ~10 minutes)

### 1. Create an Intuit developer account
- Go to <https://developer.intuit.com> → **Sign up** (free).

### 2. Create an app
- Dashboard → **My Apps** → **Create an app** → **QuickBooks Online and Payments**.
- Give it a name (e.g. "Cadence Dev").
- When asked for scopes, select **`com.intuit.quickbooks.accounting`**.

### 3. Get a sandbox company
- A free sandbox company is created automatically. Confirm under
  **Dashboard → Sandbox** (top-right menu). It comes pre-loaded with sample
  customers and invoices — that's the fake data you'll sync.

### 4. Grab the sandbox keys
- In your app → **Keys & credentials** → **Development** tab.
- Copy the **Client ID** and **Client Secret**.

### 5. Register the redirect URI
- Same page → **Redirect URIs** → add **exactly**:
  ```
  http://localhost:3000/api/connectors/callback
  ```
- Save. (This must match `ACCOUNTING_OAUTH_REDIRECT_URL` in `.env` character-for-character,
  or Intuit rejects the consent screen before you even log in.)

### 6. Put the credentials in `.env`
Set these three keys (see `.env.example` for context):
```bash
QUICKBOOKS_CLIENT_ID=<your Development Client ID>
QUICKBOOKS_CLIENT_SECRET=<your Development Client Secret>
ACCOUNTING_OAUTH_REDIRECT_URL=http://localhost:3000/api/connectors/callback
```
> Never commit `.env`. It's gitignored.

### 7. Start the stack
From the repo root:
```bash
# API + workers in one process (single-service dev), reads .env:
cd apps/api && RUN_WORKERS_IN_API=true PORT=4000 \
  node --env-file=../../.env dist/main.js      # run `pnpm --filter @cadence/api build` first if dist is stale

# Web (separate terminal). Point it at your tenant so the dashboard reads it:
cd apps/web && \
  DEMO_TENANT_ID=<your-tenant-id> API_URL=http://localhost:4000 \
  NEXT_PUBLIC_API_URL=http://localhost:4000 NEXT_PUBLIC_DEMO_TENANT_ID=<your-tenant-id> \
  pnpm exec next dev -p 3000
```
`<your-tenant-id>` is the tenant you want to attach QuickBooks to. The dev demo
tenant is `00000000-0000-4000-8000-0000000000a1` (created by
`packages/domain/scripts/bootstrap-qbo-tenant.ts`).

### 8. Connect from the UI
- Open <http://localhost:3000/dashboard>.
- In **Accounting connections**, click **+ Connect QuickBooks**.
- You're sent to Intuit → pick the **sandbox company** → **Connect**.
- You land back on `/dashboard?connected=success`. A connection row now appears.

> Under the hood: the browser hits `GET /connectors/quickbooks/authorize` (returns
> the consent URL), Intuit redirects to `/api/connectors/callback`, the API
> exchanges the code and stores **encrypted** tokens, creating a `connections` row.

### 9. Sync invoices
- Click **Sync now** on the connection.
- The API enqueues a job on the `sync` queue; a worker pulls customers + invoices
  from QuickBooks into the database.
- Refresh **/invoices** — the sandbox's invoices appear, risk-scored. The cadence
  then scores → sequences → drafts → (policy-gated) sends; watch **/activity**.

---

## Part B — Production (a real company)

Same app, different track. Intuit requires review before real companies can connect.

1. **Production keys**: app → **Keys & credentials → Production** tab. Different
   Client ID/Secret from the sandbox ones.
2. **Production redirect URI**: add your deployed callback, e.g.
   `https://<your-web-domain>/api/connectors/callback`. Set
   `ACCOUNTING_OAUTH_REDIRECT_URL` to that value in the deployed environment
   (Render env group `cadence-shared`).
3. **App assessment / review**: Intuit makes you complete an app questionnaire
   (security, EULA, privacy policy, logo, scopes justification) before production
   OAuth is enabled. Budget a few days for review.
4. **Disconnect support**: already implemented — `DELETE /connections/:id` revokes
   the token upstream at Intuit and locally. Intuit requires this for approval.
5. **Token refresh**: handled by the connector; QuickBooks refresh tokens roll and
   are re-persisted on use.

---

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Consent page shows a **redirect_uri mismatch** | The URI in step 5 doesn't exactly match `ACCOUNTING_OAUTH_REDIRECT_URL`. Match scheme, host, port, path. |
| `authorize` returns **500** | `QUICKBOOKS_CLIENT_ID/SECRET` or `ACCOUNTING_OAUTH_REDIRECT_URL` missing in the API's env. Also can be a transient DB timeout — retry. |
| Lands on `/dashboard?connected=error` | Token exchange failed — usually wrong client secret or an expired `state` (the signed handshake is valid for **10 minutes**; restart the connect flow). |
| `connected=denied` | You clicked **Cancel** on Intuit's screen. |
| Sync runs but no invoices | Sandbox company is empty, or the worker isn't running. Ensure `RUN_WORKERS_IN_API=true` (or run `apps/worker` separately). |
| New connections **hang** locally | Intermittent network to the database from your machine. The running API's warm pool still works; retry the action. |

---

## Reference — files involved

- `apps/api/src/connectors/connectors.controller.ts` — `GET /connectors/:system/authorize`
- `apps/api/src/connectors/connector-callback.controller.ts` — `POST /connectors/callback`
- `apps/api/src/connectors/oauth-config.ts` — reads the env credentials
- `apps/web/app/api/connectors/callback/route.ts` — browser landing that forwards to the API
- `apps/web/components/connections-panel.tsx` — the **Connect / Sync** UI
- `packages/adapters/src/impls/qbo-connector.ts` — the QuickBooks data sync
- Xero works the same way — swap `quickbooks` → `xero` and set `XERO_CLIENT_ID/SECRET`.
