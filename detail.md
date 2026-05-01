# StratForge Labs — MVP Architecture (Final)

**Independent open-source learning project.** Algorithmic trading research, backtesting, and (later) paper execution. **Not** affiliated with any commercial product, MetaTrader EA builder, broker, investment firm, or similarly named trading platform.

---

## 1. Project name and one-line description

**Name:** **StratForge Labs**

**One-liner:** A full-stack platform to load historical OHLCV data, define a Moving Average Crossover strategy, run backtests, and persist inspectable results (metrics, trades, equity curve) behind a FastAPI + PostgreSQL API with a React dashboard.

---

## 2. MVP goal

Ship a **credible portfolio-grade** system in ~3 months at ~2 hours/day: end-to-end flow from data → strategy config → backtest → stored results → UI charts/tables—**without** pretending to be a bank or a high-frequency trading stack.

---

## 3. What problem the MVP solves

Research teams need a **repeatable, inspectable** way to ask: “If we had traded this simple rule on this history, what would have happened?” The MVP answers that for **one** well-understood strategy (MA crossover), with **transparent** trades and risk-ish summary stats—not stock tips, not live trading.

---

## 4. Target user (simple words)

**Investment or research folks** who already think in backtests and drawdowns. They want to **configure** a strategy, **run** it on history, and **compare** outcomes (equity curve, trade list, basic stats)—not retail “signals” or social trading.

---

## 5. MVP scope

| Area | In scope |
|------|----------|
| Data | Load historical OHLCV via **CSV upload** and/or **small scripted fetch** (e.g. `yfinance`) into PostgreSQL |
| Strategy | **One** type: **Moving Average Crossover** (parameters: fast window, slow window, optional fee/slippage constants) |
| Engine | **Python** backtest package invoked **in-process** from FastAPI (same container/process as API for MVP) |
| Persistence | **PostgreSQL** for symbols, bars, strategies, runs, trades, equity points, metrics |
| API | **FastAPI** REST JSON |
| UI | **React + TypeScript + Vite + Tailwind + Recharts**: data upload/import status, strategy form, run backtest, results |
| Ops | **Docker Compose**: `api` + `db` + `frontend` (optional: `api` serves static in prod story; dev can split) |
| Docs | README + this architecture + API examples |

**Modular monolith:** one API service, one DB, one Python engine package—**not** microservices.

---

## 6. Explicitly out-of-scope (MVP)

- Real-money trading, broker connectivity, MetaTrader / MQL5  
- Crypto/options-specific product features (MVP is **generic equities-style** OHLCV; don’t build venue-specific stacks)  
- Multiple strategy families, ML predictors, hyperparameter optimization platforms  
- **Go** execution engine, **Redis**, **WebSockets**, **Prometheus/Grafana**  
- Full admin panel, multi-tenant user management  
- **User authentication** unless you expose the stack beyond localhost—default MVP assumes **local/trusted network**; add auth only when you actually deploy publicly  

---

## 7. High-level architecture diagram (text)

```text
┌─────────────────────────────────────────────────────────────────┐
│                     Browser (React + TS + Vite)                  │
│  Pages: Home, Data, Strategy, Backtest Detail                    │
└───────────────────────────────┬─────────────────────────────────┘
                                │ HTTP JSON (REST)
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    FastAPI Application (Python)                   │
│  Routers: health, symbols, bars, strategies, backtests            │
│  Services: orchestration, DB access, call into engine             │
└───────┬─────────────────────────────────────────────┬─────────────┘
        │ SQLAlchemy/SQLModel                          │
        ▼                                             │ import/call
┌──────────────────┐                         ┌─────────▼─────────────┐
│   PostgreSQL     │                         │  stratforge_engine    │
│  symbols         │                         │  (local package)      │
│  price_bars      │                         │  - load/normalize bars│
│  strategies      │                         │  - MA crossover       │
│  backtest_runs   │                         │  - backtest loop      │
│  backtest_trades │                         │  - metrics            │
│  equity_curve_   │                         └───────────────────────┘
│    points        │
│  backtest_       │
│    metrics       │
└──────────────────┘

Docker Compose: services `db`, `api`, `frontend` (dev: Vite dev server).
```

**Phase 2 (mention only):** Go **paper execution** worker, **Redis** pub/sub, **WebSockets** for live dashboard—**after** the MVP backtest story is boringly reliable.

---

## 8. Request/response system flow

```text
1. User uploads CSV OR triggers “import” for a symbol/date range
      → POST /api/symbols, POST /api/symbols/{id}/bars (bulk or batch)
      → API validates rows, upserts into price_bars

2. User creates MA Crossover strategy (name + fast + slow + optional fees)
      → POST /api/strategies
      → Row in strategies

3. User starts backtest
      → POST /api/backtests  { strategy_id, symbol_id, date range, capital }
      → API creates backtest_runs (status=queued|running)
      → API loads bars + strategy from DB
      → Engine runs; writes trades, equity_curve_points, backtest_metrics
      → API returns backtest_run id + summary

4. User views results
      → GET /api/backtests/{id}
      → GET /api/backtests/{id}/trades
      → GET /api/backtests/{id}/equity-curve
      → React charts/tables
```

**Trade-off:** synchronous backtest in request thread is OK for MVP if runs finish in seconds to a few minutes on sample data; if you outgrow that, move to background tasks **without** adding Redis—use `asyncio` + DB status polling or a simple in-process queue first.

---

## 9. Module breakdown

### 9.1 Frontend module

**Responsibilities:** Forms and visualization only—no quantitative logic.

- **API client** (`fetch` or axios): typed DTOs  
- **Pages:** Data (upload/import), Strategies (list/create), Backtests (list/detail)  
- **Components:** tables, equity line chart (Recharts), trade list, metric cards  
- **State:** local React state + light context or TanStack Query (optional); **avoid** heavy global state libraries for MVP  

### 9.2 FastAPI backend module

**Responsibilities:** Validation, persistence, orchestration.

- **Routers:** thin; delegate to services  
- **Models/ORM:** SQLAlchemy or SQLModel tables matching schema below  
- **Pydantic schemas:** request/response DTOs  
- **Services:** `BacktestService.run(backtest_id)` loads data, calls engine, writes results in a transaction  

### 9.3 Python backtesting engine module (`stratforge_engine`)

**Responsibilities:** Pure-ish functions over `pandas` DataFrames; **no FastAPI imports**.

- `data.normalize_bars(df)` — types, sorting, duplicate handling  
- `strategies.ma_crossover.generate_signals(bars, fast, slow)`  
- `backtest.run_long_only_equity(...)` — position sizing, fees, trade list, equity series  
- `metrics.compute(run_result)` — drawdown, win rate, etc.  

**Packaging:** `pip install -e ./packages/stratforge_engine` from backend Dockerfile, or monorepo subfolder on `PYTHONPATH`.

### 9.4 Database module

**Responsibilities:** Single PostgreSQL instance; migrations via Alembic (recommended) or create-all for earliest spike.

### 9.5 Documentation module

- `README.md` — positioning, disclaimer, quickstart  
- `detail.md` (this file) — MVP architecture  
- `docs/ARCHITECTURE.md` — shorter living architecture for contributors  

---

## 10. Data flow for running a backtest

1. **Input resolution:** `strategy_id` → row in `strategies` (type must be `ma_crossover` for MVP).  
2. **Bar selection:** Query `price_bars` for `symbol_id` between `start_date` and `end_date`, ordered by date.  
3. **Engine:** DataFrame → indicators → signals → portfolio simulation → list of trades + daily equity.  
4. **Persist:** Insert `backtest_trades`, `equity_curve_points`, `backtest_metrics`; update `backtest_runs.status`, `completed_at`.  
5. **Output:** API returns identifiers + headline metrics for the UI; detail endpoints stream paginated trades if needed.

---

## 11. Suggested folder structure

```text
stratforge/
├── docker-compose.yml
├── README.md
├── detail.md
├── docs/
│   └── ARCHITECTURE.md
├── packages/
│   └── stratforge_engine/
│       ├── pyproject.toml
│       ├── stratforge_engine/
│       │   ├── __init__.py
│       │   ├── data/
│       │   ├── strategies/
│       │   ├── backtest/
│       │   └── metrics/
│       └── tests/
├── backend/
│   ├── Dockerfile
│   ├── pyproject.toml
│   ├── alembic/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   ├── core/
│   │   ├── db/
│   │   ├── models/
│   │   ├── schemas/
│   │   └── services/
│   └── tests/
├── frontend/
│   ├── Dockerfile
│   ├── vite.config.ts
│   ├── package.json
│   └── src/
│       ├── pages/
│       ├── components/
│       ├── api/
│       └── types/
└── sample_data/
    └── README.txt
```

**Trade-off:** `packages/stratforge_engine` adds one small packaging step; payoff is clean separation and testability—worth it for a portfolio project.

---

## 12. Database schema (MVP only)

**Conventions:** `id` UUID primary keys; `created_at` timestamptz; dates as `date`; money as `numeric(20,6)` for equity and `numeric(20,8)` for prices—avoid float in DB for audited-looking portfolios.

### `symbols`

| Column | Type | Notes |
|--------|------|--------|
| id | UUID | PK |
| ticker | VARCHAR(32) | e.g. `AAPL` |
| exchange | VARCHAR(32) | nullable for MVP |
| currency | VARCHAR(8) | default `USD` |
| name | VARCHAR(128) | optional |
| created_at | TIMESTAMPTZ | |

Unique: `(ticker, exchange)` or just `ticker` if single-market MVP.

### `price_bars`

| Column | Type | Notes |
|--------|------|--------|
| id | UUID | PK |
| symbol_id | UUID | FK → symbols |
| bar_date | DATE | trading session date |
| open | NUMERIC(20,8) | |
| high | NUMERIC(20,8) | |
| low | NUMERIC(20,8) | |
| close | NUMERIC(20,8) | |
| volume | BIGINT | nullable if missing |
| source | VARCHAR(32) | `csv`, `yfinance`, etc. |
| created_at | TIMESTAMPTZ | |

Unique: `(symbol_id, bar_date)`.

### `strategies`

| Column | Type | Notes |
|--------|------|--------|
| id | UUID | PK |
| name | VARCHAR(128) | |
| strategy_type | VARCHAR(32) | MVP: `ma_crossover` only |
| parameters | JSONB | e.g. `{"fast": 10, "slow": 30, "fee_bps": 5}` |
| description | TEXT | optional |
| created_at | TIMESTAMPTZ | |
| updated_at | TIMESTAMPTZ | |

### `backtest_runs`

| Column | Type | Notes |
|--------|------|--------|
| id | UUID | PK |
| strategy_id | UUID | FK |
| symbol_id | UUID | FK |
| start_date | DATE | |
| end_date | DATE | |
| initial_capital | NUMERIC(20,6) | |
| parameters_snapshot | JSONB | frozen copy of strategy params |
| status | VARCHAR(16) | `pending`, `running`, `completed`, `failed` |
| error_message | TEXT | nullable |
| started_at | TIMESTAMPTZ | nullable |
| completed_at | TIMESTAMPTZ | nullable |
| created_at | TIMESTAMPTZ | |

### `backtest_metrics`

| Column | Type | Notes |
|--------|------|--------|
| backtest_run_id | UUID | PK, FK → backtest_runs |
| final_equity | NUMERIC(20,6) | |
| total_return_pct | NUMERIC(10,4) | |
| max_drawdown_pct | NUMERIC(10,4) | |
| num_trades | INTEGER | |
| win_rate_pct | NUMERIC(10,4) | winning closed trades / total closed |
| sharpe_ratio | NUMERIC(10,4) | optional; NaN → NULL |
| computed_at | TIMESTAMPTZ | |

### `backtest_trades`

| Column | Type | Notes |
|--------|------|--------|
| id | UUID | PK |
| backtest_run_id | UUID | FK |
| trade_index | INTEGER | 0-based order in run |
| entry_date | DATE | |
| exit_date | DATE | |
| side | VARCHAR(8) | MVP long-only: `long` |
| quantity | NUMERIC(20,8) | |
| entry_price | NUMERIC(20,8) | |
| exit_price | NUMERIC(20,8) | |
| pnl | NUMERIC(20,6) | realized P/L for the round trip |
| fees | NUMERIC(20,6) | |
| created_at | TIMESTAMPTZ | |

### `equity_curve_points`

| Column | Type | Notes |
|--------|------|--------|
| id | UUID | PK |
| backtest_run_id | UUID | FK |
| bar_date | DATE | |
| equity | NUMERIC(20,6) | mark-to-market or close-to-close per your engine convention—document the choice |
| created_at | TIMESTAMPTZ | |

Unique: `(backtest_run_id, bar_date)`.

---

## 13. API endpoints (MVP only)

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health` | liveness |
| GET | `/api/symbols` | list symbols |
| POST | `/api/symbols` | create symbol metadata |
| POST | `/api/symbols/{symbol_id}/bars` | bulk create bars (JSON array) |
| GET | `/api/symbols/{symbol_id}/bars` | optional: debug/preview |
| GET | `/api/strategies` | list |
| POST | `/api/strategies` | create MA strategy |
| GET | `/api/strategies/{id}` | detail |
| PUT | `/api/strategies/{id}` | update parameters |
| DELETE | `/api/strategies/{id}` | optional |
| GET | `/api/backtests` | list runs |
| POST | `/api/backtests` | **create and run** backtest |
| GET | `/api/backtests/{id}` | run + metrics summary |
| GET | `/api/backtests/{id}/trades` | trades |
| GET | `/api/backtests/{id}/equity-curve` | equity series |

**Out of scope:** WebSocket routes, admin namespaces, paper trading routes.

### Example: `POST /api/backtests`

**Request:**

```json
{
  "strategy_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "symbol_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "start_date": "2018-01-03",
  "end_date": "2023-12-29",
  "initial_capital": 100000.0
}
```

**Response (200):**

```json
{
  "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "strategy_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "symbol_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "start_date": "2018-01-03",
  "end_date": "2023-12-29",
  "initial_capital": "100000.000000",
  "status": "completed",
  "metrics": {
    "final_equity": "118500.250000",
    "total_return_pct": "18.5003",
    "max_drawdown_pct": "7.2000",
    "num_trades": 42,
    "win_rate_pct": "56.0000",
    "sharpe_ratio": "1.3400"
  },
  "error_message": null,
  "started_at": "2026-05-01T12:00:05Z",
  "completed_at": "2026-05-01T12:00:07Z"
}
```

**Error example (400):**

```json
{
  "detail": "No price bars for symbol in the requested date range."
}
```

---

## 14. Frontend pages (MVP only)

| Route | Purpose |
|-------|---------|
| `/` | Overview + links |
| `/data` | Symbol list; CSV upload; optional “fetch demo data” |
| `/strategies` | List + link to create |
| `/strategies/new` | MA crossover form |
| `/strategies/:id` | Edit parameters (simple) |
| `/backtests` | List past runs |
| `/backtests/:id` | Metrics cards, equity chart, trades table |

**No** login, paper trading, or admin consoles in MVP.

---

## 15. Backtesting engine design

**Assumptions (document in code):** Long-only, one open position at a time, trade on **bar close** when crossover fires, execute at next bar open (or same bar close—pick **one** and stick to it; next-open is slightly more honest for signal-based rules).

**Core loop:**

1. Compute fast MA and slow MA on `close`.  
2. Signal: golden cross → enter; death cross → exit (define strict inequality / tie-break).  
3. Position sizing: use **all equity** or fixed fraction—MVP: deploy **100% of equity** on each entry for simplicity, with optional min notional guard.  
4. Apply **fees** as round-trip bps on entry+exit notional.  
5. Emit **trades** and **equity** time series aligned to `bar_date`.

**Testing:** synthetic trending and sideways CSV fixtures where trade count is known by hand.

---

## 16. Moving Average Crossover strategy design

**Parameters (JSON):**

```json
{
  "fast_window": 10,
  "slow_window": 30,
  "fee_bps": 5
}
```

**Rules:**

- `fast_window < slow_window` (validate in API).  
- **Buy** when `MA_fast` crosses **above** `MA_slow` from previous bar.  
- **Sell** when `MA_fast` crosses **below** `MA_slow`.  
- First valid signals after both MAs are defined (drop warmup period).  

**Overengineering check:** Do not build a plugin registry for MVP—**one** function module is enough.

---

## 17. Metrics to calculate in MVP

| Metric | Definition (practical) |
|--------|-------------------------|
| Final equity | Last point of equity curve |
| Total return % | `(final / initial - 1) * 100` |
| Max drawdown % | Max peak-to-trough on equity curve, as positive % |
| Num trades | Count of **round trips** |
| Win rate % | Wins / closed trades |
| Sharpe (optional) | Mean daily return / std dev of daily returns × sqrt(252); if std=0 → NULL |

**Deferred:** Calmar, Sortino, exposure time, turnover—nice for v2.

---

## 18. Error handling plan

| Layer | Behavior |
|-------|----------|
| API validation | 422 with field errors (Pydantic) |
| Missing data | 400 with clear message |
| Engine exceptions | 500 in dev with traceback log; prod: generic message + `error_message` on run row |
| DB | Roll back partial writes; `backtest_runs.status=failed` |

Log structured JSON lines from FastAPI (`uvicorn` + standard logging).

---

## 19. Testing plan

| Layer | Tests |
|-------|--------|
| `stratforge_engine` | Unit tests: signals, metrics, small deterministic backtest |
| Backend | API tests with TestClient + SQLite **or** ephemeral Postgres (preferred: real Postgres in CI service) |
| Frontend | Smoke tests optional; prioritize engine correctness |

**Goal:** A reviewer can trust the MA logic without clicking the UI.

---

## 20. Docker Compose setup plan

**Services:**

- `db`: `postgres:16`, volume for data, env `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`  
- `api`: build `./backend`, depends_on `db`, env `DATABASE_URL`  
- `frontend`: build `./frontend` **or** dev command `npm run dev -- --host` with volume mount  

**Volumes:** named volume for Postgres only.

**No** Redis, no extra observability containers in MVP.

---

## 21. 3-month beginner-friendly roadmap

**Month 1 — Truth in Python**

1. Repo + `stratforge_engine` skeleton + pytest  
2. CSV → DataFrame loader + validation  
3. MA crossover signals + tests  
4. Long-only backtest loop + trade ledger  
5. Metrics: return, drawdown, win rate  

**Month 2 — Persistence + API**

6. Docker Compose Postgres  
7. SQLModel models + Alembic migrations for all MVP tables  
8. FastAPI: symbols + bars import endpoints  
9. FastAPI: strategies CRUD  
10. `POST /api/backtests` wired to engine + persistence  

**Month 3 — UI + polish**

11. Vite React app: pages and Recharts equity  
12. Trades table + run history  
13. README quickstart + sample CSV  
14. CI: `pytest` + `ruff`/`black` (optional) + frontend `npm run build`  

**Rule:** If a task doesn’t support sections 5–6, it waits.

---

## 22. MVP success criteria

- Import at least one symbol with multi-year daily bars.  
- Create and save an MA strategy from the UI.  
- Run a backtest and **reload** results after API restart (PostgreSQL).  
- Show equity curve + trades + headline metrics without manual DB inspection.  
- Engine tests cover edge cases (no trades, one trade, insufficient warmup).  
- A hiring manager can follow README and run Compose locally in **one** documented path.

---

## 23. Resume bullets (after MVP)

- Designed and implemented a **modular monolith** research stack: **FastAPI + PostgreSQL + React** for strategy configuration and backtest analytics.  
- Built a **pandas-based** event-driven backtester with **trade-level auditability** and performance metrics (return, drawdown, win rate).  
- Packaged quantitative logic as an importable **Python engine** with **pytest** coverage and **Docker Compose** local orchestration.  

---

## 24. README positioning

Lead with **research credibility**, not hype:

- **What it is:** Open-source **StratForge Labs** — historical data, MA crossover backtests, persisted results, simple UI.  
- **What it is not:** Not brokerage, not investment advice, not a prediction product.  
- **How to run:** Docker Compose + sample data.  
- **Phase 2 teaser:** Go paper execution + Redis/WebSocket **only** as future work—don’t imply they exist today.

---

## 25. Naming disclaimer (open source)

Include prominently in README and repo About text:

> **StratForge Labs** is an independent open-source learning project for algorithmic trading research, backtesting, and (future) paper execution. It is **not** affiliated with any commercial product, MetaTrader EA builder, broker, investment firm, or trading platform that uses a similar name.

---

## Appendix — Trade-offs summary

| Decision | Why | Cost |
|----------|-----|------|
| Monolith API + in-process engine | Fastest path to correctness | Less horizontal scale story |
| PostgreSQL from day one | Portfolio realism; survives restarts | Slightly slower than SQLite-only spike |
| One strategy | Learn end-to-end without framework churn | Less “feature demo” breadth |
| CSV/yfinance ingestion | Cheap data for learning | Not production market data quality |
| No auth locally | Saves weeks | Must add before any public deployment |

This document is the **MVP contract**. Anything in legacy docs that contradicts sections 5–6 is **deprecated** until Phase 2.
