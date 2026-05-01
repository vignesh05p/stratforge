# StratForge Labs — Architecture (MVP)

This document is the **short, maintainer-facing** view of the MVP. The full contract (schema, endpoints, roadmap, examples) lives in **[detail.md](../detail.md)** at the repo root.

---

## Purpose

A **modular monolith** for research-backed **Moving Average Crossover** backtests: ingest OHLCV, configure a strategy, run a Python engine, persist runs/trades/equity/metrics, visualize in React.

---

## Containers (Docker Compose)

| Service | Role |
|---------|------|
| `db` | PostgreSQL 16 |
| `api` | FastAPI app; imports `stratforge_engine` |
| `frontend` | Vite dev server or static build |

No Redis, no Go service, no Prometheus in MVP.

---

## Logical layers

```text
┌─────────────┐     REST/JSON      ┌─────────────┐
│   React     │ ◄──────────────► │   FastAPI   │
└─────────────┘                  └──────┬──────┘
                                      │ SQL
                                      ▼
                               ┌─────────────┐
                               │ PostgreSQL  │
                               └─────────────┘
                                      ▲
                                      │ read bars / write results
                               ┌──────┴──────┐
                               │ stratforge │
                               │   _engine  │
                               └─────────────┘
```

---

## Python engine boundary

- **Package:** `packages/stratforge_engine` (install editable in `backend` image).
- **Rule:** No FastAPI imports inside the engine; pandas in, structured results out.
- **Called from:** `BacktestService` (or equivalent) in the API after loading rows from the DB.

---

## Core tables (names only)

- `symbols` — tradable identifiers  
- `price_bars` — OHLCV by `symbol_id` + `bar_date`  
- `strategies` — `strategy_type = ma_crossover`, `parameters` JSONB  
- `backtest_runs` — date range, capital, status, error  
- `backtest_metrics` — 1:1 with run  
- `backtest_trades` — round trips  
- `equity_curve_points` — daily (or bar) equity series  

Column-level spec: **[detail.md](../detail.md)** §12.

---

## API surface (MVP)

REST resources: `symbols`, `symbols/{id}/bars`, `strategies`, `backtests`, `backtests/{id}/trades`, `backtests/{id}/equity-curve`.

Example JSON for `POST /api/backtests`: **[detail.md](../detail.md)** §13.

---

## Phase 2 (non-goals for MVP)

- **Go** paper execution worker  
- **Redis** + **WebSockets** for live dashboards  
- Broker connectivity and real-money paths  

---

## Related docs

- **[detail.md](../detail.md)** — full MVP architecture (25 sections)  
- **[README.md](../README.md)** — positioning, disclaimer, quick context  
