# Monitoring, Dashboards & Alerting (T10.5)

What we watch, where we watch it, and the thresholds that page someone. Signals
flow through the `@cadence/observability` seams: exceptions → Sentry via
`captureError`, product/finance events → PostHog via `captureEvent`, structured
logs via `logger`.

## 1. Dashboards

### Sentry (errors & performance)
- **Error rate by service** (api, worker) and by `queue` tag on worker errors.
- **Release health** — regressions after deploys.
- **Apdex / p95 latency** on the API transaction set.

### PostHog (product & finance)
Built from the T12.1 event taxonomy (`captureEvent`):
- **Activation funnel** — `onboarding.completed` → connection established →
  first sequence → first recovered payment.
- **North-star** — recovered cash and DSO trend (also surfaced in the tenant
  dashboard, sourced from reconciled data only — Part 8/12).
- **Send health** — sends attempted vs. delivered vs. deferred (quiet-hours /
  throttle), reply intents distribution.

### Infrastructure (host provider)
- Neon: compute utilization, connection count, storage, replication lag.
- Redis: memory, queue depths per `QUEUES` name, DLQ size.
- API/worker: CPU, memory, replica count, restart count.

## 2. Alert thresholds

Tune to traffic after the first few weeks; these are sane starting points.

| Alert | Signal | Threshold | Severity | Owner |
|-------|--------|-----------|----------|-------|
| API error spike | Sentry error rate | >2% of requests for 5 min | SEV2 | on-call |
| API down | Health check | 2 consecutive failures | SEV1 | on-call |
| API latency | p95 transaction | >1s for 10 min | SEV3 | on-call |
| Worker error spike | `captureError` (worker) | >10/min for 5 min | SEV2 | on-call |
| Send queue backlog | `send` queue depth | >1000 or rising 15 min | SEV2 | on-call |
| Sequence backlog | `sequence` queue depth | >5000 for 30 min | SEV3 | on-call |
| DLQ growth | dead-letter size | >0 sustained / >50 burst | SEV2 | on-call |
| Reconciliation drift | `reconcile.drift_detected` | drift/checked >1% in a run | SEV2 | finance-eng |
| Sync failures | `auth_revoked`+`provider_error` | >20% of connections in a run | SEV2 | on-call |
| Deliverability | bounce/complaint rate | bounce >5% or complaint >0.1% | SEV2 | on-call (auto-pause exists, T9.x) |
| Provider down | Twilio/Resend 5xx | failover engaged >5 min | SEV2 | on-call |
| LLM degraded | OpenAI 401/429 rate | sustained 5 min | SEV3 | on-call (templates keep sends safe) |
| DB connections | Neon conn count | >80% of cap for 10 min | SEV2 | on-call |
| Cost guard | LLM spend / metered usage | daily budget exceeded | SEV3 | eng-lead |

## 3. On-call

- One primary + one secondary, weekly rotation; secondary is the escalation
  path if the primary doesn't ack within 15 min.
- Pages go to the alerting tool (PagerDuty/Opsgenie); SEV3 to a non-paging
  channel reviewed each business day.
- Every page must map to a runbook section. If it doesn't, the follow-up action
  is to write one (or delete the alert as noise).

## 4. Alert hygiene

- Alert on **symptoms users feel** (backlog, errors, drift), not raw causes.
- Every alert is **actionable** and links here. Noisy alerts are tuned or cut at
  the next on-call retro — alert fatigue is itself a SEV-risk.
- Review thresholds monthly against the previous period's false-positive rate.
