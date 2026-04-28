# StratForge Architecture

## 1. Project Vision

StratForge is a quantitative trading research engine.

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

StratForge solves this by providing a backtesting and research pipeline.

---

## 3. High-Level System Flow

```text
Historical Market Data
        ↓
Data Loader
        ↓
Strategy Engine
        ↓
Signal Generator
        ↓
Execution Simulator
        ↓
Portfolio Tracker
        ↓
Metrics Engine
        ↓
Report / Dashboard
```

---

## 4. Main Components

### 4.1 Data Layer

The data layer is responsible for loading and validating market data.

Supported initial format:

```text
timestamp, open, high, low, close, volume
```

Responsibilities:

* Load CSV market data
* Validate missing values
* Sort data by timestamp
* Normalize column names
* Convert raw data into internal candle/tick models

Future improvements:

* PostgreSQL storage
* Market data API integration
* Multi-symbol data support
* Time-series indexing

---

### 4.2 Strategy Engine

The strategy engine contains trading logic.

Each strategy receives market data and returns a signal.

Possible signals:

```text
BUY
SELL
HOLD
```

Initial strategies:

* Moving Average Crossover
* RSI Mean Reversion
* Breakout Strategy

Strategy interface:

```text
Input: market candles
Output: trading signal
```

The goal is to make strategies pluggable so new strategies can be added without modifying the core engine.

---

### 4.3 Signal Generator

The signal generator converts strategy output into actionable trading instructions.

Example:

```text
Strategy says BUY
↓
Signal Generator checks current position
↓
Creates BUY signal if allowed
```

Responsibilities:

* Avoid duplicate buy/sell signals
* Validate strategy output
* Pass valid signals to execution layer

---

### 4.4 Execution Simulator

The execution simulator mimics real-world order execution.

Initial version:

* Market order simulation
* Buy/sell at close price
* Track quantity and capital

Later improvements:

* Brokerage fees
* Slippage
* Partial fills
* Stop-loss orders
* Take-profit orders
* Latency simulation

This component is important because real trading is not just about generating signals. Execution quality directly affects returns.

---

### 4.5 Portfolio Tracker

The portfolio tracker maintains the state of capital and positions.

Responsibilities:

* Track available cash
* Track open positions
* Track realized and unrealized profit/loss
* Maintain trade history
* Calculate portfolio value over time

Example state:

```text
cash: 85000
position: 10 shares
entry_price: 1500
portfolio_value: 100000
```

---

### 4.6 Metrics Engine

The metrics engine evaluates strategy performance.

Initial metrics:

* Total Return
* Win Rate
* Max Drawdown
* Sharpe Ratio
* Number of Trades
* Profit Factor

These metrics help compare different strategies objectively.

A strategy is not good just because it made profit. It must also control risk.

---

## 5. Suggested Folder Structure

```text
stratforge/
│
├── data/
│   └── sample_data.csv
│
├── stratforge/
│   ├── __init__.py
│   │
│   ├── data/
│   │   ├── loader.py
│   │   └── validator.py
│   │
│   ├── strategies/
│   │   ├── base.py
│   │   ├── moving_average.py
│   │   └── rsi.py
│   │
│   ├── engine/
│   │   ├── backtester.py
│   │   └── signal.py
│   │
│   ├── execution/
│   │   ├── order.py
│   │   └── simulator.py
│   │
│   ├── portfolio/
│   │   └── portfolio.py
│   │
│   ├── metrics/
│   │   └── performance.py
│   │
│   └── reports/
│       └── report_generator.py
│
├── tests/
│   ├── test_data_loader.py
│   ├── test_strategy.py
│   └── test_backtester.py
│
├── README.md
├── ARCHITECTURE.md
├── requirements.txt
└── main.py
```

---

## 6. Data Model

### Candle

```text
timestamp
open
high
low
close
volume
```

### Signal

```text
timestamp
symbol
action: BUY | SELL | HOLD
price
reason
```

### Order

```text
timestamp
symbol
side: BUY | SELL
quantity
price
status
```

### Trade

```text
entry_time
exit_time
entry_price
exit_price
quantity
profit_loss
return_percentage
```

---

## 7. Backtesting Flow

Detailed flow:

```text
1. Load historical data
2. Validate and clean data
3. Initialize strategy
4. Initialize portfolio with starting capital
5. Iterate through each candle
6. Strategy generates signal
7. Execution simulator processes signal
8. Portfolio updates position and cash
9. Store trade and equity curve
10. Metrics engine calculates final performance
11. Generate report
```

---

## 8. Example Strategy: Moving Average Crossover

Logic:

```text
short_moving_average > long_moving_average → BUY
short_moving_average < long_moving_average → SELL
otherwise → HOLD
```

Purpose:

This is a simple but useful first strategy because it teaches:

* trend following
* signal generation
* position management
* false signals
* drawdown behavior

---

## 9. Risk Management Layer

Initial risk controls:

* Do not invest more than available capital
* Do not buy if already in position
* Do not sell if no open position
* Fixed position sizing

Future improvements:

* Stop-loss
* Maximum drawdown limit
* Risk per trade
* Position sizing based on volatility
* Portfolio-level risk checks

---

## 10. API Layer — Future Scope

After the core engine is stable, expose it through FastAPI.

Possible endpoints:

```text
POST /backtest
GET /strategies
GET /backtests/{id}
GET /reports/{id}
```

This allows the system to later support a frontend dashboard.

---

## 11. Dashboard — Future Scope

A React dashboard can show:

* Equity curve
* Trade history
* Strategy comparison
* Drawdown chart
* Metrics summary
* Parameter tuning results

Do not build the dashboard first. Build the engine first.

---

## 12. Engineering Principles

This project should follow these principles:

* Separate strategy logic from execution logic
* Keep core engine testable
* Avoid hardcoded strategy behavior
* Use clean interfaces
* Write unit tests for each component
* Keep data validation strict
* Prefer correctness before optimization

---

## 13. What Makes This Project Strong

This project is valuable because it demonstrates:

* financial domain understanding
* backend architecture
* data processing
* simulation logic
* performance evaluation
* clean software design

For a fintech or investment engineering role, this project is much stronger than a generic CRUD app.
