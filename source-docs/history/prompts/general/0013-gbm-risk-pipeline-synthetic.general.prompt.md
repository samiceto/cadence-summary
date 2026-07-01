---
id: 0013
title: Scaffold GBM risk pipeline (synthetic)
stage: green
date: 2026-06-25
surface: agent
model: claude-opus-4-8
feature: none
branch: main
user: samiullahmalik7474@gmail.com
command: general
labels: ["t6.1", "ml", "gbm", "scoring", "risk-model"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - packages/domain/src/ml/features.ts
 - packages/domain/src/ml/gbm.ts
 - packages/domain/src/ml/synthetic.ts
 - packages/domain/src/ml/scorer.ts
 - packages/domain/src/ml/train.ts
 - packages/domain/src/ml/model.json
 - packages/domain/src/ml/ml.test.ts
 - packages/domain/src/scoring.ts
 - packages/domain/src/index.ts
 - packages/domain/package.json
 - apps/worker/src/workers.ts
tests:
 - packages/domain/src/ml/ml.test.ts (9 pass)
 - "train:risk holdout — model AUC 0.756 vs heuristic 0.726; logLoss 0.55 vs 1.04; acc 0.698 vs 0.681 (base rate 0.648)"
---

## Prompt

scaffold the T6.1 training pipeline with synthetic data

## Response snapshot

Built a pure-TypeScript late-payment model pipeline (no Python/Docker), runnable
end-to-end today on synthetic data and ready to swap in real labelled history:

- **features.ts** — deterministic feature extraction shared by train + inference
  (log-compressed balance, history-rate with cold-start prior).
- **gbm.ts** — from-scratch gradient-boosted regression trees (MART, logistic
  loss); serializable to plain JSON so inference runs in-process. Split search
  optimized to O(n) per feature via running sums (the SSE sum-of-squares term
  cancels).
- **synthetic.ts** — seeded (mulberry32) dataset with a NONLINEAR latent risk +
  noise, plus AUC (tie-averaged) / logLoss / accuracy metrics.
- **train.ts** (`pnpm --filter @cadence/domain train:risk`) — generates data,
  trains, evaluates vs the heuristic on a held-out split, writes ml/model.json.
- **scorer.ts** — a `RiskScorer` (`makeGbmScorer` / `loadRiskScorer`) that drops
  in behind `computeRiskScore`: the model supplies the probability, the shared
  channel/timing policy does the rest. Falls back to the heuristic when no
  artifact is present.
- Wired `scoreOpenInvoices` to accept a scorer (default heuristic → no behaviour
  change) and the worker to load the GBM at startup; build copies model.json to
  dist.

Holdout: model beats heuristic (AUC 0.756 vs 0.726; logLoss 0.55 vs 1.04).

## Outcome

- ✅ Impact: T6.1 has a real, swappable model pipeline; remaining work is only
  retraining on real payment data (replace `generateDataset` with a DB loader).
- 🧪 Tests: 9 ML tests pass (generator determinism, separable-pattern learning,
  AUC edge cases, artifact load, scorer policy, heuristic fallback); domain
  typecheck + worker typecheck clean.
- 📁 Files: new `packages/domain/src/ml/*` + committed `model.json`; scoring.ts
  refactor; worker wiring.
- 🔁 Next prompts: write a `loadLabeledHistory(tx)` DB loader (paid-late labels)
  + a backtest harness; promote model version once it beats heuristic on real
  data.
- 🧠 Reflection: kept the channel/timing policy shared so only the probability
  source changes between heuristic and model — keeps escalation auditable.

## Evaluation notes (flywheel)

- Failure modes observed: O(n²) split search hung training (fixed with running
  sums); `noUncheckedIndexedAccess` required index assertions; tie-unaware AUC
  scored constant predictions 1.0 (fixed with average ranks).
- Graders run and results (PASS/FAIL): ml.test.ts PASS (9/9); holdout AUC PASS
  (model > heuristic).
- Prompt variant (if applicable): n/a
- Next experiment (smallest change to try): add monotonic constraint / isotonic
  calibration and compare logLoss on real data.
