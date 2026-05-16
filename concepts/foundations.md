---
title: "Intellectual & Technical Foundations"
description: "The theoretical lineage and design choices behind AIMM"
audience: both
type: explanation
status: live
phase: n/a
order: 3
lang: en
publish: true
---
# Intellectual & Technical Foundations

> The theoretical lineage and design choices behind AIMM

---

## 1. Introduction

AIMM (Adaptive Inventory Market Maker) synthesizes ideas from quantitative market making theory, ALM-based stableswap design, and concentrated liquidity research. This document traces the intellectual genealogy of each component, acknowledges prior art, and explains our design decisions including paths not taken.

For AIMM's vision and value proposition, see [Manifesto](manifesto.md).

---

## 1.5. ALM Foundations: Vault-Layer Prior Art

> BTR Supply (Phase 1) is an ALM product: a meta-LP layer that allocates single-asset capital across concentrated-liquidity venues. Its design draws on the CLM (Concentrated Liquidity Manager) lineage and on tokenized-vault standards. *Inspired by* the references below (no shared code, no fork).

### 1.5.1. CLM Prior Art

- **Gamma Strategies** - Pioneered automated range management on Uniswap V3. Hypervisor contracts deposit dual-asset LP, expose fungible shares, and rebalance via off-chain keepers. Demonstrated the viability of professional range management as a service.
- **Arrakis Finance** - Originally G-UNI on Uniswap V3; later expanded to multi-venue strategy vaults (Arrakis V2 / PALM). Introduced the keeper-driven programmable ALM (PALM) vault pattern and the role separation between strategy author, executor, and depositor.
- **Kamino Finance** - Solana-native CLM on Orca Whirlpools and Raydium CLMM. Showed that aggressive, frequent rebalancing can be made cost-effective on low-fee chains, and that vault-level autocompounding plus pessimistic share pricing materially improves passive LP outcomes.
- **Beefy CLM** - Multi-chain CL vault aggregator. Demonstrated cross-venue adapter abstraction at scale and standardized the "deposit one asset, vault handles two-sided LP" UX.

BTR Supply takes the **adapter abstraction** (Beefy-class), **role separation + ratchet-down keeper authority** (Arrakis-class), **pessimistic share pricing** (Kamino-class), and **passive single-asset UX** (Gamma-class), then layers on the **worst-of(pool, oracle) NAV rule**, **regime-adaptive range policy**, and **per-vault deposit cap + 8-adapter ceiling**.

### 1.5.2. Tokenized Vault Standards

- **ERC-4626 (Tokenized Vault Standard)** - Canonical interface for yield-bearing vaults: `asset`, `totalAssets`, `convertToShares`, `convertToAssets`, `deposit`, `mint`, `withdraw`, `redeem`. BTR Supply implements ERC-4626 for synchronous flows.
- **ERC-7540 (Asynchronous ERC-4626 Tokenized Vaults)** - Extends ERC-4626 with request/claim semantics for capacity-constrained or settlement-delayed redemptions. BTR Supply uses ERC-7540 for async redeem tickets when sync exit liquidity is gated (intent in flight, kill switch, deviation halt).

### 1.5.3. Single-Asset Vault Model

The single-asset model - one ERC-20 in, one ERC-20 out, two-sided LP managed internally - is the dominant ALM UX. It collapses LP-side risk to one decimal (the deposit asset), aligns with stablecoin- or LST-denominated treasury mandates, and lets the vault, not the user, own the rebalance loop.

### 1.5.4. Regime-Adaptive Rebalancing

Regime-adaptive rebalancing literature combines: (a) realized-volatility-driven range width (Garman-Klass / Parkinson estimators), (b) drift-aware re-centering (EWMA / Kalman filters), (c) cost-of-rebalance gating (LVR vs gas trade-off), and (d) sequencer / oracle health gating. BTR Supply implements regime adaptation at the **vault layer**, not inside the underlying CL pool - preserving compatibility with the unmodified third-party DEX contracts.

---

## 2. Inventory-Based Market Making: The Avellaneda-Stoikov Framework

### 2.1. Origins

The foundation of AIMM's pricing philosophy traces to [Marco Avellaneda and Sasha Stoikov's 2008 paper](https://people.orie.cornell.edu/sfs33/LimitOrderBook.pdf) "High-Frequency Trading in a Limit Order Book." This work formalized how a rational market maker should quote asymmetrically based on inventory risk.

The key insight: a market maker holding excess inventory faces directional exposure. Rather than quoting symmetrically around mid-price, the optimal strategy adjusts quotes to encourage trades that reduce inventory.

### 2.2. The Reservation Price

Avellaneda-Stoikov introduces the **reservation price**:

$$r = s - q \cdot \gamma \cdot \sigma^2 \cdot \tau$$

where:
- $r$ = reservation price
- $s$ = mid-market price
- $q$ = inventory quantity (positive = long, negative = short)
- $\gamma$ = risk aversion parameter
- $\sigma$ = price volatility
- $\tau$ = time remaining (T - t)

> $\tau = T - t$ is the **finite-horizon residual** in the AS-2008 formulation. AIMM, as an infinite-horizon AMM, uses the **stationary-limit reformulation** (Guéant-Lehalle-Fernandez-Tapia 2012) - the $\gamma\sigma^2\tau$ inventory penalty is replaced by its stationary analog.

When inventory $q > 0$ (long position), the reservation price shifts below market, the market maker is willing to sell at lower prices to reduce exposure. The converse applies for short positions.

### 2.3. Optimal Spread

The optimal bid-ask spread depends on volatility, order arrival intensity, and risk aversion:

$$\Delta = \gamma \cdot \sigma^2 \cdot \tau + \frac{2}{\gamma} \cdot \ln\left(1 + \frac{\gamma}{k}\right)$$

where:
- $\Delta$ = optimal bid-ask spread
- $\tau$ = time remaining (T - t)
- $k$ = order arrival intensity parameter

Higher volatility or risk aversion widens the spread; more frequent order flow tightens it.

> **Framework assumptions:** This is the AS-2008 optimal half-width for a finite-horizon market-maker with exponential order-arrival intensity $\lambda = A \cdot \exp(-k\delta)$. For BTR's infinite-horizon AMM, the stationary reformulation (Guéant-Lehalle-Fernandez-Tapia 2012) applies - replace $\gamma\sigma^2\tau$ with the stationary inventory penalty.

### 2.4. AIMM's Adaptation

AIMM translates these continuous-time concepts to discrete blockchain execution:

| Avellaneda-Stoikov | AIMM Analog |
|--------------------|-------------|
| Inventory $q$ | Coverage ratio deviation from target |
| Volatility $\sigma$ | Oracle `fastVolEMA` + `slowVolEMA` |
| Risk aversion $\gamma$ | Per-asset `vega` and `lambda` parameters |
| Order intensity $k$ | Implicit in spline depth calibration |

The pricing model implements inventory-aware market making:
- **Volatility band** $S_v$ corresponds to the spread component
- **Directional surcharge** $U$ implements asymmetric quoting based on coverage impact
- Coverage-improving trades face only symmetric spread; coverage-worsening trades incur surcharge

See [Spread & Fees](../dex/1.%20AIMM/1.1.%20Pricing/1.1.4.%20Spread%20&%20Fees.md) for implementation details.

---

## 3. Asset-Liability Management: Platypus & Wombat

### 3.1. The ALM Innovation

[Platypus Finance](https://medium.com/platypus-finance/platypuss-asset-liability-management-eli5-92a1ee85b17) (2021) introduced **Asset-Liability Management (ALM)** to DeFi, fundamentally departing from traditional AMM accounting.

Traditional AMMs like Uniswap track only **reserves** the tokens currently held. Platypus introduced a dual-ledger model:
- **Assets**: Tokens the pool currently holds
- **Liabilities**: Tokens the pool owes to LPs

This distinction enables:
1. **Single-sided deposits**: LPs deposit one token without forced pairing
2. **Explicit coverage tracking**: Pool health visible as assets/liabilities ratio
3. **Natural supply/demand growth**: Each token accumulates independent liability

### 3.2. Coverage Ratio

The **coverage ratio** = Assets / Liabilities measures pool solvency:

| Coverage | State | Implications |
|----------|-------|--------------|
| > 100% | Overcollateralized | Excess reserves, premium withdrawal value |
| = 100% | Balanced | LPs receive exact deposited amounts |
| < 100% | Undercollateralized | Withdrawal haircut applies |

Platypus and later [Wombat Exchange](https://www.wombat.exchange/Wombat_Whitepaper_Public.pdf) proved this model works at scale for stableswaps.

### 3.3. Limitations of First-Generation ALM

Platypus and Wombat use coverage ratio as a **slippage modifier** on a modified invariant curve. The pricing mechanism remains constrained by curve geometry coverage affects *where* you trade on the curve, not *how* the curve is shaped.

From [Wombat Whitepaper](https://medium.com/wombat-exchange/the-wombat-whitepaper-tl-dr-ab4be5dbf71d):

> "Wombat introduces a new invariant curve and the concept of asset-liability to remove scalability barriers."

The invariant curve remains central. Coverage modulates pricing within that constraint.

### 3.4. AIMM's Extension

AIMM extends ALM in three directions:

**First**, we decouple price from reserves entirely. Price derives from oracles and spline profiles; reserves determine *depth*, not *direction*.

**Second**, we implement **coverage-aware fee asymmetry**. Avellaneda-Stoikov's inventory adjustment applies through differential fees, not price distortion.

**Third**, we add **liability decay** for prolonged undercollateralization. When coverage remains critically low, liabilities decay over time to restore solvency preventing permanent insolvency spirals.

See [Inventory Management](../dex/1.%20AIMM/1.1.%20Pricing/1.1.1.%20Inventory%20Management.md) for coverage mechanics.

---

## 4. Oracle-Guided Pricing: Swaap's Matrix Market Maker

### 4.1. The Swaap Innovation

[Swaap Finance](https://www.swaap.finance/whitepaper.pdf) introduced the **Matrix Market Maker (MMM)** a stochastic, asymmetric, oracle-guided, multi-asset AMM. Swaap's key insight: decouple price discovery from market making.

From [Swaap's introduction](https://www.swaap.finance/blog/introducing-swaap-protocol):

> "Swaap's v1 model, the MMM, is an oracle-guided, multi-asset AMM. Its goal is to function like a passive Index ETF, aiming for near-zero impermanent loss by decoupling price discovery (from oracles) from the act of market-making."

### 4.2. Stochastic Spread Mechanism

Rather than fixed fees, Swaap adjusts spreads based on real-time volatility:

> "Instead of simple volume-based fees, Swaap uses a 'stochastic spread mechanism.' This model adjusts fees based on real-time market volatility data, protecting liquidity providers during turbulent periods by charging higher premiums for riskier trades."

This directly influenced AIMM's volatility band $S_v$.

### 4.3. Geometric Mean Product

Swaap's constant geometric mean product enables multi-asset pools where each asset maintains target weight:

$$\prod_i x_i^{w_i} = k$$

where:
- $x_i$ = quantity of asset i
- $w_i$ = weight of asset i
- $k$ = constant product

This allows portfolio-like behavior; the pool rebalances to maintain allocations.

### 4.4. Swaap v2 Architecture

[Swaap v2](https://www.swaap.finance/v2-whitepaper.pdf) evolved toward RfQ (Request for Quote) infrastructure:

> "Swaap Maker is a non-custodial RfQ market-making infrastructure. It provides optimal liquidity services with built-in defensive modules or 'safeguards' allowing for on-chain max drawdown circuit breaker, last look, and other dynamic forms of funds protection."

This hybrid on-chain/off-chain model prioritizes professional market makers over passive LPs.

### 4.5. AIMM's Divergence

AIMM shares Swaap's oracle-first philosophy but differs architecturally:

| Aspect | Swaap | AIMM |
|--------|-------|------|
| **Price source** | External oracles | Internal TWAP primary + Chainlink base-token depeg circuit-breaker (not a quote fallback) |
| **Pool structure** | Weighted portfolio | Coverage-based ALM |
| **Spread mechanism** | Stochastic (vol-based) | Bi-factor (vol + deviation) + inventory skew |
| **Target users** | Professional market makers | Passive LPs |

AIMM's internal oracle (dual-window TWAP) reduces external dependencies while Swaap relies on Chainlink/Pyth feeds.

---

## 5. Dynamic Pegging: Curve v2 Cryptoswap

### 5.1. The Repegging Innovation

[Curve v2 Cryptoswap](https://docs.curve.finance/assets/pdf/whitepaper_cryptoswap.pdf) introduced **dynamic repegging** automatically adjusting the concentrated liquidity center based on an internal price oracle.

From [Curve documentation](https://docs.curve.finance/cryptoswap-exchange/overview/):

> "Thanks to a repegging algorithm which is run after each swap, Curve V2 allows swaps to occur near the equilibrium point."

### 5.2. Internal Oracle

Curve v2 maintains an exponential moving average (EMA) of trade prices:

> "Internally, Curve v2 has a price oracle given by an exponential moving average applied in N-dimensional price space."

This internal oracle determines where to concentrate liquidity, not just how to price trades.

### 5.3. Profit-Loss Accounting

Cryptoswap tracks `xcp_profit` and `xcp_profit_real` to ensure repegging doesn't destroy LP value:

> "It undoes p adjustment if it causes xcp_profit_real-1 to fall lower than half of xcp_profit-1."

This constrains curve updates to scenarios where accumulated fee revenue exceeds incurred losses.

### 5.4. Permanent Loss Risk

Unlike Uniswap's "impermanent" loss (recoverable if price returns), Curve v2 can cause **permanent** loss:

> "Crypto Pool is vulnerable to permanent loss (which cannot be recovered even if the price does) due to changes in the curve itself when repegging the equilibrium price."

The curve shape changes, locking in losses even if prices recover.

### 5.5. AIMM's Approach

AIMM borrows Curve's internal oracle concept but avoids dynamic invariant updates:

| Aspect | Curve v2 | AIMM |
|--------|----------|------|
| **Oracle** | Single EMA | Dual-window TWAP (5min + 1hr) |
| **Curve updates** | Automatic repegging | Static spline (curator-managed) |
| **Loss type** | Permanent (curve changes) | Impermanent (reserves change) |
| **Newton iteration** | ~35k gas per swap | None (direct spline evaluation) |

AIMM's static spline profiles avoid permanent loss from curve drift while maintaining oracle-aware pricing.

See [Oracles](../dex/3.%20Security/3.5.%20Oracles.md) for dual-window TWAP design.

---

## 6. Concentrated Liquidity: Gyroscope E-CLP

### 6.1. Elliptical Concentration

[Gyroscope's E-CLP](https://docs.gyro.finance/pools/e-clps) (Elliptic Concentrated Liquidity Pool) uses elliptical curves for asymmetric liquidity concentration:

> "Elliptic CLPs allow trading along the curve of an ellipse. Similar to other CLPs, E-CLPs are designed to concentrate liquidity within price bounds."

The ellipse is formed by transforming a circle:
- **Stretch** ($\lambda$): Elongates the curve
- **Rotation** ($\phi$): Tilts the concentration
- **Displacement** ($\alpha$, $\beta$): Shifts price bounds

### 6.2. Capital Efficiency

E-CLPs achieve significant efficiency gains over StableSwap:

> "By putting liquidity only where it is needed, E-CLPs can improve capital efficiency by upwards of 75% over StableSwap pools."

### 6.3. Passive Management

Unlike Uniswap v3's LP-managed positions, E-CLP parameters are set by pool deployers:

> "Unlike any other form of concentrated AMM curve, Gyroscope's E-CLPs provide passive liquidity management; instead of users setting price bounds themselves, the pool deployer takes on the responsibility of calibrating and establishing the trading parameters upon launch."

This aligns with AIMM's curator model.

### 6.4. Limitations

Elliptical curves have geometric constraints:

1. **Single-peaked distribution**: Cannot create bimodal liquidity profiles
2. **Smooth concentration**: Cannot create sharp edges or plateaus
3. **Symmetric options limited**: Rotation helps but doesn't enable arbitrary asymmetry

E-CLPs approximate most desired curves but cannot express:
- Double-peaked distributions (liquidity at two price points)
- Flat regions with sharp cutoffs
- Inverse or concave profiles

### 6.5. AIMM's Alternative

Spline-based profiles overcome elliptical limitations:

| Capability | E-CLP | AIMM Splines |
|------------|-------|--------------|
| **Single peak** | Yes | Yes |
| **Double peak** | No | Yes |
| **Flat regions** | Limited | Yes |
| **Sharp cutoffs** | No | Yes (via tension) |
| **Arbitrary shapes** | No | Yes (monotone cubic Hermite) |

See [Liquidity Shaping](../dex/1.%20AIMM/1.1.%20Pricing/1.1.2.%20Liquidity%20Shaping.md) for spline mechanics.

---

## 7. Volatility-Based Fees: LFJ v2 Liquidity Book

### 7.1. The Surge Pricing Innovation

[LFJ's Liquidity Book](https://github.com/traderjoe-xyz/LB-Whitepaper) (formerly Trader Joe v2) introduced **surge pricing** dynamic fees that increase with market volatility:

> "Liquidity Book introduces a fee structure that has two components, a base fee and a variable fee. The variable fee is adjusted to account for volatility. The more volatile the assets are in a Liquidity Pool, the higher the variable fee will be."

### 7.2. Volatility Accumulator

LFJ measures instantaneous volatility through a **Volatility Accumulator (VA)** without external oracles:

> "The VA is able to calculate instantaneous volatility for each Liquidity Pool, without relying on any outside oracles by tracking transactions across bins."

Bin crossings indicate price movement; more crossings per trade = higher volatility.

### 7.3. Rationale

Surge pricing compensates LPs during periods of maximum adverse selection:

> "Impermanent Loss can be viewed as a cost of the price discovery, so it is highest during the most volatile times when the market tries to correctly price assets. Surge Pricing generates additional fees from trades, used to compensate LPs for the Impermanent Loss they experience during market volatility."

### 7.4. AIMM's Implementation

AIMM's volatility band directly implements surge pricing concepts:

$$S_v = 100 + \frac{\sigma_p \cdot \nu}{100M}$$

where:
- $S_v$ = volatility band (basis points)
- $\sigma_p$ = pair volatility
- $\nu$ = vega sensitivity
- $M$ = multiplier base (10000)

Key differences:

| Aspect | LFJ v2 | AIMM |
|--------|--------|------|
| **Volatility measurement** | Bin crossings (VA) | Oracle EMA ($\sigma$) |
| **Fee components** | Base + variable | Base + volatility + inventory |
| **Inventory awareness** | No | Yes (directional surcharge) |
| **Multi-hop aggregation** | Per-hop | Path-level max |

---

## 8. CLMM/DLMM Trade-offs and Their Mitigations

> Mainstream concentrated-liquidity DEXs have addressed several of these via V4 hooks, custom curves, and active management. BTR Supply itself implements these mitigations as an integrator/LP layer over UniV3/V4.

### 8.1. The Concentrated Liquidity Tradeoff

Uniswap v3's concentrated liquidity provides capital efficiency but introduces significant challenges:

**Amplified Impermanent Loss**: Narrower ranges magnify IL when prices move out of range:

> "Concentrated positions can experience more significant impermanent loss than V2 positions if prices move beyond your range."   [Kaiko Research](https://research.kaiko.com/insights/jit-2)

**Active Management Burden**: LPs must continuously rebalance:

> "LPs, especially smaller LPs, are constrained in the degree to which they can actively manage their concentrated liquidity positions. As a result, their positions tend to drift out of range, suffering from IL and failing to capture any trading fees."

**Net Losses for Most LPs**: Research indicates majority of LPs underperform holding:

> "Between V3's launch and September 20th, analyzed pools saw over $100B in trading volume, earning LPs approximately $200M in fees. However, LPs lost more than $260M to impermanent loss, resulting in a net loss of over $60M."   [Rekt News](https://rekt.news/uniswap-v3-lp-rekt)

### 8.2. JIT Liquidity Dynamics

Concentrated liquidity enables **Just-In-Time (JIT) liquidity** adding massive liquidity before a large trade, capturing fees, and immediately removing it:

> "Just In Time Liquidity is an occurrence unique to Uniswap V3's concentrated liquidity. A large amount of liquidity is added by a JIT bot when they see a large trade in the mempool. It is then removed immediately after within the same block."

JIT bots extract value from passive LPs:

> "This liquidity added before the trade reduces price impact for the trader by increasing the pool size, but dilutes the trading fees distributed amongst existing LPs."

Research identified 36,671 JIT attacks over 20 months generating 7,498 ETH profit.

### 8.3. Capital Intensity

JIT is "a whales' game":

> "JIT transactions are significantly larger than regular transactions, generally with a minimum size of $10 million."

Retail LPs cannot compete with sophisticated market makers.

### 8.4. AIMM's Mitigations

AIMM addresses these issues structurally:

| LP concern | CLMM/DLMM | AIMM |
|---------|-----------|------|
| **Active management** | manual | Curator-managed profiles |
| **IL amplification** | range-dependent | Coverage-based (reserves change, not curve) |
| **JIT exposure** | hook-mitigated | Reduced (no discrete tick crossings) |
| **LP sophistication** | range-dependent | Not required |

See [Toxic Flow Mitigation](../dex/1.%20AIMM/1.1.%20Pricing/1.1.6.%20Toxic%20Flow%20Mitigation.md) for detailed defenses.

---

## 9. Anchor Path Pricing

### 9.1. The Multi-Asset Routing Problem

Multi-asset pools face combinatorial explosion: N tokens require N(N-1)/2 pairs for direct trading. Solutions include:

**Approach 1: All-pairs liquidity** (Balancer)
- Every pair has explicit depth
- Geometric mean invariant across all assets
- Capital inefficient for large N

**Approach 2: Base currency routing** (Traditional FX)
- All quotes relative to base (USD)
- $A \to B$ trades route through $A \to \text{USD} \to B$
- Standard in foreign exchange markets

**Approach 3: Graph-based routing** (DEX aggregators)
- Build graph of available pairs
- Find optimal path considering liquidity and fees
- External computation, not pool-native

### 9.2. AIMM's Anchor Tree

AIMM uses a **hub-and-spoke anchor tree** where a base token (typically stablecoin) serves as the routing hub:

```
        [USDC]
       /  |  \
    ETH  BTC  DAI
     |
   WSTETH
```

All swaps route through the anchor:
- $\text{ETH} \to \text{BTC}$ routes as $\text{ETH} \to \text{USDC} \to \text{BTC}$
- $\text{WSTETH} \to \text{DAI}$ routes as $\text{WSTETH} \to \text{ETH} \to \text{USDC} \to \text{DAI}$

### 9.3. Precedents

This structure mirrors traditional finance:
- **FX markets**: Cross-rates derived from USD pairs
- **Equity markets**: Market makers quote against cash
- **Crypto exchanges**: BTC or USDT as base pairs

The innovation is on-chain implementation with **Least Common Ancestor (LCA) pathfinding** mathematically optimal routing with $O(\log N)$ computation.

### 9.4. Benefits

| Aspect | All-Pairs (Balancer-style) | Anchor Tree |
|--------|---------------------------|-------------|
| **Pair / pool count (deployment)** | $O(N^2)$ | $O(N)$ |
| **Quote complexity (per swap)** | $O(1)$ direct, $O(N)$ to optimise | $O(\log N)$ LCA on balanced tree, bounded-depth → $O(1)$ in practice |
| **Capital efficiency** | Diluted | Concentrated at anchor |
| **Gas cost** | Per-pair depth | Single traversal |
| **Adding tokens** | $N$ new pairs | 1 new edge |

For a side-by-side complexity breakdown including the hub-and-spoke variant, see [Anchor Path Pricing - Key Advantages](../dex/1.%20AIMM/1.1.%20Pricing/1.1.3.%20Anchor%20Path%20Pricing.md#key-advantages).

See [Anchor Path Pricing](../dex/1.%20AIMM/1.1.%20Pricing/1.1.3.%20Anchor%20Path%20Pricing.md) for implementation.

---

## 10. Circular/Orbital Market Makers

### 10.1. Polar Coordinate AMMs

Recent research explores AMMs using polar coordinates on an n-dimensional sphere. The polar-tick mathematics and sphere/superellipse invariants in this section trace to [Paradigm's Orbital paper](https://www.paradigm.xyz/2024/06/orbital) and academic work on [Concentrated Circular Market Makers (CCMM)](https://arxiv.org/abs/2510.05428) (Tolstikov et al.). [Orbswap](https://orbswap.org/lite-paper) is one implementing DEX; its lite-paper presents the formulas as images and is not the canonical mathematical source - citations below are to Paradigm/CCMM.

**Sphere invariant** (n stablecoins, reserves $x_i$, radius $r$):

$$\sum_{i=1}^{n}(r - x_i)^2 = r^2$$

The equal-price point sits at $q = r(1 - 1/\sqrt{n})$ along the diagonal $\vec{v} = \frac{1}{\sqrt{n}}(1,\dots,1)$. Any reserve vector decomposes as $\vec{x} = \alpha\vec{v} + \vec{w}$ with $\vec{w} \perp \vec{v}$, giving the constraint $r^2 = (\alpha - r\sqrt{n})^2 + \|\vec{w}\|^2$. Instantaneous marginal price between assets $i, j$:

$$\frac{\partial x_i}{\partial x_j} = \frac{r - x_j}{r - x_i}$$

**Polar ticks** are hyperplanes $\vec{x}\cdot\vec{v} = k$. On a tick boundary the pool degenerates to a lower-dimensional sphere of radius $s = \sqrt{r^2 - (k - r\sqrt{n})^2}$. Orbswap's litepaper extends this to a **superellipse** invariant for tunable concentration; v1 introduces "superellipse ticks for concentrated liquidity."

### 10.2. The Torus Model

Combining an interior spherical tick with a lower-dimensional boundary sphere yields a torus:

$$r_{int}^2 = (\vec{x}\cdot\vec{v} - k_{bound} - r_{int}\sqrt{n})^2 + (\|\vec{x} - (\vec{x}\cdot\vec{v})\vec{v}\| - \sqrt{r_{bound}^2 - (k_{bound} - r_{bound}\sqrt{n})^2})^2$$

> "By rotating the sphere around the circle, we obtain a single torus, or donut shape."

Reported capital efficiency vs Curve for n=5 stables (per the Paradigm Orbital paper): ~15× at a 0.90 depeg threshold, ~150× at 0.99 - pegged-only in v1; LST extensions are theoretical. Risk-isolation claim (Paradigm Orbital / Orbswap litepaper): *"if one of the stablecoins depegs, the others can all trade at efficient prices, while the depegged one will become worthless much faster than a traditional AMM curve."* This is structural: the sphere geometry penalizes any single coordinate diverging far from the diagonal, so a depeg asymmetrically drains the bad asset rather than dragging the pool.

### 10.3. Why AIMM Didn't Pursue This

Despite theoretical elegance, circular/orbital approaches face practical challenges:

**Mathematical Complexity**:
- Polar coordinate transforms add computational overhead
- Torus invariants require more complex Newton iterations
- Debugging and auditing more difficult

**Limited Flexibility**:
- Optimized for symmetric stablecoin pools
- Asymmetric liquidity profiles harder to express
- Assumes correlated assets (stablecoins)

**LP Comprehension**:
- Polar "ticks" less intuitive than price ranges
- Visualization tools more complex
- Position management opaque

**AIMM's Alternative**:
- Cartesian splines are computationally simpler
- Arbitrary liquidity profiles via monotone cubic Hermite (Fritsch-Carlson) interpolation
- Direct price/depth correspondence
- Easier visualization and debugging

**Honest positioning**: Orbital/Orbswap and AIMM occupy adjacent-but-distinct niches.

| Dimension | Orbital / Orbswap (CCMM) | AIMM (BTR) |
|---|---|---|
| **Asset scope** | Pegged only (stables, can extend to LSTs at matching pegs) | Any volatility profile, mixed pools |
| **Risk isolation** | Intrinsic via sphere geometry - depeg drains the bad asset | Via anchor-tree topology - bad branch isolated, but shares oracle path |
| **Capital efficiency (stables)** | ~15-150× Curve at near-peg ticks | Spline-shaped, comparable near peg, broader off-peg |
| **Capital efficiency (mixed-vol)** | N/A (invariant breaks for volatile pairs) | Native via anchor-path pricing |
| **Multi-hop / N-asset routing** | Implicit through n-sphere | Explicit O(log N) LCA pathfinding |
| **Oracle dependency** | None (curve-derived prices) | Yes (oracle mid + inventory skew) |
| **Math elegance** | High - closed-form invariant | Lower - monotone cubic Hermite + Avellaneda-Stoikov composite |
| **Asymmetric depth** | Hard (sphere is symmetric) | Native (per-knot spline control) |

For a pure stables / pure LST pool, Orbital is likely the better technical primitive. For mixed-volatility deployments (USDC + WBTC + WETH + LSTs together), AIMM is the only viable design here - the spherical invariant cannot price volatile-vs-pegged. Where the two **do** compete is BTR's stables-pool deployments; there Orbswap has a real edge on intrinsic depeg isolation, and AIMM compensates via oracle-sync, inventory feedback, and regime-adaptive fees rather than via geometry.

---

## 11. Spline-Based Liquidity Profiles

### 11.1. Novel Contribution

AIMM's spline-based liquidity profiles appear to be **novel** in DeFi AMM design. Extensive literature search found no prior implementations of monotone cubic Hermite or similar interpolating splines for AMM pricing functions.

### 11.2. Mathematical Foundation

AIMM uses the [Fritsch-Carlson monotone cubic Hermite interpolation](https://en.wikipedia.org/wiki/Monotone_cubic_interpolation) (1980), not Catmull-Rom. The implementation in `Spline.sol::_tangents` applies Fritsch-Carlson **sign-preservation** on adjacent secants plus the `α² + β² ≤ 9` tangent-magnitude clamp, which guarantees the interpolant is monotone between knots - a hard requirement for an AMM depth profile (a non-monotone segment would imply negative marginal liquidity, breaking pricing). Catmull-Rom has C1 continuity but is **not** monotone in general; it can overshoot between control points.

Properties relevant to AMM design:
- **Passes through control points**: LP-intuitive (price$\to$depth mapping exact at knots)
- **C1 continuous**: No discontinuous jumps in liquidity
- **Local control**: Moving one knot doesn't affect distant regions
- **Monotone by construction**: Fritsch-Carlson sign-preservation + magnitude clamp prevents overshoot

### 11.3. Advantages Over Alternatives

| Curve Type | Pros | Cons | AMM Examples |
|------------|------|------|--------------|
| **Constant product** | Simple, proven | No concentration | Uniswap v2 |
| **StableSwap** | Good for pegs | Single parameter (A) | Curve v1 |
| **Concentrated ticks** | High efficiency | Active management | Uniswap v3/v4 |
| **Elliptical** | Asymmetric concentration | Limited shapes | Gyroscope |
| **Splines** | Arbitrary profiles | Curator trust | AIMM |

Splines enable liquidity profiles impossible with parametric curves:
- Multi-modal distributions (liquidity at multiple price points)
- Flat regions with sharp edges
- Asymmetric tails
- Profiles fitted to historical trade density

### 11.4. Implementation

AIMM uses **monotone cubic Hermite interpolation** for guaranteed monotonicity:

```
Given knots: [(price_0, depth_0), (price_1, depth_1), ...]
For query price p:
  1. Binary search to find bracketing knots
  2. Hermite interpolation between knots
  3. Return interpolated depth
```

Gas cost: $O(\log k)$ binary search (`Spline.sol::_search`) + ~4 knots touched per segment (p0/p1 + neighbor tangents for Hermite interpolation), where $k$ = number of profile knots (typically 3-7).

See [Liquidity Shaping](../dex/1.%20AIMM/1.1.%20Pricing/1.1.2.%20Liquidity%20Shaping.md) for spline mathematics.

---

## 12. Synthesis: How AIMM Combines Prior Art

### 12.1. Component Attribution

| AIMM Feature | Primary Influence | Secondary Influence |
|--------------|-------------------|---------------------|
| Inventory-aware pricing | Avellaneda-Stoikov | Swaap MMM |
| Coverage ratio | Platypus/Wombat |   |
| Liability accounting | Platypus |   |
| Internal TWAP oracle | Curve v2 | Uniswap v3 TWAP |
| Dual-window smoothing | Novel (defense against manipulation) |   |
| Volatility fees | LFJ v2 Surge | Swaap stochastic spread |
| Directional surcharge | Novel (combines AS + coverage) |   |
| Spline profiles | Novel | Computer graphics literature |
| Anchor tree routing | FX markets | Balancer weighted pools |
| Curator model | Gyroscope E-CLP | Curve pool deployers |

### 12.2. Novel Contributions

AIMM's original contributions:

1. **Bi-factor fees + inventory skew**: Volatility band + directional surcharge for fees; inventory skew adjusts mid-price
2. **Spline-based liquidity profiles**: First known AMM using interpolating splines
3. **Dual-window TWAP with offset encoding**: Compact representation of fast/slow divergence
4. **Coverage-aware asymmetric fees**: Avellaneda-Stoikov inventory adjustment via fee mechanics
5. **Anchor tree + LCA pathfinding**: Efficient multi-asset routing without $N^2$ pairs

### 12.3. Design Philosophy

AIMM takes an **opinionated** stance on DeFi AMM design:

> Most liquidity providers don't want to run market-making strategies. They want to deposit capital and earn yield.

This drives choices away from LP flexibility (Uniswap v3) toward curated, professionally-managed liquidity profiles.

---

## 13. RFQ & Intent-Based Protocols

### 13.1. The Intent Paradigm

A significant trend in DeFi trading has been the rise of **intent-based protocols** that outsource execution to specialized solvers or market makers. Rather than trading directly against an AMM, users express desired outcomes and solvers compete to fulfill them.

**Key Intent/RFQ Protocols**:

| Protocol | Mechanism | Centralization Point |
|----------|-----------|---------------------|
| **CoW Swap** | Batch auction with solver competition | Solver whitelist, off-chain matching |
| **UniswapX** | Dutch auction with fillers | Filler network, off-chain discovery |
| **1inch Fusion** | Auction-based order fulfillment | Resolver network, off-chain pricing |
| **0x RFQ** | Request-for-quote from market makers | Market maker API, off-chain quotes |
| **Hashflow** | Signed quotes from professional MMs | MM signatures, off-chain pricing |
| **DeBridge** | Cross-chain intent fulfillment | Solver network, multi-chain |
| **NEAR Intents** | Chain-abstracted intent execution | Relayer network |

### 13.2. Tradeoffs of Intent Systems

**Advantages**:
- Better execution for sophisticated users
- MEV protection through batch auctions
- Cross-chain composability

**Disadvantages**:
- **Solver centralization**: Limited number of active solvers
- **Off-chain dependencies**: Quote servers, APIs, signature aggregation
- **Permissioned access**: Not anyone can become a solver
- **Latency requirements**: Solvers need fast infrastructure
- **Regulatory surface**: Identifiable entities providing quotes

### 13.3. Why AIMM Chose Not to Go This Route

Intent protocols sacrifice **permissionlessness** for execution quality. AIMM maintains:

- **Anyone can provide liquidity** without approval
- **Anyone can trade** without solver intermediation
- **No off-chain infrastructure** required for operation
- **No whitelisted participants** at any layer

The tradeoff: slightly worse execution during extreme conditions vs. full decentralization at all times.

---

## 14. Oracle-Based & Hybrid AMMs

### 14.1. The Oracle-AMM Design Space

A parallel trend has been AMMs that use oracles for proactive pricing rather than reactive reserve-based discovery. These designs attempt to reduce LVR by pricing closer to "true" market value.

**Oracle-Based AMM Research**:

The [UAMM paper](https://arxiv.org/abs/2402.01928) ("UAMM: Price-oracle based Automated Market Maker") formalizes this design space, showing that oracle-based pricing can theoretically eliminate LVR at the cost of oracle dependency.

### 14.2. Hybrid Protocol Comparison

| Protocol | Pricing Mechanism | Centralization Point | LVR Mitigation |
|----------|-------------------|---------------------|----------------|
| **DODO PMM** | Proactive MM with oracle | External oracle dependency | Good |
| **Swaap v1** | Oracle-based stochastic spread | Chainlink/Pyth dependency | Good |
| **Swaap v2** | Off-chain pricing engine | Off-chain quoter, RFQ hybrid | Excellent |
| **Valantis HOT** | RFQ solver as MEV blocker | Solver whitelist | Excellent |
| **Balancer Weighted (oracle)** | Oracle price feeds for weights | Oracle dependency | Moderate |
| **Balancer LSD pools** | LST price feeds | Oracle dependency | Moderate |
| **Bancor v2/v3** | Off-chain "fair value" feeds | Bancor-controlled oracle | Good |

### 14.2.1. DODO PMM Reference Formula

DODO's PMM curve ([2020 whitepaper](https://dodoex.github.io/docs/docs/pmm), §3):

$$ P_{excess} = i \cdot \left(1 - k + k \cdot (B_0/B)^2\right), \quad B > B_0 $$
$$ P_{deficit} = \frac{i}{1 - k + k \cdot (B/B_0)^2}, \quad B < B_0 $$

where $i$ is the oracle/market mid-price, $B$ is base reserves, $B_0$ is target base reserves (set by oracle), and $k \in [0, 1]$ controls slippage curvature. AIMM's `computeInventorySkew` is inspired by this excess/deficit symmetry but uses a piecewise-linear surrogate over coverage ratio $c$, not a quadratic over reserve ratio $B/B_0$.

### 14.3. The Oracle Dependency Problem

All oracle-based designs share a fundamental vulnerability:

**If the oracle fails, the AMM fails.**

Failure modes include:
- **Oracle manipulation**: Flash loan or multi-block attacks
- **Oracle latency**: Stale prices during volatility
- **Oracle downtime**: No trading possible
- **Oracle compromise**: Malicious price injection

Hybrid designs (Swaap v2, Valantis) add solver/quoter dependency on top of this.

### 14.4. AIMM's Middle Path

AIMM learned from these designs that **fighting price lag is critical for reducing LVR**. However, we reject external oracle dependency as the solution.

**AIMM's approach**:

1. **Internal TWAP** (dual-window) as primary price reference
2. **External oracle** as optional fallback, not requirement
3. **Inventory skew** pricing rather than oracle-based mid-price
4. **Cooperative Arbitrage** (proposed) to internalize stat arb without oracle dependency

Pool prices remain **reactive** (not pro-active), but:
- Inventory skew discourages adverse selection
- (proposed) Cooperative Arbitrage **would** accelerate price convergence via whitelisted cooperators
- No single point of oracle failure

The result: LVR mitigation comparable to oracle-based systems, without their failure modes.

---

## 15. Permissionless Design Philosophy

### 15.1. Core Principle

AIMM holds that **permissionless operation is non-negotiable** for a DeFi primitive. Any centralization creates:

- **Regulatory surface**: Identifiable operators can be compelled
- **Single points of failure**: Downtime, hacks, key compromise
- **Rent extraction**: Gatekeepers can extract value
- **Censorship risk**: Transactions can be blocked

### 15.2. What "Permissionless" Means for AIMM

| Layer | Permissionless? | Details |
|-------|-----------------|---------|
| **Liquidity provision** | ✅ Yes | Anyone can deposit, no whitelist |
| **Trading** | ✅ Yes | Anyone can swap, no KYC |
| **Price discovery** | ✅ Yes | Internal TWAP, no external dependency |
| **Pool creation** | ✅ Yes | Anyone can deploy pools |
| **Cooperative Arbitrage** | ⚠️ Whitelisted | (roadmap) DAO-curated cooperators w/ competitive rebates (see §15.4) |
| **Governance** | ✅ Yes | Token-based, on-chain voting |

### 15.3. Contrast with Centralized Alternatives

**Intent/RFQ Systems** (CoW, UniswapX, 1inch Fusion):
- ❌ Solver/filler whitelist required
- ❌ Off-chain infrastructure dependency
- ❌ Centralized matching/auction

**Oracle-Based AMMs** (DODO, Swaap v1, Bancor):
- ❌ Oracle provider dependency
- ❌ Oracle feed curation
- ⚠️ Fallback mechanisms vary

**Hybrid Systems** (Swaap v2, Valantis):
- ❌ Off-chain quoter dependency
- ❌ Solver/market maker whitelist
- ❌ Complex trust assumptions

**AIMM**:
- ⚠️ (roadmap) Cooperative Arbitrage would be whitelisted via DAO curation
- ✅ Trading remains fully permissionless
- ✅ No off-chain dependencies for pricing
- ✅ Optional (not required) external oracles

### 15.4. The Cooperative Arbitrage Model

> 🚧 **FUTURE WORK - NOT YET IMPLEMENTED.** Cooperative Arbitrage is a designed feature on the BTR DEX roadmap, not yet shipped on-chain. Feature target: post-mainnet, phase TBD.

Cooperative Arbitrage (proposed) would balance permissionless trading with curated arbitrageur access: trading remains fully permissionless for anyone, while a whitelisted Cooperator program would earn rebates proportional to a donations/rebates reputation score. Cooperators use the same swap interface as everyone, the whitelist controls rebate eligibility, not trading access. This differs from solver/filler whitelists (CoW, UniswapX) where non-whitelisted users cannot access certain execution paths.

For the full mechanism, parameters, and rationale, see [Manifesto §5.4 Cooperative Arbitrage](manifesto.md#54-cooperative-arbitrage).

### 15.5. Implications for Resilience

A fully permissionless design means:

- **No regulatory chokepoint**: No single entity to subpoena
- **No key compromise risk**: No privileged operators
- **No downtime risk**: No off-chain services to fail
- **Geographic resilience**: Works from any jurisdiction
- **Censorship resistance**: No transaction filtering

These properties matter for DeFi's long-term survival as critical financial infrastructure.

---

## 16. Acknowledgments

AIMM builds on decades of research:

**Academic Foundations**:
- Avellaneda & Stoikov (2008): "High-Frequency Trading in a Limit Order Book," *Quantitative Finance* 8(3), [DOI:10.1080/14697680701381228](https://doi.org/10.1080/14697680701381228) · [Cornell mirror](https://people.orie.cornell.edu/sfs33/LimitOrderBook.pdf)
- Guéant, Lehalle & Fernandez-Tapia (2012): [Optimal Portfolio Liquidation with Limit Orders](https://arxiv.org/abs/1206.4810)
- Fritsch & Carlson (1980): Monotone Piecewise Cubic Interpolation, *SIAM J. Numer. Anal.* 17(2)
- Parkinson (1980): "The Extreme Value Method for Estimating the Variance of the Rate of Return," *Journal of Business* 53(1)
- Garman & Klass (1980): "On the Estimation of Security Price Volatilities from Historical Data," *Journal of Business* 53(1)
- Milionis, Moallemi, Roughgarden & Zhang (2022): [Automated Market Making and Loss-Versus-Rebalancing](https://arxiv.org/abs/2208.06046); see also the [CowSwap LVR exposition](https://docs.cow.fi/cow-amm/concepts/the-problem-of-lvr)

**DeFi Protocols**:
- [Uniswap](https://uniswap.org): Constant product AMMs, concentrated liquidity
- [Curve Finance](https://curve.fi): StableSwap, Cryptoswap, internal oracle
- [Platypus Finance](https://platypus.finance): Asset-Liability Management
- [Wombat Exchange](https://wombat.exchange): Coverage ratio stableswap
- [Swaap Finance](https://swaap.finance): Oracle-guided, stochastic spread
- [Gyroscope](https://gyro.finance): Elliptic CLPs, passive curator model
- [LFJ (Trader Joe)](https://lfj.gg): Liquidity Book, surge pricing
- [Paradigm](https://paradigm.xyz): Orbital AMM research

**Research**:
- [Milionis et al.](https://arxiv.org/abs/2208.06046): LVR formalization
- [Adams et al.](https://uniswap.org/whitepaper-v3.pdf): Concentrated liquidity
- [Egorov](https://curve.fi/files/crypto-pools-paper.pdf): Cryptoswap

---

## 17. Further Reading

- [AIMM Architecture](../dex/1.%20AIMM/Overview.md), System architecture
- [Manifesto](manifesto.md), Vision and value proposition
- [Spread & Fees](../dex/1.%20AIMM/1.1.%20Pricing/1.1.4.%20Spread%20&%20Fees.md), Bi-factor fee model
- [Oracles](../dex/3.%20Security/3.5.%20Oracles.md), Dual-window TWAP design
- [Liquidity Shaping](../dex/1.%20AIMM/1.1.%20Pricing/1.1.2.%20Liquidity%20Shaping.md), Spline profiles
- [Toxic Flow Mitigation](../dex/1.%20AIMM/1.1.%20Pricing/1.1.6.%20Toxic%20Flow%20Mitigation.md), Defense mechanisms

