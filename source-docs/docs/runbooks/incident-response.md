# Incident Response Runbook (T7.5)

How Cadence detects, declares, mitigates, and learns from incidents. Cadence
sends money-related communications on behalf of tenants and holds receivables
data, so the two cardinal rules are: **never send an unsafe or wrong message**,
and **never expose one tenant's data to another**. When in doubt, fail closed.

## 1. Severity model

| Sev | Definition | Examples | Response |
|-----|------------|----------|----------|
| **SEV1** | Customer-facing outage, data exposure, or wrong/unsafe outbound sends | RLS bypass; mass-send of incorrect dunning; API down | Page immediately, all-hands, public status |
| **SEV2** | Major degradation, no data loss | Worker backlog growing unbounded; provider (Stripe/Twilio/Resend) hard-down; sync failing for many tenants | Page on-call, mitigate within 1h |
| **SEV3** | Minor / single-tenant / cosmetic | One tenant's sync stalled; elevated 4xx; dashboard metric lag | Next business day |

## 2. Roles

- **Incident Commander (IC)** — owns the incident, makes the call to mitigate vs.
  investigate, coordinates. The first responder is IC until handed off.
- **Ops/Comms** — owns status updates (internal + external) and the timeline.
- **Subject expert** — pulled in per the affected component.

## 3. Flow

1. **Detect** — alert fires (see [alerting.md](./alerting.md)) or a human reports.
2. **Declare** — open an incident channel, name an IC, set severity.
3. **Stabilize before diagnose** — reach for the kill switches in §5 to stop
   harm first; root-cause after the bleeding stops.
4. **Communicate** — post status on declare, on mitigation, and on resolution
   (cadence: SEV1 every 30 min, SEV2 hourly).
5. **Resolve** — confirm signals are back to baseline and the kill switch is
   safely re-enabled.
6. **Learn** — write a blameless postmortem within 3 business days (§6).

## 4. Triage by signal

| Symptom | First look | Likely cause |
|---------|-----------|--------------|
| Spike in `captureError` from `queue:send` | Worker logs, provider status | Twilio/Resend outage → failover/throttle (T9.3) |
| `reconcile.drift_detected` count rising | `reconcile.drift` audit events | Webhook gap or out-of-order updates (Part 8) |
| Sequence backlog (`sequence`/`send` queue depth) | Redis queue depth, worker replicas | Under-provisioned workers (T9.2) or a poison job in DLQ |
| Auth errors from a connector (`auth_revoked`) | `connections` row, token vault | Tenant revoked OAuth → re-consent flow |
| Cross-tenant data report | RLS policies, `withTenant` usage | **SEV1** — see §5 kill switch 1 |
| LLM 401 / drafting degraded to templates | `OPENAI_API_KEY`, rate limiter | Key rotation needed; system is *safe* (templates are tone-gated) |

## 5. Kill switches (stop harm first)

1. **Halt all outbound sends** — pause the `send` queue (BullMQ
   `queue.pause()`); messages remain queued and resume on unpause. Use for any
   suspected wrong/unsafe send or a data-exposure SEV1.
2. **Pause a single tenant** — disable the tenant's `connections` row so `sync`
   and `sequence` jobs no-op; isolates a blast radius without a global halt.
3. **Disable LLM rewriting** — unset `OPENAI_API_KEY`/flip the adapter to the
   fake; drafting falls back to the tone-safe templates (never unsafe). Safe to
   leave running.
4. **Stop write-back** — connectors degrade gracefully when
   `capabilities.writeBackPayments` is off; accounting remains source of truth.

Outbound safety is defense-in-depth: tone-safety gating, consent/suppression
checks, quiet-hours, and per-tenant throttles all run *before* a send, so a
single failure should not produce an unsafe message.

## 6. Postmortem template

```
# Postmortem: <title> (<date>, SEV<n>)
## Impact         — who/what, how long, how measured
## Timeline       — detection → declare → mitigate → resolve (UTC)
## Root cause     — the actual cause, not the trigger
## Detection gap  — how could we have caught it sooner?
## Action items   — owner + due date; at least one prevents recurrence
## What went well / what was lucky
```

Postmortems are **blameless** — focus on systems and gaps, never individuals.

## 7. Security incidents (data exposure / breach)

1. Declare **SEV1**; IC + security owner.
2. Contain: revoke the implicated credential/token, apply kill switch 1 or 2.
3. Preserve evidence: snapshot logs and the audit-event trail (`audit_events`)
   before remediating.
4. Assess scope from the audit trail (which tenants/PII).
5. Notify per the data-processing agreement and applicable breach-notification
   timelines. Erasure/retention obligations: see `eraseCustomer` (T8.5).
6. Rotate secrets; confirm gitleaks + dependency scans are green (CI
   `security-scan`).
