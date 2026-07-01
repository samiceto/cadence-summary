# Monitoring as code (T10.5)

Dashboards and alert rules for Cadence, kept in version control so they're
reviewable and reproducible. Three layers:

| Layer | Tool | Files |
|-------|------|-------|
| Errors / performance / latency alerts | Sentry | `sentry/alert-rules.json`, `sentry/apply-sentry.sh` |
| Product / finance dashboards + funnels | PostHog | `posthog/insights.json`, `posthog/apply-posthog.sh` |
| Canonical thresholds (source of truth) | — | `alerts.yaml` |

`alerts.yaml` is the single source of truth; it mirrors
[`docs/runbooks/alerting.md`](../docs/runbooks/alerting.md) §2. Edit thresholds
there, re-run the apply scripts, and keep the runbook table in sync.

## Apply

Both scripts are **idempotent** (skip anything that already exists by name) and
need only an API token + project identifiers.

```bash
# Sentry
SENTRY_AUTH_TOKEN=… SENTRY_ORG=… SENTRY_PROJECT=… \
  bash monitoring/sentry/apply-sentry.sh

# PostHog
POSTHOG_HOST=https://us.posthog.com POSTHOG_PROJECT_ID=… POSTHOG_API_KEY=… \
  bash monitoring/posthog/apply-posthog.sh
```

Both require `jq` and `curl`.

## Events powering the PostHog dashboards

These fire through `@cadence/observability captureEvent()` (the T12.1 taxonomy),
wired at their lifecycle anchors:

Names are the canonical taxonomy in `packages/metrics/src/events.ts` (`EVENTS`):

| Event | Emitted from | Powers |
|-------|--------------|--------|
| `tenant_onboarded` | `domain/onboarding.ts` | activation funnel |
| `connector_installed` | `api/connectors/connector-callback` | activation funnel |
| `sequence_started` | `domain/sequences.ts` | activation funnel |
| `payment_applied` | `domain/payments.ts` (`recovered_cents`) | north-star, activation funnel |
| `send_attempted` / `message_gated` | `domain/send.ts` | send health |
| `message_sent` | `domain/send.ts` | send health (delivered) |
| `reply_classified` (`intent`) | `domain/replies.ts` | reply-intent distribution |
| `queue_depth` / `queue_dlq_size` | `worker/heartbeat.ts` (every 60s) | backlog/DLQ alerts |

The event sink is wired to PostHog in the app bootstrap via
`setEventSink(...)`; until then `captureEvent` is a no-op-safe debug log, so these
calls are harmless in tests/local. To light up the dashboards in production,
confirm the bootstrap calls `setEventSink` with the PostHog client.

## Alert delivery

The apply scripts create the **rules**; attach notification channels
(Slack/PagerDuty/Opsgenie) once in each tool's UI per the on-call model in the
alerting runbook. SEV1/SEV2 page; SEV3 goes to a non-paging channel.
