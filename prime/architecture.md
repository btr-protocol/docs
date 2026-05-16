---
title: "BTR Prime - Architecture"
description: "ML-driven trading platform for crypto spot and concentrated liquidity management."
audience: tech
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: false
---
# BTR Prime - Architecture

ML-driven trading platform for crypto spot and concentrated liquidity management.

**Scope boundary:** BTR consumes information bars produced by NX Rates and produces trading signals executed against exchanges. The unified pipeline uses GBM to predict Parkinson-normalized returns with deterministic sigma-based trading rules -- no chained optimizers. Full ML methodology is documented in the numbered chapters under this directory; see [README](./README.md) for the index.

**Wire/Bar format:** Bars are 96-byte `mitch::Bar` records (64B OHLCV + 32B microstructure); see NX Rates MITCH Bar spec for the canonical layout. The composite TDWAP Index that feeds bar generation is documented in NX Rates aggregation-methodology.md.

---

## Service Inventory

| Service | Instances | Binary | Resources |
|---------|-----------|--------|-----------|
| `optimizer-{strategy}` | 1 per strategy | `btr-ml` | ~2C / 4Gi each |
| `executor-{strategy}` | 1 per strategy | `btr-runtime` | 100m / 128Mi each |
| `web-app` | 1 | Svelte dashboard | - |

### Strategy Naming

`{pair}-{type}` where type is:
- **LO** - Long-Only (buys on signal, exits on reversal or SL/TP)
- **SO** - Short-Only (short on signal; requires leverage via AAVE or CEX margin)
- **CL** - Concentrated Liquidity (manages Uniswap V3-style LP ranges)

Examples: `optimizer-btc-usdt-lo`, `executor-eth-usdt-cl`

---

## Data Flow

```
NX Rates
  └── mitch::Bar mmap files (96B per bar, adaptive Renko)
              │
              ▼
    optimizer-{strategy}  [btr-ml]
      ├── Feature extraction (29 per-bar + 8 cross-asset + 12 Renko-tracker = 49 features)
      ├── Walk-forward GBM (Perpetual, 4-fold decay-weighted ensemble)
      ├── Parkinson-normalized labeling (sigma-scaled returns)
      └── Fitness evaluation (7-component weighted geometric mean)
              │
              ▼
      shared PVC
        ├── model.bin       (trained GBM ensemble)
        ├── signals.json    (historical signal index)
        └── config.yml      (all trading params: SL, TP, leverage, thresholds)
              │
              ▼
    executor-{strategy}  [btr-runtime]
      ├── RenkoStream  (live candles → Renko bricks)
      ├── OptimizerLoop (periodic retrain trigger)
      └── Provider dispatch
              │
      ┌───────┼───────────┐
      ▼       ▼           ▼
   Paper   Binance    Li.Fi / Jumper
  (sim)    Spot REST  (on-chain swaps)
```

---

## Execution Providers

| Provider | Mode | Transport | Use Case |
|----------|------|-----------|----------|
| `PaperProvider` | Paper | In-memory | Forward testing, strategy validation |
| `BinanceSpotProvider` | Live | HMAC-signed REST v3 | Spot BTC/ETH/BNB USDT pairs |
| `LifiProvider` | Live | Li.Fi / Jumper API + on-chain | DeFi CL rebalancing (Arbitrum) |

### Signal Structure

```rust
pub struct MLSignal {
    pub ts: u64,               // Bar timestamp (unix ms)
    pub predicted_return: f64, // Parkinson-normalized predicted return (sigma units)
    pub signal: i8,            // -1 (short), 0 (flat), +1 (long/enter)
    pub confidence: f64,       // |predicted_return| - signal magnitude in sigma units
}
```

---

## Shared State

| State File | Writer | Reader | Update Frequency | Mechanism |
|------------|--------|--------|------------------|-----------|
| `model.bin` | Optimizer | Executor | Hours | Atomic rename |
| `signals.json` | Optimizer | Executor | Hours | Atomic rename |
| `config.yml` | User / Optimizer | All | Manual / per optimization | Single source of truth for all trading params |
| `*.bars` (mitch::Bar mmap) | NX Rates Processor | Optimizer, Executor | Continuous | mmap MAP_SHARED |
| `live.bars` (accumulated) | Executor (OptimizerLoop) | btr-ml subprocess | Per new bar | Append + atomic rename |

---

## Monitoring

BTR web app (`btr.supply`) at `prime/monitor/` provides:
- Equity curves with trade markers (entry/exit triangles)
- CL dual-view: Alpha (fee - LVR - gas) and Absolute $ (LP position value with IL)
- Renko box chart overlaid on price
- Strategy-level Sharpe, Sortino, max drawdown, fitness breakdown
- Rolling monthly metrics table (monthly PF, Sortino, CVaR)

---

## Source Code Layout

```
btr/
├── prime/                      # Rust workspace
│   ├── Cargo.toml
│   ├── config.yml              # Single source of truth for all trading params
│   ├── crates/
│   │   ├── backtest/           # Fitness function (7-component), rolling monthly metrics, stats
│   │   ├── bin/                # btr-prime: strategy orchestrator
│   │   ├── config/             # Domain types: Candle, MlConfig, StrategyMode, Forces
│   │   ├── core/               # Indicators, CMA-ES (CL only), CL simulation, forces model
│   │   ├── execution/          # On-chain: Li.Fi/Jumper swaps, AAVE leverage, alloy tx signing
│   │   ├── ml/                 # btr-ml binary: features, GBM training, Parkinson rules, CL pipeline
│   │   └── runtime/            # btr-runtime binary: Provider trait, 3 providers, RenkoStream, engine
│   ├── monitor/                # Svelte 5 monitoring dashboard (btr.supply)
│   └── data/                   # Bar data files: BTCUSDT_renko.bars, etc.
│
├── docs/                       # This documentation
├── research/                   # Research notes and benchmark results
└── registry/                   # Container registry config
```

### Key Binaries

| Binary | Crate | Purpose |
|--------|-------|---------|
| `btr-ml` | `crates/ml` | Walk-forward GBM optimizer: `btr-ml BTC lo --data path.bars` |
| `train-bars` | `crates/ml` | Standalone .bars trainer: `train-bars path.bars lo --regression` |
| `btr-runtime` | `crates/runtime` | Live trading engine: `btr-runtime --pair BTC-USDT --provider paper` |
| `btr-prime` | `crates/bin` | Strategy orchestrator |

---

## btr-runtime CLI Reference

```
btr-runtime [OPTIONS]

Options:
  --pair <PAIR>              Trading pair: BTC-USDT, ETH-USDT, BNB-USDT  [default: BTC-USDT]
  --mode <MODE>              Strategy mode: lo | so | cl                  [default: lo]
  --provider <PROVIDER>      paper | binance | lifi                       [default: paper]
  --data <PATH>              Historical .bars file (initial optimizer seed)
  --work-dir <DIR>           Working directory for live bars + output      [default: /tmp/btr-runtime]
  --paper-capital <USD>      Paper account initial balance                 [default: 10000]
  --api-key <KEY>            Binance API key (binance provider)
  --api-secret <SECRET>      Binance API secret (binance provider)
  --wallet <ADDR>            Wallet address 0x... (lifi provider)
  --position-size <FRAC>     Fraction of balance per trade                 [default: 0.95]
  --min-confidence <FLOAT>   Minimum ML confidence to enter                [default: 0.1]
  --retrain-every <N>        New Renko bricks between retrains             [default: 500]
  --interval <INTERVAL>      Kline subscription interval                   [default: 1m]

Environment:
  BTR_ML_BIN    Path to btr-ml binary (default: ./target/release/btr-ml)
  RUST_LOG      Log filter (e.g., info, debug)
```

---

## Kubernetes Resources

| Service | Namespace | CPU | Memory |
|---------|-----------|-----|--------|
| `optimizer-{strategy}` | `btr` | ~2C (burst) | 4Gi |
| `executor-{strategy}` | `btr` | 100m | 128Mi |
| `web-app` | `btr` | - | - |

**Storage:** Shared PVC for model.bin + bars files; mmap MAP_SHARED between optimizer and executor.

See cluster-architecture.md for the full infrastructure picture.
