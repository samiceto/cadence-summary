# Load testing (T9.5)

Capacity + latency validation for the Cadence API using [k6](https://k6.io).
Run before a GA / marketing push and after any change to a hot read path.

## Install k6

```bash
# macOS
brew install k6
# Debian/Ubuntu/WSL
sudo gpg -k && \
  sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg \
    --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69 && \
  echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" \
    | sudo tee /etc/apt/sources.list.d/k6.list && \
  sudo apt-get update && sudo apt-get install k6
# or: docker run --rm -i grafana/k6 run - < load-test/api-load.js
```

## Run

> ⚠️ Point this at **staging**, never production — it generates real load and the
> dev-auth headers only work on non-prod deploys.

```bash
# Against a deployed staging API, using a seeded tenant + dev headers:
BASE_URL=https://cadence-api-staging.onrender.com \
TENANT_ID=<seeded-tenant-uuid> \
k6 run load-test/api-load.js

# Or with a real JWT instead of dev headers:
BASE_URL=https://cadence-api-staging.onrender.com \
TENANT_ID=<uuid> AUTH_BEARER=<clerk-jwt> \
k6 run load-test/api-load.js
```

## Profile

`api-load.js` runs two scenarios concurrently:
- **steady** — ramps 0→50 VUs over ~3 min on the read paths (baseline).
- **spike** — a 6× arrival-rate burst (50→300 rps) layered on top at T+1m,
  simulating a sync/score fan-out.

## Pass/fail (SLO thresholds — the run exits non-zero if breached)

| Metric | Threshold |
|--------|-----------|
| Error rate (`http_req_failed`) | < 1% |
| Latency p95 (all) | < 500 ms |
| Latency p99 (all) | < 1.2 s |
| `/health` p95 | < 150 ms |
| `/metrics/*` p95 | < 600 ms |

## Interpreting results

- **All thresholds green** → capacity is adequate at the tested instance count;
  record the result in the release notes.
- **Latency breaches but errors low** → scale up (`render.yaml` `maxInstances`)
  or check DB plans: run `EXPLAIN (ANALYZE, BUFFERS)` on the slow metric query
  and confirm it uses the 0008 indexes (`idx_invoices_tenant_customer`,
  `idx_consent_latest`, `idx_risk_scores_invoice_recent`). A seq scan within a
  tenant partition = a missing/unused index.
- **Errors climb under spike** → inspect Sentry for the error class; check Redis
  (`noeviction`) and DB connection-pool saturation (Neon pooled URL).

## What to test next (not yet automated)

- The **write** path (ingest/score/send) under load — needs a seeded tenant with
  a connector + a fake comms transport so the test doesn't actually send.
- Data **archival** policy for closed invoices/messages beyond N months (the
  remaining half of T9.5) — define retention, then a batched archival job.
