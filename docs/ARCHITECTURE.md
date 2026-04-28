# StratForge Architecture

## 1. Project Vision

StratForge is a hybrid Python + Go quantitative research and execution simulation engine.

Its purpose is to help developers and researchers test trading ideas before risking real money. The system takes historical market data, applies trading strategies, simulates order execution, tracks portfolio performance, and generates useful performance metrics.

This is not a simple CRUD app. This project is designed as a backend-heavy financial engineering system.

---

## 2. Core Problem

Before deploying any trading strategy, we need to answer:

- Would this strategy have worked historically?
- How risky is the strategy?
- How much capital drawdown can happen?
- How often does the strategy win or lose?
- Is the strategy better than simply holding the asset?

StratForge solves this by providing a robust backtesting and research pipeline with highly reliable execution simulation.

---

## 3. Hybrid Architecture

We use a language-specific approach where each layer plays to its strengths.

### Python = Quant Research Layer
**Components:**
- Strategy logic
- Backtesting
- Metrics engine
- Data analysis & reports
- Jupyter Notebooks

**Why Python?**
- Ecosystem: `pandas`, `numpy`, `scipy`, `matplotlib`
- Speed of research and iteration
- Standard language for quant finance and machine learning

### Go = Systems / Execution Layer
**Components:**
- Market data ingestion
- Real-time tick streaming
- Order execution simulator
- API gateway
- Concurrent workers
- Latency-sensitive services

**Why Go?**
- Fast, predictable concurrency
- Clean, standalone binaries
- Strong backend systems signal
- Better for high-performance infra and reliability

---

## 4. High-Level System Flow

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

## 5. Main Components

### 5.1 Data Layer (Go + Python)
- **Go** handles real-time market data ingestion and streaming, normalizing data and writing it to storage/queues.
- **Python** loads historical CSV/DB data for backtesting using `pandas`.

### 5.2 Strategy Engine (Python)
The Python engine evaluates trading logic against data and returns:
```text
BUY
SELL
HOLD
```

### 5.3 Execution Simulator (Go)
Mimics real-world order execution efficiently:
- Market/Limit orders
- Brokerage fees & slippage simulation
- Stop-loss and Take-profit orders

### 5.4 Risk Management Layer (Go)
Before an order executes, Go verifies:
- Available capital
- Maximum drawdown limits
- Portfolio-level risk checks

### 5.5 Portfolio Tracker & Metrics Engine (Python)
Evaluates execution results:
- Total Return & Win Rate
- Max Drawdown
- Sharpe Ratio
- Generates equity curves and dashboards

---

## 6. Suggested Folder Structure

```text
stratforge/
├── python-engine/
│   ├── stratforge/
│   │   ├── data/
│   │   ├── strategies/
│   │   ├── backtesting/
│   │   ├── metrics/
│   │   └── reports/
│   ├── tests/
│   └── requirements.txt
│
├── go-engine/
│   ├── cmd/
│   │   └── executor/
│   ├── internal/
│   │   ├── stream/
│   │   ├── execution/
│   │   ├── risk/
│   │   └── models/
│   └── go.mod
│
├── docs/
│   └── ARCHITECTURE.md
│
├── README.md
└── docker-compose.yml
```

---

## 7. Data Model

### Candle / Tick
```text
timestamp, open, high, low, close, volume
```

### Signal
```text
timestamp, symbol, action: BUY | SELL | HOLD, price, reason
```

### Order
```text
timestamp, symbol, side: BUY | SELL, quantity, price, status
```

### Trade
```text
entry_time, exit_time, entry_price, exit_price, quantity, profit_loss, return_percentage
```

---

## 8. Development Roadmap & Engineering Priority

### Strict Advice
**Do Python first.**
Build: CSV data loader → moving average strategy → backtest → metrics.
Then add Go.
If you start with Go first, you’ll waste time building infra before the trading logic exists. That is wrong engineering order.

### Phase 1 — Python Core Engine
* Create data loader
* Add moving average strategy
* Build backtesting loop
* Generate basic metrics

### Phase 2 — Go Execution Infra
* Add Go execution engine
* Connect Python signals to Go simulator
* Track portfolio state in Go
* Save executed trades for Python metrics

### Phase 3 — Realism & Scale
* Streaming data ingestion
* Real-time risk checks
* Slippage & brokerage models

---

## 9. What Makes This Project Strong

This project is valuable because it demonstrates:
* financial domain understanding
* polyglot backend architecture (Python + Go)
* systems engineering (concurrency, streams, latency)
* data processing & quantitative analysis
* clean interface boundaries
