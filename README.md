# roger brain — an ops brain that grades its own work

**Live demo:** https://michiel-dk.github.io/roger-brain-demo/

A multi-tenant operations assistant for restaurant groups that forecasts, orders, and then
**grades its own forecasts** against what actually happened. This repo is the public demo — a
self-contained `index.html` running an in-browser simulation on synthetic data. It's a showcase,
not production code; the platform source is private. Methods below.

## What it does

- **Derive, don't count** — stock on-hand derives continuously from sales + deliveries instead of
  manual entry; the system quantifies uncertainty per ingredient and asks for verification only
  where risk is highest.
- **Forecast-anchored ordering** — daily orders draft from learned day-of-week demand minus
  available stock, respecting pack sizes and live pricing.
- **Invoices as data** — invoice photos are extracted line-by-line and feed food-cost and
  ordering calculations.

## Methods

**Forecasting & uncertainty**
- Per-tenant demand models (learned weekday/seasonal rhythms) with **drift detection** — recent
  window vs baseline, flagged past a threshold; per-venue stock and par levels layered on top.
- Predictions carry **80% confidence bands** (prediction intervals): an analytic Student-t
  interval that **upgrades to a split-conformal calibration** from each tenant's own graded
  residuals once enough accrue — not bare point estimates.
- Uncertainty quantification drives an **active-verification queue** — a doubt-ranked list
  (staleness × volatility × value-at-risk) that goes quiet when everything's fresh, instead of nagging.

**Self-grading / evaluation**
- Every decision is written to an append-only **decision ledger**; history is **replayed** (no
  look-ahead, idempotent) to produce report cards — predicted vs actual, worst misses,
  band-coverage accuracy.
- Backtesting-by-replay is an eval loop the system runs on itself with **no external labels** —
  ground truth is what later happened (actual revenue, owner spot-counts).
- Guards against the degenerate wide-band forecaster (coverage 1.0 but useless) by scoring
  **band sharpness** alongside coverage.

**LLM & structured extraction**
- Invoice **OCR / line-item extraction via a cheap vision LLM** (Claude Haiku or Gemini Flash —
  the cheap tier by design; Gemini preferred when a key is configured), returning typed,
  structured Pydantic records rather than free text; offline stub fallback when no key is set.
- A **typed, question-based** floor UI — the brain asks one bounded, structured question at a time
  ("how many X do you count right now?"), so answers are auditable, not open chat.

**Multi-tenancy**
- Per-venue isolation with a group tier that surfaces only high-priority exceptions (band misses,
  overpays, drift).
- Isolation enforced by **hashed API keys** (SHA-256, plaintext shown once) + **app-layer tenant
  scoping** on every query. A Postgres row-level-security path is scaffolded (session-scoped tenant
  var) for the managed-DB deployment; app-layer scoping is the enforcing layer today.

## Engineering

- **Stack:** Python · FastAPI · SQLAlchemy 2 / Pydantic v2 · PostgreSQL / SQLite · React 19 /
  TypeScript · TanStack Query · Anthropic (Claude) + Gemini · Railway
- **Typed end to end** — Pydantic v2 on the backend, TypeScript types generated from the FastAPI
  OpenAPI schema — and covered by **567 passing tests**, including no-look-ahead and idempotence
  guards on the replay harness.
- Cheap derivations and structured questions carry the load; the LLM is reserved for what only it
  can do (reading an invoice photo) — a deliberate **accuracy vs latency vs cost** split.
