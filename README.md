# StratForge

StratForge combines Python-based quantitative research with Go-based execution infrastructure to test, simulate, and evaluate algorithmic trading strategies.

This project is built as a serious fintech/backend engineering project, not as a simple demo app.

---

## Why StratForge?

Trading strategies should not be trusted just because they sound logical.

Before using any strategy in real markets, we need to test:

- how it performed historically
- how much risk it carried
- how large the drawdowns were
- how consistent the returns were
- whether it beats a simple buy-and-hold approach

StratForge provides a robust, dual-language system to answer these questions reliably and quickly.

---

## Hybrid Architecture

StratForge uses the right tool for the right job:

- **Python (Quant Research Layer):** Handles strategy logic, backtesting, metrics, data analysis, reports, and notebooks. It leverages `pandas`, `numpy`, `scipy`, and `matplotlib` for unparalleled research speed.
- **Go (Systems / Execution Layer):** Handles market data ingestion, real-time tick streaming, order execution simulation, API gateway, concurrent workers, and latency-sensitive services. It offers fast concurrency, clean binaries, and strong backend reliability.

### System Flow

```text
Python Research Engine
        ↓
Generates strategy signals
        ↓
Go Execution Engine
        ↓
Simulates orders / streaming / risk checks
        ↓
Stores results
        ↓
Python Metrics Engine
```

---

## Development Roadmap & Philosophy

**Strict Engineering Order:** We build the Python research logic first. 
1. CSV Data Loader
2. Moving Average Strategy
3. Backtest Loop
4. Metrics

*Why?* If we start with Go first, we waste time building infrastructure before the core trading logic even exists. Once the Python core is proven, we wrap it in the highly concurrent Go execution layer.

---

## Core Features

- Historical market data loading
- Strategy engine (Moving Average Crossover, RSI, Breakout)
- Buy/Sell/Hold signal generation
- Backtesting engine
- Order execution simulation (Slippage, Fees, Fills)
- Portfolio tracking
- Trade history
- Risk-adjusted performance metrics

---

## Initial Strategy

The first strategy planned for this project is the **Moving Average Crossover Strategy**.

Logic:
```text
Short moving average crosses above long moving average → BUY
Short moving average crosses below long moving average → SELL
Otherwise → HOLD
```

This strategy is simple enough to implement first, but strong enough to build the full system pipeline.

---

## Planned Tech Stack

- **Python:** `pandas`, `numpy`, `scipy`, `matplotlib`, `pytest`
- **Go:** `goroutines`, `channels`, standard library for high-performance infra
- **Future:** FastAPI / gRPC, PostgreSQL, React, Docker, Prometheus, Grafana

---

## Suggested Project Structure

```text
stratforge/
├── python-engine/
│   ├── stratforge/
│   ├── tests/
│   └── requirements.txt
│
├── go-engine/
│   ├── cmd/
│   ├── internal/
│   └── go.mod
│
├── docs/
│   └── ARCHITECTURE.md
│
├── README.md
└── docker-compose.yml
```

*(See `docs/ARCHITECTURE.md` for the full detailed structure.)*

---

## Example Output

A completed backtest should produce output like:

```text
Initial Capital: ₹100000
Final Capital: ₹118500
Total Return: 18.5%
Max Drawdown: 7.2%
Win Rate: 56%
Sharpe Ratio: 1.34
Total Trades: 42
```

---

## What This Project Demonstrates

This project demonstrates:
* fintech domain interest
* polyglot backend engineering (Python + Go)
* high-performance systems thinking
* data processing & analysis
* clean architecture & separation of concerns
* performance evaluation

---

## Status

Project is in early development.

Current goal:
```text
Build the first working backtesting pipeline in Python using the Moving Average Crossover strategy, before adding the Go execution engine.
```

---

## Important Note

This project is for learning and research purposes only. It does not provide financial advice and should not be used for live trading without proper risk controls, testing, and validation.
