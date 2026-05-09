# BTR Prime -- ML Methodology

ML-driven trading strategy optimization and execution for crypto spot and concentrated liquidity.

**Scope boundary:** BTR receives memory-mapped information bars ([mitch::Bar](../../nx/nx-rates/mitch/model/bar.md), 96 bytes = 64B OHLCV + 32B microstructure) produced by [NX Rates](../../nx/nx-rates/docs/architecture.md). BTR computes features, trains models, evaluates fitness, and executes trades using a unified pipeline: GBM predicts Parkinson-normalized returns, deterministic sigma-based trading rules translate predictions into orders. No chained optimizers.

**Upstream references** (canonical specs live in NX Rates):
- Bar wire format: [nx-rates/mitch/model/bar.md](../../nx/nx-rates/mitch/model/bar.md)
- Composite Index (TDWAP) methodology: [nx-rates/docs/aggregation-methodology.md](../../nx/nx-rates/docs/aggregation-methodology.md)
- Service architecture: [nx-rates/docs/architecture.md](../../nx/nx-rates/docs/architecture.md)

---

## Strategy Types

| Code | Name | Description |
|------|------|-------------|
| LO | Long-Only | Takes only long positions on perpetual futures |
| SO | Short-Only | Takes only short positions on perpetual futures |
| CL | Concentrated Liquidity | Manages LP positions in Uniswap V3-style AMMs |

Each strategy runs independently per ticker (e.g., `btc-usdt-lo`, `eth-usdt-cl`).

---

## Feature Engineering

All features are information-agnostic: computed in bar-counts, not clock time. A "bar" is a unit of market information regardless of whether it is a Renko brick, dollar bar, or time candle.

### Lookback Derivation

Feature lookback windows are derived from the prediction horizon via a 1:3:10 ratio (Lopez de Prado 2018, Kakushadze 2016):

| Scale | Multiplier | Default (H=20) | Rationale |
|-------|------------|-----------------|-----------|
| Short (S) | 1x H | 20 | Immediate momentum matching prediction window |
| Medium (M) | 3x H | 60 | Intraday trend context |
| Long (L) | 10x H | 200 | Regime detection (Hurst needs >= 100 samples) |

### Core Features (29)

Organized as multi-timeframe (MTF) triplets at S/M/L lookbacks, plus cross-scale and derived features:

| # | Group | Features | Count |
|---|-------|----------|-------|
| 1-3 | MTF Momentum: Returns | `ret_s`, `ret_m`, `ret_l` | 3 |
| 4-6 | MTF Momentum: RSI | `rsi_s`, `rsi_m`, `rsi_l` | 3 |
| 7-9 | MTF Volatility: Parkinson | `pvol_s`, `pvol_m`, `pvol_l` | 3 |
| 10-12 | MTF Trend: Efficiency Ratio | `er_s`, `er_m`, `er_l` | 3 |
| 13-15 | MTF Trend: ADX | `adx_s`, `adx_m`, `adx_l` | 3 |
| 16-18 | MTF Trend: Autocorrelation | `autocorr_s`, `autocorr_m`, `autocorr_l` | 3 |
| 19-21 | MTF Mean Reversion: Z-score | `zscore_s`, `zscore_m`, `zscore_l` | 3 |
| 22-23 | Cross-scale Regime | `vol_regime` (pvol_s/pvol_l), `er_regime` (er_s/er_l) | 2 |
| 24-28 | Derived Single-scale | `hurst`, `ema_cross`, `price_accel`, `obv_slope`, `clv` | 5 |
| 29 | Vol-of-Vol Regime | `vol_of_vol` (std(pvol)/mean(pvol) over M window) | 1 |

Optional: +2 time features (`time_sin`, `time_cos`) for time-based bars only.

Z-scores are clipped to [-4, 4] to prevent GBM gain bias from unbounded features (Zheng et al. 2017).

Minimum bars before features are valid: `1.5 * L` (longest lookback warmup).

### Renko-Specific Features (18)

When using Renko/Dollar Imbalance Bars, the mitch::Bar cache-line-2 microstructure fields (realized_var, drift, avg_spread_bps, signed vol_imbalance/OFI, up_tick_ratio, autocorr1, avg_ci_ubp, reject_rate) are projected through z-score and regime-ratio transforms and appended to the core feature set.

### Feature Selection

The GBM acts as the feature selector -- we provide each indicator at multiple scales and let importance-based selection discover which timeframes carry alpha. This avoids lookback parameter optimization which would overfit to historical regime durations (Coqueret & Guida 2020).

---

## Model: PerpetualBooster

Hyperparameter-free gradient boosted trees (Perpetual). The `budget` parameter controls bias-variance tradeoff via self-generalizing regularization -- the model stops adding trees when the expected generalization loss exceeds the budget. No manual tuning of learning rate, depth, or iterations required (Hasan 2024).

| Parameter | Value | Notes |
|-----------|-------|-------|
| budget | 0.3 | Lower = more conservative; 0.0-inf range |
| max_bin | 64 | Sufficient for <= 10K training rows |
| stopping_rounds | 10 | Early convergence detection |
| iteration_limit | 200 | Hard cap (rarely reached) |
| num_threads | 1 | Avoids rayon pool thrash in parallel processes |

### Classification vs Regression

| Mode | Labels | Output | Use Case |
|------|--------|--------|----------|
| Classification | Binary (return > 0.001 -> 1, else 0) | Probability [0, 1] | Direction filtering |
| Regression | Parkinson-normalized return y = r / (sigma_P * sqrt(H)) | Continuous | Unified pipeline (default) |

The label threshold (0.001) is fixed and never coupled to conviction thresholds. Changing it would alter the class balance and invalidate model calibration.

---

## Parkinson Volatility

Parkinson (1980) volatility is used throughout the pipeline as the canonical volatility measure, replacing ATR.

### Theory

The Parkinson estimator uses the high-low range of each bar:

```
sigma_P = sqrt( mean(ln(H/L)^2) / (4 * ln(2)) )
```

This is 5.2x more statistically efficient than the standard close-to-close estimator (Parkinson 1980, Garman & Klass 1980). The constant `1/(4*ln2) ~ 0.3607` is the scaling factor that makes the estimator unbiased under geometric Brownian motion.

### Why Parkinson Over ATR

| Property | ATR | Parkinson sigma |
|----------|-----|-----------------|
| Statistical efficiency | Low (uses O, H, L, C equally) | 5.2x higher (optimal use of range) |
| Theoretical grounding | Empirical (Wilder 1978) | Derived from GBM maximum likelihood |
| Smoothing | Exponential moving average | Rolling window (no decay bias) |
| Cross-instrument comparability | No (absolute units) | Yes (log-return units) |
| Composability | Not sqrt(t)-scalable | Scales as sigma * sqrt(t) |

The sqrt(t) scaling property is critical: it lets us express all thresholds in sigma * sqrt(H) units, making them automatically adapt to both the current volatility regime and the prediction horizon.

### Implementation

Rolling circular-buffer computation, O(1) per bar:

```
sigma_P[i] = sqrt( sum(ln(H[j]/L[j])^2 for j in [i-lookback..i]) * INV_4_LN2 / n_valid )
```

Default lookback: 50 bars (`config.yml: ml.parkinson_lookback`).

### Parkinson-Normalized Labeling

Forward returns are normalized by local Parkinson volatility to make predictions scale-invariant across volatility regimes (Lopez de Prado 2018):

```
y[i] = (close[i+H] / close[i] - 1) / (sigma_P[i] * sqrt(H))
```

This means a prediction of +1.0 represents a one-sigma move, regardless of whether absolute volatility is 0.5% or 5%. Labels are clamped to [-5, 5] to bound outliers.

---

## Walk-Forward Protocol

```
|<--- train_bars (5000) --->|<- embargo ->|<- retrain_bars (500) ->|
|         TRAIN             |   GAP (H+E) |        EVALUATE        |
```

| Parameter | Default | Source | Description |
|-----------|---------|--------|-------------|
| train_bars | 5,000 | config.yml | Training window (information events) |
| retrain_bars | 500 | config.yml | Bars between retrains |
| horizon_bars | 20 | config.yml | Prediction horizon |
| eval_interval | 20 | config.yml | Signal generation frequency |
| min_bars_per_fold | 100 | code | Minimum data per fold |
| ensemble_count | 4 | code | Models retained in ensemble |
| ensemble_decay | 0.5 | code | Exponential decay factor for older models |

The embargo gap (`horizon_bars + eval_interval`) between training and evaluation prevents label leakage -- a point emphasized by Lopez de Prado (2018) as critical for avoiding overfitting in walk-forward validation.

### Genuine Out-of-Sample Evaluation

Each fold trains on historical data and predicts strictly forward. No fold ever sees data from its own evaluation window or any future fold. This is the anchored walk-forward method recommended by Pardo (2008) and formalized by Lopez de Prado (2018, Ch. 7).

### Train Stride

GBM has diminishing returns past ~7,000 samples for histogram-based splitting (Chen & Guestrin 2016). When train_bars > 7,000, rows are subsampled with stride = train_bars / 7,000.

### Ensemble

- Retain up to `ensemble_count` (4) most recent fold models
- Exponential decay weighting: newest model weight = 1.0, each older fold *= 0.5
- Final prediction: weighted average across all retained folds
- Variance reduction follows Breiman (2001): ensemble of M models reduces variance by ~1/M when models are sufficiently diverse

### Signal Generation (Regression Mode)

```
prediction = weighted_ensemble_average(fold_predictions)

if |prediction| > entry_threshold_sigma:
    signal = +1 (enter)
    confidence = |prediction|
else:
    signal = 0 (flat)
```

The entry threshold operates directly in sigma-normalized space, making it comparable to the prediction magnitude without unit conversion.

---

## Trading Rules (Unified Pipeline)

All trading rules are deterministic functions of the GBM prediction and Parkinson volatility. No second optimizer (CMA-ES was eliminated for directional trading). Every threshold is expressed in Parkinson sigma units, grounding the rules in the statistical properties of the underlying price process.

### Entry Threshold

**Rule:** Enter when `|prediction| > entry_threshold_sigma` (default: 0.20).

**Theoretical basis:** Grinold & Kahn (1999) derive the optimal signal threshold as a function of the information coefficient (IC) and transaction costs. For low-IC signals (typical of financial ML with R-squared ~ 1-3%), the threshold should be low enough to capture weak signals but high enough to exceed transaction cost drag. At 0.20 sigma with typical crypto vol, the expected move exceeds round-trip costs by 3-5x.

**Why sigma-normalized:** The prediction is already in sigma units (Parkinson-normalized labeling), so the entry threshold is directly comparable without unit conversion. A threshold of 0.20 means "enter when the predicted move exceeds 0.20 standard deviations over the horizon."

### Stop Loss (Risk-Budget)

**Rule:**
```
SL_distance = max_loss_per_trade_pct / (leverage * position_size_pct)
SL_clamped  = clamp(SL_distance / sigma_H, sl_min_sigma, sl_max_sigma) * sigma_H
```

Where `sigma_H = sigma_P * sqrt(H)` is the horizon-scaled Parkinson volatility.

| Parameter | Default | Description |
|-----------|---------|-------------|
| max_loss_per_trade_pct | 0.02 | 2% maximum capital loss per trade |
| sl_min_sigma | 0.5 | Minimum SL distance in sigma units (below -> reject trade) |
| sl_max_sigma | 4.0 | Maximum SL distance in sigma units |

**Theoretical basis:** The risk-budget approach derives SL from the maximum acceptable portfolio loss, not from price patterns. This is grounded in the Kelly criterion (Kelly 1956) and modern risk management (Vince 1992):

- **Kelly framework:** Optimal bet size depends on edge and variance. The SL is the inverse problem: given a fixed bet size and leverage, what price distance keeps total loss within budget?
- **Sigma bounds:** The [0.5, 4.0] sigma clamp ensures SL is neither too tight (whipsawed by normal volatility) nor too wide (ineffective risk control). A 0.5-sigma SL has ~31% probability of being hit by random walk; 4.0-sigma has ~0.006%.
- **Trade rejection:** If the risk-budget SL falls below 0.5 sigma, the trade is rejected rather than entered with an unrealistically tight stop. This typically happens at high leverage (>5x) with low volatility.

**Why not distribution-based (MAE):** Sweeney (1993) Maximum Adverse Excursion requires a history of completed trades to estimate the loss distribution. At walk-forward inception (cold start), no such history exists. The risk-budget approach works from bar one. Furthermore, conditioning SL on the model's own predictions is unreliable when model R-squared is low (1-3%) -- the SL would mostly track noise.

### Take Profit (Prediction + Sigma Adaptive)

**Rule:**
```
TP_distance = |prediction_in_price_space| + tp_sigma_mult * sigma_H
```

Where `prediction_in_price_space = prediction_sigma * sigma_H` for regression mode.

| Parameter | Default | Description |
|-----------|---------|-------------|
| use_take_profit | true | Enable/disable TP |
| tp_sigma_mult | 2.0 | Sigma multiples above prediction for TP |

**Theoretical basis:** This design combines two ideas:

1. **Sweeney (1993) Maximum Favorable Excursion (MFE):** The TP should capture moves beyond the model's prediction. Setting TP at `prediction + 2 * sigma` targets the region where price exceeds the prediction by 2 standard deviations -- roughly the 97.7th percentile of a normal distribution.

2. **Prediction-aware anchoring:** Unlike a fixed TP, this adapts to the model's conviction. A strong prediction (2.0 sigma) gets TP at 2.0 + 2.0 = 4.0 sigma, while a weak prediction (0.3 sigma) gets TP at 0.3 + 2.0 = 2.3 sigma. This naturally captures the asymmetry between high-conviction and low-conviction trades.

**Why TP is asymmetric to SL:** The SL uses a risk-budget derivation (protect capital) while the TP uses a prediction-adaptive derivation (capture upside). This asymmetry is intentional -- SL and TP serve fundamentally different objectives:

- SL limits downside: a portfolio management constraint (Vince 1992)
- TP captures upside: a return optimization target (Sweeney 1993)

Making both distribution-based would couple them to the same R-squared limitation. The risk-budget SL is robust to model quality; the adaptive TP is a bet on model quality.

### Exit Priority

Exits are evaluated in strict priority order:

1. **Stop loss** -- risk-budget SL hit (non-negotiable capital preservation)
2. **Take profit** -- adaptive TP hit (capture outlier favorable moves)
3. **Signal reversal** -- model prediction flips or goes flat for 2+ consecutive signals
4. **Horizon exit** -- position held for H bars without SL/TP/reversal (time decay)

The horizon exit ensures no position outlives its prediction window. This is analogous to the "triple barrier" labeling method (Lopez de Prado 2018, Ch. 3), adapted from a labeling scheme to an execution rule.

### Position Sizing

```
size = base_capital * position_size_pct * confidence_scale * drawdown_scale
```

| Factor | Formula | Range | Source |
|--------|---------|-------|--------|
| base_capital | initial_capital (non-compounding) | Fixed | config.yml |
| position_size_pct | 0.15 (15% of capital) | Fixed | config.yml |
| confidence_scale | clamp(abs(prediction) / entry_threshold, 0.2, 1.0) | [0.2, 1.0] | Grinold & Kahn (1999) |
| drawdown_scale | max(0.3, 1 - 0.7 * (dd_pct / 0.12)) | [0.3, 1.0] | Risk management |

**Confidence scaling:** Based on the Grinold & Kahn (1999) fundamental law of active management: bet size should scale with signal strength. A prediction of 2x the entry threshold gets full size; at the threshold boundary, size is reduced to 20%.

**Drawdown scaling:** Position size is linearly reduced as drawdown increases, reaching 30% of base size at 12% drawdown. This implements a basic anti-martingale rule: reduce exposure during losing streaks.

**Non-compounding:** Position sizes are computed from initial capital, not current equity. This prevents runaway sizing after gains and provides more stable risk exposure. Compounding can be enabled via `config.yml: ml.compound`.

### Leverage

| Parameter | Default | Description |
|-----------|---------|-------------|
| leverage | 3.0 | Position leverage multiplier |
| leverage_min | 0.1 | Lower bound (10% of capital exposure) |
| leverage_max | 10.0 | Upper bound |

The fitness function gatekeeps excessive leverage via drawdown penalties. A 10x leveraged strategy that breaches the 25% max drawdown hard filter will score zero fitness regardless of returns.

---

## Fitness Function

Industry-standard fitness for strategy evaluation. Weighted geometric mean with rolling monthly risk metrics. All components bounded [0, 1], continuous scoring with no dead zones.

Based on expert synthesis from quantitative finance literature (Pardo 2008, Kaufman 2013, Aronson 2007).

### Hard Filters (Fast Reject)

| Filter | Default | Description |
|--------|---------|-------------|
| min_trades | 20 | Absolute floor (dynamic 2/week filter does real work) |
| min_profit_factor | 1.0 | Gross wins / gross losses |
| min_sortino | 0.0 | Allow breakeven strategies |
| hard_dd_stop_pct | 25% | Max drawdown -- 40% requires 67% to recover (Vince 1992) |
| min_time_coverage | 20% | Fraction of days with trading activity |
| min_trades_per_week | 2.0 | Trade frequency floor |
| min_hold_hours | 0.083 | 5-minute minimum (reject micro-trades) |
| max_hold_hours | 720 | 30-day maximum (reject stuck positions) |

### Score Components

```
Fitness = Product(component_i ^ weight_i) * confidence_discount
```

| Component | Weight | Formula | Interpretation |
|-----------|--------|---------|----------------|
| Trajectory t-stat | 0.30 | `tanh(t_stat / 3.0)` | Linear log-equity growth significance |
| Drawdown magnitude | 0.10 | `exp(-dd / 8.0%)` | Penalizes avg monthly max drawdown |
| DD duration | 0.10 | `exp(-uw_days / 16.0)` | Penalizes avg monthly underwater time |
| Tail risk (CVaR) | 0.10 | `1 / (1 + (es_ratio / 1.5)^2)` | Monthly CVaR(95%) / vol ratio |
| Profit factor | 0.10 | `tanh((PF - 1.0) / 0.4)` (capped at 3.0) | Distribution sanity; diminishing returns past PF~2.5 |
| Coverage | 0.15 | `tanh(coverage)` | Temporal consistency of trading |
| Win rate | 0.15 | `tanh((wr - 0.20) / 0.15)` | Penalizes lottery-ticket strategies |

**Confidence discount:** `0.7 + 0.3 * sqrt(n_trades / 100)`, capped at 1.0.

### Why Trajectory T-Stat Dominates

The trajectory t-stat measures the statistical significance of log-equity growth via OLS regression. This is the single best predictor of genuine alpha vs overfitting:

- **Slope captures growth:** linear regression slope of log-equity ~ daily returns
- **SE captures noise:** standard error penalizes jagged, inconsistent growth
- **t-stat = slope / SE:** high t-stat requires both good returns AND low noise

Harvey et al. (2016) recommend t-stat > 2.0 as the minimum threshold for declaring significance in backtested strategies. The tanh scaling maps this to [0, 1] with the steepest gradient around t-stat ~ 2-3.

### Rolling Monthly Metrics

All risk metrics use rolling monthly computation rather than full-period. This prevents a single exceptional month from masking systematic problems:

- **Monthly max DD**: Equity curve split into ~30-day buckets, max DD per month, arithmetic mean
- **Monthly underwater days**: Time underwater per month, arithmetic mean
- **Monthly profit factor**: PF per month, geometric mean (all-win months capped at 3.0)
- **Monthly CVaR**: CVaR(95%) per month, arithmetic mean (fallback to full-period if <5 trades/month)
- **Monthly Sharpe/Sortino**: Per calendar month, shifted geometric mean: `(exp(mean(ln(1 + S/10))) - 1) * 10`

Design rationale (Lopez de Prado 2018): geometric mean for ratios (PF, Sharpe, Sortino) naturally penalizes bad months more than arithmetic mean. Arithmetic mean for magnitudes (DD, underwater days, CVaR) gives stable aggregation.

### CL Fitness Overrides

CL strategies use relaxed fitness parameters due to infrequent rebalancing:

| Parameter | CL Value | Default |
|-----------|----------|---------|
| min_trades | 5 | 20 |
| hard_dd_stop_pct | 40% | 25% |
| min_time_coverage | 15% | 20% |
| dd_duration_halflife_days | 10 | 16 |

---

## Concentrated Liquidity (CL) Strategy

Manages LP positions in Uniswap V3-style concentrated liquidity pools. ML model predicts when to rebalance range positions. CMA-ES optimizer is retained for CL range management (5D search space) because LP range width is not a direction prediction -- it requires explicit parameter optimization.

### Fee & LVR Model

Based on Milionis et al. (2022) Loss-Versus-Rebalancing framework:

```
LVR_rate = sigma^2 * sqrt(P) / (8 * LP_value)
```

Where sigma is bar-level Parkinson volatility with dampening factor 0.75.

#### Volatility-Fee Multiplier

Determines the fee premium/discount based on realized vol relative to reference:

```
ref_sigma = REF_SIGMA * sqrt(35064 / bars_per_year)
vol_fee_mult = sigma / ref_sigma
```

`REF_SIGMA = 0.0013` is calibrated for M15 bars (96 bars/day, 35064 bars/year). The scaling factor adjusts for different bar cadences.

### CL Labels

CL labels are net alpha vs HODL: fee income minus LVR losses minus gas costs, compared to simply holding the underlying. Positive label = LP outperforms HODL over the horizon.

---

## Backtesting

### Equity Curve Construction

The equity curve includes adverse mark-to-market points within each bar's SL/TP scan. When scanning candle wicks for SL/TP hits, the adverse price (low for longs, high for shorts) is recorded as an intra-bar equity point before the exit point. This prevents equity curves from hiding drawdowns within bars.

### Fee Model

```
fee_rate = fee_rate_bps / 10000   (default: 3 bps = 0.03% per leg)
entry_fee = size * fee_rate
exit_fee  = size * max(0, 1 + pnl_pct / leverage) * fee_rate
net_pnl   = size * pnl_pct - entry_fee - exit_fee
```

Exit fee scales with PnL because the notional position value changes with profit/loss.

---

## Live Execution (btr-runtime)

After optimization, `btr-runtime` drives live trading using the same signal pipeline applied to a real-time Renko stream.

### RenkoStream

Converts live 1m candles (Binance WebSocket) into Renko bricks using the last historical brick size as the initial step. Rolling Parkinson volatility (winsorized mean of ln(H/L)^2, 480-candle window) recomputes brick size every 60 candles with 7.5% hysteresis threshold. This matches the offline Parkinson estimator used in series-factory.

### OptimizerLoop

Accumulates new bricks into `live.bars` (appended to historical `.bars`). Every `retrain_every` new bricks (default: 500), spawns `btr-ml` as a subprocess on the full dataset. Parses the JSON output to extract the latest open trade direction as the active signal.

### Position State Machine

```
FLAT --(signal != 0, |pred| > entry_threshold_sigma)--> LONG or SHORT
  ^                                                           |
  +---(SL | TP | signal reversal | horizon exit)-------------+
```

SL/TP evaluated on each Renko brick close. Positions sized as `balance * position_size_pct * confidence_scale * dd_scale`.

### Providers

| Provider | Entry | Exit | Notes |
|----------|-------|------|-------|
| Paper | Fill at last price | Fill at last price | No slippage model; configurable fee |
| Binance Spot | Market BUY | Market SELL | HMAC-SHA256; spot only, no shorts |
| Li.Fi | Jumper quote -> on-chain swap | Reverse swap | Dry-run until wallet key injected |

Short selling (SO mode) on spot requires AAVE leverage -- not yet integrated in btr-runtime.

---

## Configuration

All tunable parameters live in `config.yml` under the `ml:` section. Nothing is hardcoded. The full parameter set:

```yaml
ml:
  # Model
  label_threshold: 0.001          # Classification label (fixed)
  budget: 0.3                     # GBM complexity budget
  train_bars: 5000                # Training window
  step_bars: 500                  # Retrain interval
  horizon_bars: 20                # Prediction horizon
  eval_interval: 20               # Signal frequency

  # Entry (Parkinson sigma-based)
  entry_threshold_sigma: 0.20     # |pred| must exceed k * sigma_P * sqrt(H)

  # Stop loss (risk-budget)
  max_loss_per_trade_pct: 0.02    # 2% max capital loss per trade
  sl_min_sigma: 0.5               # SL floor in sigma units
  sl_max_sigma: 4.0               # SL cap in sigma units

  # Take profit (prediction + sigma adaptive)
  use_take_profit: true           # TP = |pred| + tp_sigma * sigma_P * sqrt(H)
  tp_sigma_mult: 2.0              # Sigma multiples above prediction

  # Position sizing / leverage
  position_size_pct: 0.15         # 15% of capital per trade
  leverage: 3.0                   # Default leverage
  leverage_min: 0.1               # Grid search lower bound
  leverage_max: 10.0              # Grid search upper bound

  # Parkinson volatility
  parkinson_lookback: 50          # Rolling sigma_P window (bars)
```

---

## Source Code References

| Component | Key Files |
|-----------|-----------|
| Feature engineering | `prime/crates/ml/src/features.rs` |
| Walk-forward training | `prime/crates/ml/src/trainer.rs` |
| Parkinson volatility | `prime/crates/ml/src/trainer.rs:compute_rolling_parkinson()` |
| Trading rules (backtest) | `prime/crates/ml/src/trainer.rs:backtest_signals()` |
| CL pipeline | `prime/crates/ml/src/cl.rs` |
| Fitness function | `prime/crates/backtest/src/fitness.rs:compute_fitness()` |
| Statistics (Sharpe, Sortino) | `prime/crates/backtest/src/stats.rs` |
| CLI / orchestration | `prime/crates/ml/src/main.rs` |
| Configuration types | `prime/crates/config/src/params.rs:MlConfig` |
| GBM model wrapper | `prime/crates/ml/src/model.rs` |

---

## Theoretical References

| Author(s) | Year | Work | Application in BTR |
|-----------|------|------|-------------------|
| Parkinson, M. | 1980 | "The Extreme Value Method for Estimating the Variance of the Rate of Return" (Journal of Business) | Parkinson volatility estimator: sigma_P = sqrt(mean(ln(H/L)^2) / (4*ln2)). Used for labeling, entry threshold, SL bounds, TP distance. 5.2x more efficient than close-to-close. |
| Garman, M. & Klass, M. | 1980 | "On the Estimation of Security Price Volatilities from Historical Data" (Journal of Business) | Confirms Parkinson efficiency; provides the family of range-based estimators that our implementation belongs to. |
| Kelly, J. | 1956 | "A New Interpretation of Information Rate" (Bell System Technical Journal) | Risk-budget stop loss: SL derived from max acceptable loss as fraction of capital, analogous to Kelly's optimal bet sizing under uncertainty. |
| Vince, R. | 1992 | "The Mathematics of Money Management" | Risk-budget framework: position sizing and stop placement derived from portfolio-level loss budgets. The 25% max drawdown hard filter reflects Vince's observation that 40%+ DD requires 67%+ recovery -- unrealistic for most strategies. |
| Sweeney, J. | 1993 | "Maximum Adverse Excursion" (Wiley) | MFE/MAE framework: TP design based on favorable excursion beyond prediction. Our TP = |prediction| + 2*sigma captures moves in the tail of the MFE distribution. |
| Grinold, R. & Kahn, R. | 1999 | "Active Portfolio Management" (McGraw-Hill) | Fundamental law of active management: entry threshold and confidence-scaled position sizing. Signal strength should be proportional to bet size. |
| Breiman, L. | 2001 | "Random Forests" (Machine Learning) | Ensemble variance reduction: M diverse models reduce prediction variance by ~1/M. Our 4-fold decay-weighted ensemble follows this principle. |
| Pardo, R. | 2008 | "The Evaluation and Optimization of Trading Strategies" (Wiley) | Walk-forward validation methodology; fitness function design principles; strategy evaluation via rolling metrics. |
| Aronson, D. | 2007 | "Evidence-Based Technical Analysis" (Wiley) | Statistical rigor in strategy evaluation; minimum t-stat thresholds for declaring significance; overfitting detection. |
| Lopez de Prado, M. | 2018 | "Advances in Financial Machine Learning" (Wiley) | Walk-forward with embargo (Ch. 7); triple barrier labeling adapted for execution (Ch. 3); meta-labeling concept; feature lookback ratios; volatility-normalized returns; purged k-fold cross-validation. |
| Milionis, J. et al. | 2022 | "Automated Market Making and Loss-Versus-Rebalancing" | LVR framework for CL fee/loss model: LVR_rate = sigma^2 * sqrt(P) / (8 * LP_value). |
| Chen, T. & Guestrin, C. | 2016 | "XGBoost: A Scalable Tree Boosting System" (KDD) | Histogram-based GBM: diminishing returns past ~10K samples for gradient boosting; informs our train stride optimization. |
| Kaufman, P. | 2013 | "Trading Systems and Methods" (Wiley) | Fitness function component selection; multi-metric strategy evaluation. |
| Harvey, C. et al. | 2016 | "...and the Cross-Section of Expected Returns" (Review of Financial Studies) | t-stat > 2.0 minimum for backtested significance; informs our trajectory t-stat weighting. |
| Kakushadze, Z. | 2016 | "101 Formulaic Alphas" (Wilmott) | Multi-timeframe feature scaling ratios (1:3:10). |
| Coqueret, G. & Guida, T. | 2020 | "Machine Learning for Factor Investing" (CRC Press) | Letting the model select features rather than hand-tuning lookbacks; GBM as implicit feature selector. |
| Hasan, M. | 2024 | "Perpetual Booster: Hyperparameter-Free Gradient Boosting" | Self-generalizing regularization via budget parameter; automatic bias-variance tradeoff without CV. |
| Zheng, A. & Casari, A. | 2017 | "Feature Engineering for Machine Learning" (O'Reilly) | Z-score clipping to prevent gradient boosting gain bias from unbounded features. |
