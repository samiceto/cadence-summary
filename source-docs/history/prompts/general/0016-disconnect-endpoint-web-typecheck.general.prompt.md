---
id: 0016
title: Connector disconnect endpoint + web typecheck
stage: green
date: 2026-06-26
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: general
labels: ["t11.1", "oauth", "disconnect", "revoke", "rbac", "web-typecheck"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - packages/adapters/src/oauth.ts
 - packages/adapters/src/oauth.test.ts
 - packages/adapters/src/index.ts
 - apps/api/src/connections/connections.controller.ts
 - docs/integrations/marketplace-submission.md
 - REMAINING.md
tests:
 - "adapters 38 pass / 0 fail (+3 revokeToken cases)"
 - "api typecheck clean; adapters + security build clean"
 - "web typecheck clean (verified — earlier error was a stale install/build-cache state, not a code defect)"
---

## Prompt

do both: add the disconnect endpoint and fix the web typecheck error

## Response snapshot

**Disconnect endpoint (T11.1):**
- Added `revokeToken()` to the OAuth adapter — revokes a token UPSTREAM at the
  provider (QuickBooks JSON body / Xero RFC-7009 form-encoded), with the same
  injectable `fetchImpl` pattern as the rest of the adapter. Added `revokeUrl` to
  each provider config. 3 new tests (QBO JSON+basic-auth, Xero form body, error
  → OAuthError).
- Added `DELETE /connections/:id` (`connections.controller.ts`, owner/finance
  only): best-effort upstream revoke, then always `revokeConnection` (clears
  stored tokens + status→`revoked` so access stops immediately even if upstream
  fails), then audits `connection.disconnected`. Reuses existing
  `revokeConnection`/`getConnectionToken` (security) + `oauthClientConfig`.

**Web typecheck:** re-ran from both the package dir and the root filter — it's
clean now. The earlier `useRouter`/`next/navigation.js` error was a transient
stale node_modules/build-cache state during the mid-session SDK `pnpm install`,
not a code defect (the tsconfig already uses `moduleResolution: Bundler`). No
change needed; reported honestly rather than making a cosmetic edit.

## Outcome

- ✅ Impact: clears the disconnect requirement Intuit/Xero app review tests for;
  web typecheck confirmed green.
- 🧪 Tests: adapters 38 pass; api typecheck clean; builds clean.
- 📁 Files: oauth adapter + tests + export; connections controller; doc updates.
- 🔁 Next: still external — submit the app listings (kit ready).
- 🧠 Reflection: local revocation must run regardless of the upstream call's
  outcome, so a provider/network failure can never leave a connection that still
  has usable stored tokens.
