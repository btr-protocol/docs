# Intellectual & Technical Foundations

> The theoretical lineage and design choices behind AIMM

---

## 1. Introduction

AIMM (Adaptive Inventory Market Maker) synthesizes ideas from quantitative market making theory, ALM-based stableswap design, and concentrated liquidity research. This document traces the intellectual genealogy of each component, acknowledges prior art, and explains our design decisions including paths not taken.

For AIMM's vision and value proposition, see [Manifesto](/docs/manifesto).

---

## 2. Inventory-Based Market Making: The Avellaneda-Stoikov Framework

### 2.1. Origins

The foundation of AIMM's pricing philosophy traces to [Marco Avellaneda and Sasha Stoikov's 2008 paper](https://people.orie.cornell.edu/sfs33/LimitOrderBook.pdf) "High-Frequency Trading in a Limit Order Book." This work formalized how a rational market maker should quote asymmetrically based on inventory risk.

The key insight: a market maker holding excess inventory faces directional exposure. Rather than quoting symmetrically around mid-price, the optimal strategy adjusts quotes to encourage trades that reduce inventory.

### 2.2. The Reservation Price

Avellaneda-Stoikov introduces the **reservation price**:

$$r = s - q * gamma * sigma^2 * tau$$

where:
- $r$ = reservation price
- $s$ = mid-market price
- $q$ = inventory quantity (positive = long, negative = short)
- $gamma$ = risk aversion parameter
- $sigma$ = price volatility
- $tau$ = time remaining (T - t)

When inventory $q > 0$ (long position), the reservation price shifts below market—the market maker is willing to sell at lower prices to reduce exposure. The converse applies for short positions.

### 2.3. Optimal Spread

The optimal bid-ask spread depends on volatility, order arrival intensity, and risk aversion:

$$Delta = gamma * sigma^2 * tau + {2}/{gamma} * ln(1 + gamma/k)$$

where:
- $Delta$ = optimal bid-ask spread
- $tau$ = time remaining (T - t)
- $k$ = order arrival intensity parameter

Higher volatility or risk aversion widens the spread; more frequent order flow tightens it.

### 2.4. AIMM's Adaptation

AIMM translates these continuous-time concepts to discrete blockchain execution:

| Avellaneda-Stoikov | AIMM Analog |
|--------------------|-------------|
| Inventory $q$ | Coverage ratio deviation from target |
| Volatility $sigma$ | Oracle `fastVolEMA` + `slowVolEMA` |
| Risk aversion $gamma$ | Per-asset `vega` and `lambda` parameters |
| Order intensity $k$ | Implicit in spline depth calibration |

The pricing model implements inventory-aware market making:
- **Volatility band** $S_v$ corresponds to the spread component
- **Directional surcharge** $U$ implements asymmetric quoting based on coverage impact
- Coverage-improving trades face only symmetric spread; coverage-worsening trades incur surcharge

See [Spread & Fees](/docs/1.1.4-Spread-&-Fees) for implementation details.

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

See [Inventory Management](/docs/1.1.1-Inventory-Management) for coverage mechanics.

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

$$prod_i x_i^{w_i} = k$$

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
| **Price source** | External oracles | Internal TWAP + external fallback |
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

See [Oracles](/docs/3.5-Oracles) for dual-window TWAP design.

---

## 6. Concentrated Liquidity: Gyroscope E-CLP

### 6.1. Elliptical Concentration

[Gyroscope's E-CLP](https://docs.gyro.finance/pools/e-clps) (Elliptic Concentrated Liquidity Pool) uses elliptical curves for asymmetric liquidity concentration:

> "Elliptic CLPs allow trading along the curve of an ellipse. Similar to other CLPs, E-CLPs are designed to concentrate liquidity within price bounds."

The ellipse is formed by transforming a circle:
- **Stretch** ($lambda$): Elongates the curve
- **Rotation** ($phi$): Tilts the concentration
- **Displacement** ($alpha$, $beta$): Shifts price bounds

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
| **Arbitrary shapes** | No | Yes (Catmull-Rom) |

See [Liquidity Shaping](/docs/1.1.2-Liquidity-Shaping) for spline mechanics.

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

$$S_v = 100 + (sigma_p * nu)/(100M)$$

where:
- $S_v$ = volatility band (basis points)
- $sigma_p$ = pair volatility
- $nu$ = vega sensitivity
- $M$ = multiplier base (10000)

Key differences:

| Aspect | LFJ v2 | AIMM |
|--------|--------|------|
| **Volatility measurement** | Bin crossings (VA) | Oracle EMA ($sigma$) |
| **Fee components** | Base + variable | Base + volatility + inventory |
| **Inventory awareness** | No | Yes (directional surcharge) |
| **Multi-hop aggregation** | Per-hop | Path-level max |

---

## 8. CLMM/DLMM Limitations

### 8.1. The Concentrated Liquidity Tradeoff

Uniswap v3's concentrated liquidity provides capital efficiency but introduces significant challenges:

**Amplified Impermanent Loss**: Narrower ranges magnify IL when prices move out of range:

> "Concentrated positions can experience more significant impermanent loss than V2 positions if prices move beyond your range."   [Kaiko Research](https://research.kaiko.com/insights/jit-2)

**Active Management Burden**: LPs must continuously rebalance:

> "LPs, especially smaller LPs, are constrained in the degree to which they can actively manage their concentrated liquidity positions. As a result, their positions tend to drift out of range, suffering from IL and failing to capture any trading fees."

**Net Losses for Most LPs**: Research indicates majority of LPs underperform holding:

> "Between V3's launch and September 20th, analyzed pools saw over $100B in trading volume, earning LPs approximately $200M in fees. However, LPs lost more than $260M to impermanent loss, resulting in a net loss of over $60M."   [Rekt News](https://rekt.news/uniswap-v3-lp-rekt)

### 8.2. JIT Liquidity Attacks

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

| Problem | CLMM/DLMM | AIMM |
|---------|-----------|------|
| **Active management** | Required | Curator-managed profiles |
| **IL amplification** | Severe in narrow ranges | Coverage-based (reserves change, not curve) |
| **JIT vulnerability** | High | Reduced (no discrete tick crossings) |
| **LP sophistication** | Required | Not required |

See [Toxic Flow Mitigation](/docs/1.1.6-Toxic-Flow-Mitigation) for detailed defenses.

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
- A�B trades route through A�USD�B
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
- ETH � BTC routes as ETH � USDC � BTC
- WSTETH � DAI routes as WSTETH � ETH � USDC � DAI

### 9.3. Precedents

This structure mirrors traditional finance:
- **FX markets**: Cross-rates derived from USD pairs
- **Equity markets**: Market makers quote against cash
- **Crypto exchanges**: BTC or USDT as base pairs

The innovation is on-chain implementation with **Least Common Ancestor (LCA) pathfinding** mathematically optimal routing with O(log N) computation.

### 9.4. Benefits

| Aspect | All-Pairs | Anchor Tree |
|--------|-----------|-------------|
| **Pair count** | O(N�) | O(N) |
| **Capital efficiency** | Diluted | Concentrated at anchor |
| **Gas cost** | Per-pair depth | Single traversal |
| **Adding tokens** | N new pairs | 1 new edge |

See [Anchor Path Pricing](/docs/1.1.3-Anchor-Path-Pricing) for implementation.

---

## 10. Circular/Orbital Market Makers

### 10.1. Polar Coordinate AMMs

Recent research explores AMMs using polar coordinates. [Paradigm's Orbital](https://www.paradigm.xyz/2025/06/orbital) and academic work on [Concentrated Circular Market Makers (CCMM)](https://arxiv.org/abs/2510.05428) represent this direction:

> "This research expands on n-dimensional automated market makers for stablecoins by showing a way to build concentrated liquidity positions with ticks in polar coordinates."

### 10.2. The Torus Model

Orbital extends to N dimensions using toroidal geometry:

> "Logically speaking, we are now dealing with one spherical AMM and one circular AMM. By rotating the sphere around the circle, we obtain a single torus, or donut shape."

This enables N-asset stablecoin pools with concentrated liquidity.

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
- Arbitrary liquidity profiles via Catmull-Rom interpolation
- Direct price/depth correspondence
- Easier visualization and debugging

For N-asset stablecoins specifically, Orbital may be superior. For general multi-asset pools with varying volatility, AIMM's anchor tree + spline approach provides more flexibility.

---

## 11. Spline-Based Liquidity Profiles

### 11.1. Novel Contribution

AIMM's spline-based liquidity profiles appear to be **novel** in DeFi AMM design. Extensive literature search found no prior implementations of Catmull-Rom or similar interpolating splines for AMM pricing functions.

### 11.2. Mathematical Foundation

[Catmull-Rom splines](https://en.wikipedia.org/wiki/Cubic_Hermite_spline#Catmull%E2%80%93Rom_spline) (1974) are widely used in computer graphics for smooth curve interpolation:

> "Catmull-Rom is an interpolating cubic spline with built-in C1 continuity."

Properties relevant to AMM design:
- **Passes through control points**: LP-intuitive (price�depth mapping exact at knots)
- **C1 continuous**: No discontinuous jumps in liquidity
- **Local control**: Moving one knot doesn't affect distant regions
- **Centripetal variant**: Avoids self-intersections and cusps

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

Gas cost: O(log k) where k = number of knots (typically 3-7).

See [Liquidity Shaping](/docs/1.1.2-Liquidity-Shaping) for spline mathematics.

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
5. **Anchor tree + LCA pathfinding**: Efficient multi-asset routing without N� pairs

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
4. **Cooperative Arbitrage** to internalize stat arb without oracle dependency

Pool prices remain **reactive** (not pro-active), but:
- Inventory skew discourages adverse selection
- Cooperative Arbitrage accelerates price convergence via whitelisted cooperators
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
| **Cooperative Arbitrage** | ⚠️ Whitelisted | DAO-curated cooperators, competitive rebates |
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
- ⚠️ Cooperative Arbitrage is whitelisted (DAO-curated)
- ✅ Trading remains fully permissionless
- ✅ No off-chain dependencies for pricing
- ✅ Optional (not required) external oracles

### 15.4. The Cooperative Arbitrage Model

Cooperative Arbitrage balances permissionless trading with curated arbitrageur access:

1. **Trading is fully permissionless**: Anyone can swap, no restrictions
2. **Cooperator whitelist**: Arbitrageurs apply to DAO at [btr.supply/coop](https://btr.supply/coop)
3. **Reputation-based competition**: All cooperators start equal (e.g., 20% rebate), earn higher tiers via donations
4. **Not MEV, but stat arb**: Targets cross-exchange (CEX-DEX) statistical arbitrage, not block ordering
5. **Rebate distribution**: `reputation = donations / rebates`; higher reputation → higher rebates (up to 80%)
6. **Revocation**: Low reputation (<0.9) → loss of Cooperator status

**Key distinction**: Cooperators use the same swap interface as everyone—they don't get execution privileges or off-chain access. The whitelist controls rebate eligibility, not trading access.

**Why whitelist?**:
- **Quality control**: Ensure cooperators don't expose LP deposits to excessive CEX risk
- **Inventory management**: Trusted actors carry bounded inventory risk
- **Active monitoring**: Track donations, reputation, and behavior
- **Competitive alignment**: Only high-performing cooperators remain

This differs from solver/filler whitelists (CoW, UniswapX) where non-whitelisted users cannot access certain execution paths. In AIMM, everyone trades on equal terms; cooperators just earn rebates for improving pool state.

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
- Avellaneda & Stoikov (2008): [High-Frequency Trading in a Limit Order Book](https://people.orie.cornell.edu/sfs33/LimitOrderBook.pdf)
- Gu�ant, Lehalle & Fernandez-Tapia (2012): [Optimal Portfolio Liquidation with Limit Orders](https://arxiv.org/abs/1206.4810)
- Catmull & Rom (1974): A Class of Local Interpolating Splines

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

- [AIMM Architecture](/docs/overview-aimm) — System architecture
- [Manifesto](/docs/manifesto) — Vision and value proposition
- [Spread & Fees](/docs/1.1.4-Spread-&-Fees) — Bi-factor fee model
- [Oracles](/docs/3.5-Oracles) — Dual-window TWAP design
- [Liquidity Shaping](/docs/1.1.2-Liquidity-Shaping) — Spline profiles
- [Toxic Flow Mitigation](/docs/1.1.6-Toxic-Flow-Mitigation) — Defense mechanisms

