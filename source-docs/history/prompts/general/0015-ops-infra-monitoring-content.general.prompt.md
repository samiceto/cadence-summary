---
id: 0015
title: Ops/infra, monitoring wiring, content
stage: green
date: 2026-06-26
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: general
labels: ["t9.2", "t9.5", "t10.5", "t8.5", "t11.1", "t11.5", "t12.1", "render", "sentry", "posthog", "observability"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - render.yaml
 - scripts/queue-autoscaler.ts
 - scripts/dr-restore-drill.sh
 - docs/runbooks/deployment.md
 - docs/runbooks/disaster-recovery.md
 - load-test/api-load.js
 - load-test/README.md
 - monitoring/alerts.yaml
 - monitoring/sentry/alert-rules.json
 - monitoring/sentry/apply-sentry.sh
 - monitoring/posthog/insights.json
 - monitoring/posthog/apply-posthog.sh
 - monitoring/README.md
 - packages/observability/src/providers.ts
 - packages/observability/src/index.ts
 - packages/observability/package.json
 - packages/metrics/src/events.ts
 - packages/domain/src/{onboarding,sequences,payments,replies,send}.ts
 - apps/api/src/connectors/connector-callback.controller.ts
 - apps/api/src/main.ts
 - apps/worker/src/{main,heartbeat}.ts
 - docs/integrations/marketplace-submission.md
 - content/{README,blog/what-is-dso-and-how-to-reduce-it,blog/ar-collections-automation-guide,benchmarks}.md
 - REMAINING.md
tests:
 - "all packages build clean; api+worker+domain typecheck clean"
 - "domain 53 pass / 9 DB-gated skipped; metrics 21 pass (both runs exit 0)"
 - "observability builds against real @sentry/node 8.55 + posthog-node 4.18"
---

## Prompt

okey now start completing remaining tasks, tell me what you need from me i will do

## Response snapshot

Asked 3 gating decisions (deploy host → **Render**, monitoring → **Sentry+PostHog**,
content → **draft it**), then built every code-completable remainder:

- **T9.2** — `render.yaml` Blueprint (api/worker/Redis + native autoscaling) +
  `scripts/queue-autoscaler.ts` (queue-depth scaling via Render API w/ hysteresis)
  + `docs/runbooks/deployment.md`.
- **T10.5** — `monitoring/` as-code (alerts.yaml, Sentry rules + idempotent apply,
  PostHog insights + apply). Found the real gap: the T12.1 event taxonomy was
  almost entirely unwired and the telemetry sinks were never set. Wired
  `captureEvent` at onboarding/connector-install/sequence/payment/send/reply
  anchors (canonical names in `metrics/events.ts`), added a worker queue-depth
  heartbeat, and connected the sinks to Sentry+PostHog via new
  `observability/providers.ts initTelemetry()` (env-gated, no-op without keys),
  called from api+worker bootstraps.
- **T9.5** — k6 harness (`load-test/`, steady+spike, SLO thresholds).
- **T8.5** — `scripts/dr-restore-drill.sh` (Neon PITR branch + integrity + RTO).
- **T11.1** — `docs/integrations/marketplace-submission.md` kit.
- **T11.5** — `content/` DSO/AR articles + benchmarks.

## Outcome

- ✅ Impact: every remaining task now has its code/artifacts; what's left is
  running them (deploy, apply monitoring, run drills) or genuinely external
  (app-review, real-data retrain, publish content).
- 🧪 Tests: all packages build; api/worker/domain typecheck clean; domain 53 pass
  /9 skipped; metrics 21 pass. No test observes captureEvent (emits are safe).
- 📁 Files: see list. Added deps `@sentry/node`, `posthog-node` to observability.
- 🔁 Next (needs user/host): deploy Blueprint; run `monitoring/*/apply-*.sh` with
  tokens + set DSN/POSTHOG keys; run `dr-restore-drill.sh` + k6 vs staging;
  submit app-review; `train:risk:real` once data accrues; publish content.
- 🧠 Reflection: aligning to the existing `EVENTS` taxonomy (not inventing dotted
  names) kept dashboards, alerts, and emits on one canonical vocabulary.

## Evaluation notes (flywheel)

- Failure modes: initial emits used ad-hoc dotted names that didn't match the
  canonical `metrics/events.ts` taxonomy (fixed); dashboards initially referenced
  events that didn't fire yet (fixed by wiring them); telemetry sinks were never
  connected to providers (fixed via initTelemetry).
- Graders: build PASS, typecheck PASS, domain+metrics tests PASS (exit 0).
- Pre-existing (not mine): web `useRouter` NodeNext `.js` typecheck error in
  `partner-clients.tsx` (T11.2 session).
