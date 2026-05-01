# StratForge Labs

**StratForge Labs** is an independent open-source learning project for **algorithmic trading research, backtesting, and (later) paper execution**. It is **not** affiliated with any commercial product, MetaTrader EA builder, broker, investment firm, or trading platform that uses a similar name.

This repository targets a **serious portfolio narrative** for backend and full-stack roles in investment tech: clean APIs, persisted research artifacts, and inspectable backtests—not stock tips or retail “signals.”

---

## What it does (MVP)

- Load historical **OHLCV** data (CSV and/or simple fetch) into **PostgreSQL**
- Define **one** strategy type: **Moving Average Crossover**
- Run a **Python** backtest (pandas/numpy) behind **FastAPI**
- Persist **metrics**, **trades**, and **equity curve** points
- Explore results in a **React + TypeScript + Vite + Tailwind + Recharts** UI
- Run locally with **Docker Compose**

**Not in MVP:** real-money trading, broker integration, Go execution engine, Redis, WebSockets, multi-strategy frameworks, or heavy observability stacks. Those belong in **Phase 2** after the backtest path is solid.

---

## Why backtests matter

Strategies should not be trusted because they sound clever. Before any real deployment, research teams need to see **historical P/L**, **drawdowns**, **trade counts**, and **win rates** on transparent, reproducible runs. StratForge Labs is a small system for asking that question for a **simple, well-understood** rule on **your** loaded history.

---

## Architecture (MVP)

```text
React UI  →  FastAPI  →  PostgreSQL
                ↓
         stratforge_engine (Python package)
```

Full specification: **[detail.md](./detail.md)**. Contributor-oriented summary: **[docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)**.

---

## Tech stack (MVP)

| Layer | Stack |
|--------|--------|
| Frontend | React, TypeScript, Tailwind CSS, Recharts, Vite |
| Backend | FastAPI, Pydantic, SQLAlchemy or SQLModel, Alembic |
| Engine | Python, pandas, numpy, pytest |
| Database | PostgreSQL |
| Local dev | Docker Compose |

---

## Status

Early development. Current focus: **Python backtest engine + schema + API + minimal UI** per the MVP roadmap in `detail.md`.

---

## Disclaimer

This project is for **learning and research** only. It does not provide financial advice. Do not use it for live trading without independent validation, compliance, and risk controls.
