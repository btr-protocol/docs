---
title: "Quant / Researcher Best Practices"
description: "## Quick Reference"
audience: tech
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# Quant / Researcher Best Practices

**Focus**: Market Making + DeFi AMMs + Linear Algebra + ML + Python (UV) + Zig

---

## Quick Reference

| Area | Standard |
|------|----------|
| **Prototyping** | Python with UV package manager |
| **Numerical Computing** | NumPy, SciPy, Pandas |
| **Machine Learning** | scikit-learn, PyTorch, XGBoost |
| **High Performance** | Zig for hot paths, Python bindings |
| **Data** | SQLite + Parquet for storage |
| **Visualization** | Matplotlib, Plotly |
| **Testing** | pytest, hypothesis |
| **Version Control** | DVC for data, Git for code |

---

## 1. Python Environment with UV

### UV Installation and Setup

```bash
# Install UV (recommended method)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create new project
uv init my-research-project
cd my-research-project

# Create virtual environment
uv venv

# Activate (platform-specific)
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows

# Install dependencies
uv add numpy pandas scipy scikit-learn

# Install with dev dependencies
uv add --dev pytest hypothesis ruff mypy

# Install from requirements.txt
uv pip install -r requirements.txt

# Export to requirements.txt
uv pip freeze > requirements.txt
```

### Project Structure

```
research/
├── src/
│   ├── data/            # Data loading and preprocessing
│   ├── models/          # Model implementations
│   ├── strategies/      # Trading strategies
│   ├── backtest/        # Backtesting engine
│   └── utils/           # Utilities
├── notebooks/           # Jupyter notebooks
├── data/                # Raw data (gitignored)
├── results/             # Experiment results
├── tests/               # Unit tests
├── pyproject.toml       # UV project config
└── README.md
```

### pyproject.toml Configuration

```toml
[project]
name = "my-research"
version = "0.1.0"
description = "DeFi market making research"
requires-python = ">=3.11"
dependencies = [
    "numpy>=2.0.0",
    "pandas>=2.0.0",
    "scipy>=1.13.0",
    "scikit-learn>=1.5.0",
    "xgboost>=2.0.0",
    "matplotlib>=3.9.0",
    "plotly>=5.20.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "hypothesis>=6.100.0",
    "ruff>=0.5.0",
    "mypy>=1.10.0",
    "jupyter>=1.0.0",
    "ipykernel>=6.29.0",
]

[tool.uv]
dev-dependencies = [
    "pytest",
    "hypothesis",
    "ruff",
    "mypy",
]

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
```

---

## 2. DeFi AMM Fundamentals

### AMM Types and Characteristics

| AMM Type | Formula | Use Case |
|----------|---------|----------|
| **Constant Product (x*y=k)** | `y = k / x` | Standard liquidity pairs (Uniswap V2) |
| **Constant Sum (x+y=k)** | `y = k - x` | Stable pairs (rare) |
| **StableSwap** | Hybrid | Stablecoins (Curve) |
| **Concentrated** | Custom range | Capital efficient (Uniswap V3) |
| **Hooks** | Custom | Advanced strategies (Uniswap V4) |

### Constant Product AMM Math

```python
import numpy as np

def constant_product_swap(
    reserve_x: float,
    reserve_y: float,
    amount_in: float,
    fee_bps: int = 30
) -> tuple[float, float]:
    """
    Calculate output amount for constant product AMM.
    Formula: amount_out = (amount_in * 997) / (reserve_x * 1000 + amount_in * 997)
    """
    fee_multiplier = 10000 - fee_bps
    amount_in_with_fee = amount_in * fee_multiplier

    numerator = amount_in_with_fee * reserve_y
    denominator = (reserve_x * 10000) + amount_in_with_fee

    amount_out = numerator // denominator

    # Update reserves
    new_reserve_x = reserve_x + amount_in
    new_reserve_y = reserve_y - amount_out

    return float(amount_out), new_reserve_x, new_reserve_y


def calculate_price(reserve_x: float, reserve_y: float) -> float:
    """Calculate spot price (y per x)."""
    return reserve_y / reserve_x


def calculate_slippage(
    reserve_x: float,
    reserve_y: float,
    amount_in: float,
    fee_bps: int = 30
) -> float:
    """Calculate price impact / slippage."""
    spot_price = calculate_price(reserve_x, reserve_y)
    amount_out, _, _ = constant_product_swap(reserve_x, reserve_y, amount_in, fee_bps)
    execution_price = amount_out / amount_in
    return (spot_price - execution_price) / spot_price
```

### Liquidity Provision Analysis

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class LiquidityPosition:
    """Track liquidity position and P&L."""
    reserve_x_0: float
    reserve_y_0: float
    liquidity: float
    entry_price: float

    @property
    def current_value(self, reserve_x: float, reserve_y: float, price: float) -> float:
        """Calculate current position value."""
        share = self.liquidity / (self.reserve_x_0 * self.reserve_y_0) ** 0.5
        return share * (reserve_x + price * reserve_y)

    @property
    def impermanent_loss(self, reserve_x: float, reserve_y: float, price: float) -> float:
        """Calculate impermanent loss."""
        # HODL value
    hodl_value = (self.reserve_x_0 * price + self.reserve_y_0)

    # LP value
    lp_value = self.current_value(reserve_x, reserve_y, price)

    return (hodl_value - lp_value) / hodl_value


def calculate_impermanent_loss(price_ratio: float) -> float:
    """
    Calculate impermanent loss for constant product AMM.
    price_ratio = current_price / entry_price
    """
    sqrt_ratio = np.sqrt(price_ratio)
    return (2 * sqrt_ratio) / (1 + price_ratio) - 1
```

---

## 3. Market Making Strategies

### Avellaneda-Stoikov Model

```python
import numpy as np
from dataclasses import dataclass

@dataclass
class ASModelParams:
    """Avellaneda-Stoikov market making parameters."""
    gamma: float      # Risk aversion parameter
    sigma: float      # Volatility
    k: float          # Order book depth parameter
    T: float          # Time horizon

def optimal_spread(params: ASModelParams, inventory: float, tau: float) -> tuple[float, float]:
    """
    Calculate optimal bid and ask spreads using Avellaneda-Stoikov.
    tau: Time to quote expiration
    """
    gamma = params.gamma
    sigma = params.sigma
    k = params.k

    # Half spread
    delta = (gamma * sigma**2 * tau + 2 * gamma * k * sigma) / (gamma - 1) * inventory

    # Optimal reservation price
    r = inventory - gamma * sigma**2 * (params.T - tau)

    # Optimal bid and ask
    bid = r - delta
    ask = r + delta

    return float(bid), float(ask)


def quote_generator(
    mid_price: float,
    inventory: float,
    params: ASModelParams,
    tau: float = 1.0
) -> tuple[float, float]:
    """Generate bid-ask quotes."""
    reservation_adjustment, half_spread = optimal_spread(params, inventory, tau)

    bid_price = mid_price + reservation_adjustment - half_spread
    ask_price = mid_price + reservation_adjustment + half_spread

    return bid_price, ask_price
```

### Inventory Management

```python
class InventoryManager:
    """Manage trading inventory and risk limits."""

    def __init__(
        self,
        max_position: float,
        target_inventory: float = 0.0,
        risk_tolerance: float = 1.0
    ):
        self.max_position = max_position
        self.target_inventory = target_inventory
        self.risk_tolerance = risk_tolerance
        self.current_inventory = 0.0

    def should_quote(self, side: str, price: float) -> bool:
        """Determine if we should quote on a side."""
        if side == 'buy':
            return self.current_inventory < self.max_position
        else:
            return self.current_inventory > -self.max_position

    def adjust_quote_size(self, base_size: float, side: str) -> float:
        """Adjust quote size based on inventory."""
        inventory_ratio = abs(self.current_inventory) / self.max_position

        # Reduce size as inventory approaches limits
        if side == 'buy' and self.current_inventory > 0:
            return base_size * (1 - inventory_ratio * 0.5)
        elif side == 'sell' and self.current_inventory < 0:
            return base_size * (1 - inventory_ratio * 0.5)

        return base_size

    def update_inventory(self, side: str, size: float, price: float):
        """Update inventory after fill."""
        if side == 'buy':
            self.current_inventory += size
        else:
            self.current_inventory -= size
```

### Backtesting Engine

```python
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime

@dataclass
class Trade:
    timestamp: datetime
    side: str
    price: float
    size: float
    fee: float

@dataclass
class BacktestResult:
    total_pnl: float
    total_fees: float
    win_rate: float
    sharpe_ratio: float
    max_drawdown: float
    trades: List[Trade]

class Backtester:
    """Simple backtesting engine for market making strategies."""

    def __init__(
        self,
        initial_capital: float = 100_000,
        fee_bps: int = 5,
        slippage_bps: int = 1
    ):
        self.initial_capital = initial_capital
        self.fee_rate = fee_bps / 10000
        self.slippage_rate = slippage_bps / 10000
        self.trades: List[Trade] = []

    def run(
        self,
        price_data: np.ndarray,
        timestamps: List[datetime],
        strategy
    ) -> BacktestResult:
        """Run backtest on price data."""
        capital = self.initial_capital
        position = 0.0
        total_fees = 0.0
        pnl_history = []

        for i, (price, ts) in enumerate(zip(price_data, timestamps)):
            # Get strategy signals
            bid, ask = strategy.get_quotes(price, position, i)

            # Simulate fills (simplified)
            if i > 0:
                prev_price = price_data[i - 1]
                if ask <= prev_price and position >= 0:
                    # Sell filled
                    size = min(1.0, position + capital / ask) if position >= 0 else 0
                    if size > 0:
                        fee = size * ask * self.fee_rate
                        capital += size * ask - fee
                        position -= size
                        total_fees += fee
                        self.trades.append(Trade(ts, 'sell', ask, size, fee))

                elif bid >= prev_price and position <= 0:
                    # Buy filled
                    size = min(1.0, capital / bid) if position <= 0 else 0
                    if size > 0:
                        fee = size * bid * self.fee_rate
                        capital -= size * bid + fee
                        position += size
                        total_fees += fee
                        self.trades.append(Trade(ts, 'buy', bid, size, fee))

            # Calculate PnL
            unrealized_pnl = position * price
            total_pnl = capital + unrealized_pnl - self.initial_capital
            pnl_history.append(total_pnl)

        # Calculate metrics
        returns = np.diff(pnl_history) / self.initial_capital
        sharpe = np.mean(returns) / np.std(returns) * np.sqrt(252) if len(returns) > 0 else 0
        max_dd = self._calculate_max_drawdown(pnl_history)
        win_rate = sum(1 for t in self.trades if t.fee < t.price * t.size) / len(self.trades) if self.trades else 0

        return BacktestResult(
            total_pnl=capital + position * price_data[-1] - self.initial_capital,
            total_fees=total_fees,
            win_rate=win_rate,
            sharpe_ratio=sharpe,
            max_drawdown=max_dd,
            trades=self.trades
        )

    def _calculate_max_drawdown(self, pnl_history: List[float]) -> float:
        """Calculate maximum drawdown."""
        peak = -float('inf')
        max_dd = 0.0

        for value in pnl_history:
            if value > peak:
                peak = value
            dd = (peak - value) / peak if peak > 0 else 0
            max_dd = max(max_dd, dd)

        return max_dd
```

---

## 4. Linear Algebra for Quant Finance

### Covariance Matrix and PCA

```python
import numpy as np
from numpy.linalg import eig, inv
from typing import Tuple

def calculate_returns(prices: np.ndarray, period: int = 1) -> np.ndarray:
    """Calculate log returns."""
    return np.log(prices[period:] / prices[:-period])

def calculate_covariance(returns: np.ndarray) -> np.ndarray:
    """Calculate covariance matrix."""
    return np.cov(returns.T)

def pca_analysis(returns: np.ndarray, n_components: int = 3) -> Tuple[np.ndarray, np.ndarray]:
    """
    Perform PCA on returns.
    Returns: (eigenvalues, eigenvectors)
    """
    cov_matrix = calculate_covariance(returns)
    eigenvalues, eigenvectors = eig(cov_matrix)

    # Sort by eigenvalue
    idx = np.argsort(eigenvalues)[::-1]
    eigenvalues = eigenvalues[idx]
    eigenvectors = eigenvectors[:, idx]

    return eigenvalues[:n_components], eigenvectors[:, :n_components]

def markowitz_portfolio(
    expected_returns: np.ndarray,
    cov_matrix: np.ndarray,
    risk_free_rate: float = 0.0
) -> np.ndarray:
    """
    Calculate optimal Markowitz portfolio weights.
    """
    n = len(expected_returns)
    ones = np.ones(n)

    # Inverse covariance matrix
    inv_cov = inv(cov_matrix)

    # Excess returns
    excess_returns = expected_returns - risk_free_rate

    # Optimal weights
    numerator = inv_cov @ excess_returns
    denominator = ones.T @ inv_cov @ ones
    weights = numerator / denominator

    return weights
```

### Optimal Stopping Theory

```python
def optimal_stopping_threshold(values: np.ndarray, cost: float) -> float:
    """
    Calculate optimal stopping threshold using backward induction.
    Useful for timing entry/exit decisions.
    """
    n = len(values)
    value_function = np.zeros(n + 1)

    for i in range(n - 1, -1, -1):
        # Value of stopping now
        stop_value = values[i] - cost

        # Value of continuing
        continue_value = value_function[i + 1]

        # Optimal decision
        value_function[i] = max(stop_value, continue_value)

    return value_function[0]
```

---

## 5. Machine Learning for Trading

### Feature Engineering

```python
import numpy as np
import pandas as pd
from typing import List

def create_technical_features(prices: pd.Series, window: int = 20) -> pd.DataFrame:
    """Create technical indicator features."""
    features = pd.DataFrame(index=prices.index)

    # Returns
    features['returns'] = prices.pct_change()
    features['log_returns'] = np.log(prices / prices.shift(1))

    # Moving averages
    features['sma_short'] = prices.rolling(window=window).mean()
    features['sma_long'] = prices.rolling(window=window * 2).mean()
    features['ema'] = prices.ewm(span=window).mean()

    # Price relative to MA
    features['price_to_sma'] = prices / features['sma_short']

    # Volatility
    features['volatility'] = features['returns'].rolling(window=window).std()
    features['atr'] = _calculate_atr(prices, window)

    # Momentum
    features['rsi'] = _calculate_rsi(prices, window)
    features['momentum'] = prices / prices.shift(window) - 1

    # Bollinger Bands
    bb_upper = features['sma_short'] + 2 * features['volatility']
    bb_lower = features['sma_short'] - 2 * features['volatility']
    features['bb_position'] = (prices - bb_lower) / (bb_upper - bb_lower)

    return features.dropna()

def _calculate_rsi(prices: pd.Series, window: int) -> pd.Series:
    """Calculate RSI indicator."""
    delta = prices.diff()
    gain = (delta.where(delta > 0, 0)).rolling(window=window).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=window).mean()
    rs = gain / loss
    return 100 - (100 / (1 + rs))

def _calculate_atr(prices: pd.Series, window: int) -> pd.Series:
    """Calculate Average True Range."""
    high = prices
    low = prices
    close = prices.shift(1)

    tr = pd.concat([
        high - low,
        (high - close).abs(),
        (low - close).abs()
    ], axis=1).max(axis=1)

    return tr.rolling(window=window).mean()
```

### Price Prediction Model

```python
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import mean_squared_error, mean_absolute_error
from sklearn.preprocessing import StandardScaler
import xgboost as xgb

class PricePredictor:
    """Machine learning price prediction model."""

    def __init__(self, model_type: str = 'xgboost'):
        self.model_type = model_type
        self.scaler = StandardScaler()
        self.model = None

    def train(
        self,
        features: pd.DataFrame,
        target: pd.Series,
        test_size: float = 0.2
    ) -> dict:
        """Train the prediction model."""
        # Split data
        X_train, X_test, y_train, y_test = train_test_split(
            features, target, test_size=test_size, shuffle=False
        )

        # Scale features
        X_train_scaled = self.scaler.fit_transform(X_train)
        X_test_scaled = self.scaler.transform(X_test)

        # Initialize model
        if self.model_type == 'xgboost':
            self.model = xgb.XGBRegressor(
                n_estimators=100,
                max_depth=6,
                learning_rate=0.05,
                subsample=0.8,
                colsample_bytree=0.8,
            )
        elif self.model_type == 'rf':
            self.model = RandomForestRegressor(
                n_estimators=100,
                max_depth=10,
                min_samples_split=10,
            )
        else:
            self.model = GradientBoostingRegressor(
                n_estimators=100,
                max_depth=6,
                learning_rate=0.05,
            )

        # Train
        self.model.fit(X_train_scaled, y_train)

        # Evaluate
        train_pred = self.model.predict(X_train_scaled)
        test_pred = self.model.predict(X_test_scaled)

        metrics = {
            'train_mse': mean_squared_error(y_train, train_pred),
            'test_mse': mean_squared_error(y_test, test_pred),
            'train_mae': mean_absolute_error(y_train, train_pred),
            'test_mae': mean_absolute_error(y_test, test_pred),
            'cv_score': cross_val_score(
                self.model, X_train_scaled, y_train, cv=5, scoring='neg_mean_squared_error'
            ).mean()
        }

        return metrics

    def predict(self, features: pd.DataFrame) -> np.ndarray:
        """Make predictions."""
        X_scaled = self.scaler.transform(features)
        return self.model.predict(X_scaled)

    def feature_importance(self, feature_names: List[str]) -> pd.Series:
        """Get feature importance."""
        if hasattr(self.model, 'feature_importances_'):
            return pd.Series(
                self.model.feature_importances_,
                index=feature_names
            ).sort_values(ascending=False)
        return pd.Series()
```

---

## 6. Zig for High-Performance Simulation

### When to Use Zig

| Use Case | Reason |
|----------|--------|
| Monte Carlo simulation | 10-100x faster than Python |
| Real-time pricing | Microsecond latency required |
| Large-scale backtesting | Process millions of ticks |
| Hot path calculations | Optimize critical loops |

### Zig Price Simulation

```zig
// simulation.zig
const std = @import("std");

// Price path using geometric Brownian motion
pub fn generateGBMPath(
    allocator: std.mem.Allocator,
    S0: f64,
    mu: f64,
    sigma: f64,
    T: f64,
    steps: usize,
    rng: *std.rand.Random,
) ![]f64 {
    const dt = T / @as(f64, @floatFromInt(steps));
    var path = try allocator.alloc(f64, steps + 1);
    path[0] = S0;

    var i: usize = 1;
    while (i <= steps) : (i += 1) {
        const Z = rng.floatNorm(f64);
        path[i] = path[i - 1] * std.math.exp((mu - 0.5 * sigma * sigma) * dt + sigma * std.math.sqrt(dt) * Z);
    }

    return path;
}

// Monte Carlo option pricing
pub fn monteCarloOptionPrice(
    S0: f64,
    K: f64,
    T: f64,
    r: f64,
    sigma: f64,
    simulations: usize,
    steps: usize,
    rng: *std.rand.Random,
) f64 {
    var sum: f64 = 0;
    var i: usize = 0;

    while (i < simulations) : (i += 1) {
        const path = generateGBMPath(
            std.heap.page_allocator,
            S0,
            r,
            sigma,
            T,
            steps,
            rng,
        ) catch continue;

        const payoff = @max(0, path[path.len - 1] - K);
        sum += payoff;

        std.heap.page_allocator.free(path);
    }

    const payoff_mean = sum / @as(f64, @floatFromInt(simulations));
    return std.math.exp(-r * T) * payoff_mean;
}

// Batch pricing for multiple options
pub fn batchPriceOptions(
    options: []const OptionParams,
    simulations: usize,
) []f64 {
    // Implementation for parallel pricing
    _ = options;
    _ = simulations;
    unreachable;
}

pub const OptionParams = struct {
    S0: f64,
    K: f64,
    T: f64,
    r: f64,
    sigma: f64,
};
```

### Compile and Use from Python

```bash
# Build as library
zig build-lib simulation.zig -dynamic -OReleaseFast

# Or build as Python extension (requires ziglang-py)
```

---

## 7. Data Management

### Data Storage Patterns

```python
import sqlite3
import pandas as pd
from pathlib import Path
from typing import Optional

class DataManager:
    """Manage research data storage."""

    def __init__(self, db_path: str = "data/research.db"):
        self.db_path = Path(db_path)
        self.db_path.parent.mkdir(parents=True, exist_ok=True)
        self.conn = sqlite3.connect(self.db_path)
        self._init_db()

    def _init_db(self):
        """Initialize database schema."""
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS price_data (
                symbol TEXT NOT NULL,
                timestamp INTEGER NOT NULL,
                open REAL,
                high REAL,
                low REAL,
                close REAL,
                volume REAL,
                PRIMARY KEY (symbol, timestamp)
            )
        """)

        self.conn.execute("""
            CREATE INDEX IF NOT EXISTS idx_symbol_time
            ON price_data (symbol, timestamp DESC)
        """)

    def store_prices(self, df: pd.DataFrame, symbol: str):
        """Store price data."""
        df['symbol'] = symbol
        df.to_sql('price_data', self.conn, if_exists='append', index=False)

    def load_prices(
        self,
        symbol: str,
        start: Optional[int] = None,
        end: Optional[int] = None
    ) -> pd.DataFrame:
        """Load price data."""
        query = "SELECT * FROM price_data WHERE symbol = ?"
        params = [symbol]

        if start is not None:
            query += " AND timestamp >= ?"
            params.append(start)
        if end is not None:
            query += " AND timestamp <= ?"
            params.append(end)

        query += " ORDER BY timestamp ASC"

        return pd.read_sql_query(query, self.conn, params=params)

    def store_parquet(self, df: pd.DataFrame, path: str):
        """Store data as Parquet (for large datasets)."""
        Path(path).parent.mkdir(parents=True, exist_ok=True)
        df.to_parquet(path, compression='snappy')

    def load_parquet(self, path: str) -> pd.DataFrame:
        """Load data from Parquet."""
        return pd.read_parquet(path)
```

---

## 8. Testing and Validation

### Hypothesis Testing

```python
from hypothesis import given, strategies as st
import numpy as np

@given(
    prices=st.lists(st.floats(min_value=1, max_value=10000), min_size=10, max_size=1000),
    amounts=st.floats(min_value=0.001, max_value=1000),
    fees=st.integers(min_value=0, max_value=100)
)
def test_constant_product_invariant(prices, amounts, fees):
    """Test that constant product AMM maintains invariant."""
    price = prices[0]
    amount_in = amounts

    # Simulate swap
    reserve_x, reserve_y = 1000.0, 1000.0
    k_initial = reserve_x * reserve_y

    amount_out, new_x, new_y = constant_product_swap(
        reserve_x, reserve_y, amount_in, fees
    )

    k_final = new_x * new_y

    # K should increase (fees) or stay same
    assert k_final >= k_initial * 0.999999  # Allow small rounding
```

### Property-Based Testing

```python
import pytest

@pytest.mark.parametrize("n_assets", [2, 3, 5, 10])
def test_markowitz_portfolio_weights_sum_to_one(n_assets):
    """Test that Markowitz weights sum to one."""
    np.random.seed(42)
    returns = np.random.randn(n_assets) * 0.01 + 0.05
    cov_matrix = np.random.randn(n_assets, n_assets)
    cov_matrix = cov_matrix @ cov_matrix.T  # Make positive semi-definite

    weights = markowitz_portfolio(returns, cov_matrix)

    assert np.abs(np.sum(weights) - 1.0) < 1e-10
    assert all(weights >= -1e-10)  # Allow small negative due to numerical error
```

---

## 9. Research Workflow

### Experiment Tracking

```python
from dataclasses import dataclass, asdict
from datetime import datetime
import json
from pathlib import Path

@dataclass
class ExperimentConfig:
    model_type: str
    features: list[str]
    train_period: str
    test_period: str
    hyperparameters: dict

@dataclass
class ExperimentResult:
    config: ExperimentConfig
    metrics: dict
    timestamp: str
    run_id: str

def save_experiment(result: ExperimentResult, path: str = "results/"):
    """Save experiment results to JSON."""
    Path(path).mkdir(parents=True, exist_ok=True)

    file_path = Path(path) / f"{result.run_id}.json"
    with open(file_path, 'w') as f:
        json.dump(asdict(result), f, indent=2, default=str)

def load_experiment(run_id: str, path: str = "results/") -> ExperimentResult:
    """Load experiment results."""
    file_path = Path(path) / f"{run_id}.json"
    with open(file_path) as f:
        data = json.load(f)

    return ExperimentResult(**data)
```

### Jupyter Notebook Best Practices

```python
# At the start of every notebook
%load_ext autoreload
%autoreload 2

# Import project modules
import sys
sys.path.append('../src')

from models.amm import ConstantProductAMM
from backtest.engine import Backtester

# Set random seeds for reproducibility
import numpy as np
np.random.seed(42)

# Document parameters
CONFIG = {
    'initial_capital': 100_000,
    'fee_bps': 5,
    'symbols': ['ETH-USDT', 'BTC-USDT'],
}
```

---

## 10. Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Look-ahead bias | Ensure no future data in features |
| Overfitting | Use cross-validation, holdout sets |
| Ignoring fees | Include transaction costs in backtest |
| Unrealistic fills | Model slippage and market impact |
| Data snooping | Test on out-of-sample data |
| Stationarity issues | Use returns, not prices |
| Surviviorship bias | Include delisted assets |
| Regime changes | Test across different market periods |

---

## 11. Performance Optimization

```python
# Vectorization instead of loops
# ❌ SLOW
for i in range(len(returns)):
    if returns[i] > 0:
        returns[i] *= (1 - fee)

# ✅ FAST
returns = np.where(returns > 0, returns * (1 - fee), returns)

# Use numba for JIT compilation
from numba import jit

@jit(nopython=True)
def fast_pnl_simulation(prices: np.ndarray, quantities: np.ndarray) -> float:
    """Fast PnL calculation with numba."""
    pnl = 0.0
    for i in range(len(prices) - 1):
        pnl += quantities[i] * (prices[i + 1] - prices[i])
    return pnl
```

---

## 12. Development Workflow

```bash
# Setup project
uv init quant-research
cd quant-research
uv venv
source .venv/bin/activate  # or activate on Windows

# Install dependencies
uv add numpy pandas scipy scikit-learn xgboost matplotlib plotly
uv add --dev pytest hypothesis ruff mypy jupyter

# Code quality
ruff check src/
ruff check --fix src/
mypy src/

# Testing
pytest tests/
pytest tests/ -hypothesis-seed=0  # Reproducible property tests

# Run notebook
jupyter notebook

# Or use Jupyter Lab
uv add --dev jupyterlab
jupyter-lab
```

---

## Internal References

- [`CONTRIBUTING.md`](../CONTRIBUTING.md) - Commit conventions
- [`BACKEND.md`](./BACKEND.md) - Back-end patterns

---

*Last updated: 2025-01*
