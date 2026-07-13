# roger brain

Live demo: https://michiel-dk.github.io/roger-brain-demo/

An operations platform for independent restaurant groups: ingest sales and invoices, model the
recipe → ingredient → supplier graph, forecast demand, draft the supplier orders, ask the floor
one question a day, and check its own predictions against reality and publish the score.

This repo hosts a self-contained demo of the product: a scroll-driven walk-through
(`index.html`) plus a **static build of the real frontend** (`app/`) — the group brain map, the
FOH pill and the receiving pill, all working in the browser. The platform's source code is
private; this page is the showcase.

> Everything in the demo is a fictional tenant ("Burger House", three venues, world frozen at
> 2026-05-31). Every number shown is a real response from the production API over that seeded
> world — captured byte-for-byte, not composed. The write loops (counts, order sends,
> corrections, invoice upload) are simulated in-browser against the same rules the backend
> applies.

---

## What it does

One nightly loop per venue, one group brain over all of them:

- **Derive, don't count** — on-hand stock is derived continuously from sales (POS) and
  deliveries (invoices), so nobody types inventory into a form. The brain tracks its own
  **doubt** per ingredient (value at risk × days since a human confirmed) and spends one
  question a day where doubt × money is highest. The answer re-anchors the books.
- **Forecast-anchored ordering** — tomorrow's draft order comes from the venue's *learned*
  demand rhythm (weekday skew, hour-of-day, weather/event features) minus what's on hand,
  respecting pack sizes, priced from the latest invoice actually paid. Amending a line is
  itself a signal the brain learns from.
- **Invoices as data** — a photo at the back door is extracted line-by-line and echoed back
  verbatim. Prices land on the books and feed the order €, the overpay checks and food cost.
- **A brain per venue, exceptions up** — venues learn separately (they genuinely differ);
  the group tier sees the league table and only what deserves a human: band misses, overpays,
  drift.

## Self-grading loop

Every forecast ships with an 80%-confidence band and becomes a row in a decision ledger
alongside order drafts, drift flags and spot-checks. When reality arrives, each row is graded.
A replay harness re-runs the whole history and publishes a report card of past forecasts against
outcomes, including the worst miss and the measured band coverage. When measured coverage (~68%)
fell short of the claimed 80%, a conformal-style band-width fix was applied.

The ingest seams fail loud instead of degrading to valid-looking wrong values (a pre-commit
ratchet flags any new silent fallback at a data seam), and a derived stock figure is never
presented as a counted one.

## Architecture

- **Ingestion pipeline** — POS exports (SAF-T, SumUp, Zonesoft API), invoice OCR (LLM
  extraction behind a fail-loud seam), weather; idempotent re-runs.
- **Demand forecasting** — per-venue learned rhythms with a keep-the-winner forecaster and
  banded predictions; drift detection flags when a learned pattern stops matching reality.
- **Decision ledger + replay** — every material decision recorded with its rationale, graded
  against outcomes, replayable end-to-end to regenerate the report card from history.
- **Doubt-ranked counting** — value-at-risk × staleness picks the single count worth a human's
  minute; corrections propagate back through the derivation.
- **Multi-tenant isolation** — per-tenant hashed API keys; app-layer tenant scoping as the
  primary guard with Postgres RLS as defense-in-depth; one tenant's data can never reach
  another's queries or prompts.
- **The pill grammar** — the floor-facing UI is one typed-out question at a time (FOH pill and
  receiving pill share it): ask → number or voice → thanks → recede. The manager app is a live
  brain map where each venue's rooms open in a routed detail pane.
- **Jobs and deployment** — nightly jobs behind a channel-delivery seam, seeded demo deploy on
  Railway; test suite on the private repo.

## This demo, technically

The static build is the real React frontend with one seam swapped: the typed API client routes
every request to a fixture-seeded in-memory world instead of `fetch`. Reads replay captured API
responses; the five mutations mutate that world under the backend's rules, so the query-cache
invalidations that follow a count or a send observe genuinely changed state. Normal builds
tree-shake the demo layer away entirely.

## Tech

`Python` · `FastAPI` · `SQLAlchemy 2` / `Pydantic v2` · `PostgreSQL` (RLS) / `SQLite` ·
`React 19` / `TypeScript` / `TanStack Query` · `Claude` (invoice extraction) · `Railway`
