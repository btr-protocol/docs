# BTR Manifesto: Why We're Building AIMM

> *Adaptive Inventory Market Maker — Bringing Order Book Intelligence to On-Chain Liquidity*

---

> **See also**: [Foundations](/docs/Foundations) — Theoretical lineage and prior art: Avellaneda-Stoikov, Platypus/Wombat ALM, Swaap MMM, Curve v2.

---

## 1. Origin Story: From ALM to AIMM

### 1.1. Where We Started

BTR began as an **advanced Automated Liquidity Manager (ALM)** aimed at solving the key issues observed with existing liquidity management solutions—Arrakis, Gamma, Kamino Liquidity, and similar platforms.

**The core problems with existing ALMs:**
- **Ranges too static**: Most ALMs use simple heuristics without market awareness—no volatility tracking, no momentum signals. Ranges end up either too narrow (exposed to extreme LVR and daily massive swap slippage executed at sub-market rates) or too wide (not yielding at all).
- **No quantitative foundation**: The technical infrastructure was solid, but the "performant market making" claims were never reality. Under the hood, these were reactive, not predictive systems.
- **Hidden losses**: Professional market makers profit; retail ALM users consistently lose when accounting for LVR. The shiny APY numbers correlate inversely with position value decay.

The founding team has been **providing liquidity for years**. We wanted to offer an easy liquidity management solution for users—a UX as simple as the much-beloved Uniswap V2, while yielding more than most self-managed positions. The goal: passive, set-it-and-forget-it liquidity provision that actually makes money.

After months of development on existing DEX stacks (primarily Uniswap V3/V4, Raydium CLMM, Meteora DLMM, and Orca Whirlpools), we came to a realization that changed everything.

### 1.2. The Pivot

Building ALM strategies on top of existing concentrated liquidity protocols was a **waste of resources**. The underlying DEX architecture fundamentally limited what we could achieve:

- **Gas inefficiency**: Every rebalance costs transaction fees
- **Rebalancing costs dominate**: The real killer isn't information leakage—it's market impact and slippage at scale. Rebalancing optimization becomes a trader cost optimization dilemma. We built [BTR Swap](https://github.com/btr-supply/btr-swap) and achieved better rates than competing ALMs 95% of the time. We explored integration of intent-based platforms (1inch Fusion, UniswapX, CoW Swap, Velora) just to make rebalancing viable. The result: 80% of engineering effort went to integration plumbing, only 20% to actual quantitative, value-adding work.
- **Constrained optimization**: We could only optimize *within* the tick-based model
- **Index price requirements**: Off-chain decision making required an extremely reliable, transparent, and fast price aggregator—faster than existing oracles (Pyth, Chainlink). We ended up building [BTR Markets](https://github.com/btr-supply/btr-markets) to track 50+ CEXs and 2000+ liquidity pools, covering ~98% of on- and off-chain volumes (excluding OTC).

A **better DEX design** could enable:
- Dynamic fees computed on-chain by properly estimating volatility and price deviation (inventory risk)—no off-chain infrastructure
- Capital efficiency through unified multi-asset pools: stables, LSTs, and volatiles together, serving different user needs (low risk, yield, exposure) with reduced fragmentation
- Native inventory management eliminating rebalance transactions
- Full decentralization without external dependencies

In a word, we could be **more decentralized**, **more capital efficient**, and have a **much leaner codebase**—all at once.

This is when we pivoted from building an ALM to building **AIMM**.

> **Legacy ALM codebase**: [github.com/btr-supply/alm-contracts](https://github.com/btr-supply/alm-contracts) — Our original ALM work, preserved for historical reference.

---

## 2. The Zeitgeist: Order Books vs AMMs

### 2.1. Our Background

The BTR founding team has **years of experience market-making on order books**—both centralized and decentralized. We're not anti-CLOB activists; we understand and respect order book mechanics deeply.

### 2.2. Order Books Are Mature; AMMs Are Not

Order books are **mature infrastructure**. The dominant narrative—that on-chain CLOBs (dYdX, Hyperliquid, Aster, Lighter) represent DeFi's future—has merit. Professional market makers prefer order books. CEXs use order books. On-chain CLOBs are a natural evolution.

But maturity cuts both ways. Order book mechanics are well-understood—there's less unexplored design space. AMMs, by contrast, remain in their infancy. Concentrated liquidity (2021) was the last major architectural shift. The design space for oracle-aware, inventory-conscious, dynamically-priced AMMs is largely untapped.

**We're not here to replace order books. We're here to explore what AMMs can do that order books cannot.**

### 2.3. Complementary, Not Competing

The debate is often framed as "AMM vs. Order Book"—but this is a false dichotomy. Each architecture has structural advantages:

| Market Type | Order Book Strength | AMM Strength |
|-------------|---------------------|--------------|
| **High liquidity assets** (ETH, BTC) | Good | **Can be better** with proper design |
| **Low liquidity assets** (long tail) | Poor (no market makers) | **Excellent** (passive liquidity) |
| **Stablecoins** | Good on CEXs | **Superior** (hyper-concentration around peg, fully automatable) |
| **Yield-bearing assets** (LSTs) | Requires constant-drift spread pricing, active inventory management | **Formalizable** (drift-aware pricing, automated) |
| **Novel assets** | Non-existent (who will quote?) | **Always quoting**—a price exists regardless of liquidity or volatility |

Order books require active market makers willing to quote. No maker interest = no liquidity. This is why the long tail of crypto assets will always need AMMs.

**Two structural advantages AMMs have over CLOBs:**

1. **N-asset pooling**: Order books pair assets 1:1. AMMs can pool N assets together, sharing liquidity across all pairs. This is impossible in current CLOB designs and enables capital efficiency that scales with asset count.

2. **Dynamic depth curves**: Implementing volatility-responsive liquidity concentration in an order book would require continuous quote updates across the entire book—prohibitive infrastructure and gas costs. AMMs encode this behavior in their pricing function natively. Gyroscope's E-CLP demonstrated concentrated elliptical curves; AIMM extends this with volatility-scaled dispersion that automatically widens or tightens the depth curve based on market conditions.

### 2.4. The Same Tech That Enables CLOBs Makes AMMs Better

The narrative for on-chain order books rests on blockchain improvements:
- **Cheap transactions** → market makers can quote/cancel profitably
- **Fast block times** → real-time price discovery
- **Low latency finality** → competitive with CEX execution

Here's what CLOB proponents miss: **these same advances supercharge AMMs even more**.

| Blockchain Improvement | CLOB Benefit | AMM Benefit |
|------------------------|--------------|-------------|
| **Sub-second blocks** | Near real-time quotes | Near real-time TWAP updates → **minimal price lag** |
| **Cheap gas** | Affordable order placement | Affordable dynamic compute **and storage** → volatility tracking, dynamic fees |
| **Low latency finality** | Competitive execution | Block-scoped MEV window shrinks → **organic MEV protection** |
| **High throughput** | More orders per second | More swaps feeding internal oracle → better price accuracy; affordable [Cooperative Arbitrage](/docs/1.1.6-Toxic-Flow-Mitigation#36-cooperative-arbitrage) to repeg price vs external markets |

**The MEV surface shrinks dramatically on fast chains:**
- Block time drops from 12s → as low as 10ms on the fastest L2s/alt-L1s = **1000× smaller MEV window**
- Arbitrageurs have less time to front-run oracle updates
- Statistical arbitrage becomes harder (price lag approaches zero)
- JIT attacks become unprofitable (must predict sub-second flow)

**Dynamic compute is now viable everywhere:**
- Since early 2025, L1 Ethereum base fees have dropped by more than 90%, making on-chain volatility tracking and dynamic fee computation economically viable—especially when combined with efficient compute and data storage like we implement. Swapping on AIMM costs the same as Uniswap V4 (<$0.50 at median gas levels), while providing smarter pricing and increased capital efficiency.
- On modern L2s (Base, Arbitrum) and alt-L1s (Monad, MegaETH, HyperEVM): basically free.

The irony: **fast chains don't just enable CLOBs—they make AMMs competitive with CLOBs** by eliminating most of their historical weaknesses (lag, static fees, MEV vulnerability). Not all—but enough to compete.

### 2.5. Embedding CLOB Psychology into AMM Design

Order books succeeded because they embed market maker psychology:

- **Adaptive strategies**: Widen spreads during volatility
- **Inventory protection**: Quote asymmetrically based on position
- **Risk pricing**: Charge more for adverse conditions

Traditional AMMs (Uniswap, Curve) lack this sophistication. They're passive price-takers that bleed value to informed traders.

**AIMM embeds CLOB market-making psychology into pool-based architecture:**

| CLOB Market Maker Behavior | AIMM Analog |
|---------------------------|-------------|
| Widen spreads during volatility | Volatility-based fees (vega × σ) |
| Quote asymmetrically based on inventory | Coverage-aware skew pricing (inventory adjusts mid-price) |
| Reduce exposure when overexposed | Directional surcharge on imbalancing flow (momentum/deviation) |
| Adjust aggression based on conditions | Dynamic fees = f(volatility, momentum) |

The result: an AMM that thinks like a market maker but operates permissionlessly.

**Note on terminology**: We use "tri-factor" for the complete pricing model (mid-price calculation from inventory skew + fees from volatility and momentum), but fees themselves are two-factor (volatility + deviation/momentum). The inventory skew adjusts mid-price, not fees directly.

### 2.6. The Permissionless Imperative

Order books require someone to *actively* provide liquidity. This creates centralization risk—market makers can withdraw, coordinate, or be regulated away.

**Permissionless liquidity is DeFi's core innovation.** Any attempt to re-centralize liquidity provision—whether through RFQ systems, solver auctions, or off-chain quote infrastructure—sacrifices what makes DeFi valuable.

AIMM maintains strict permissionlessness:
- **Privileged arbitrageurs (Cooperators)** exist, but the model is fair: any stat-arb trader can enlist anonymously to compete. See [Cooperative Arbitrage](/docs/1.1.6-Toxic-Flow-Mitigation#36-cooperative-arbitrage).
- **No off-chain components** for pricing decisions
- **Internal multi-timeframe TWAP and volatility** by default—no external oracle dependency
- **Anyone can provide liquidity** without approval

Most AMM design research in 2025 is trending toward centralization: oracle-dictated pricing, whitelisted market makers, MEV auctions. AIMM solves the same problems in a fully decentralized manner.

This differentiates AIMM from modern "hybrid" approaches. See [Foundations § Permissionless Design](/docs/Foundations#15-permissionless-design-philosophy) for detailed comparison with RFQ/intent protocols.

---

## 3. The Problem: AMMs Are Stuck in 2021

Decentralized exchanges processed [$1.3 trillion in spot volume in Q3 2025](https://coinlaw.io/decentralized-exchanges-dex-statistics/), with perpetual derivatives hitting [$1.8 trillion](https://themarketperiodical.com/2025/09/25/spot-and-perps-dexs-see-surge-in-volume-amidst-rise-of-hyperliquid-and-aster/)—an 87% quarterly surge. Yet despite this explosive growth, the fundamental architecture powering these markets remains largely unchanged since concentrated liquidity arrived in 2021: whether the concentrated-fee CFMM style (Curve) or tick-based CLMM style (Uniswap V3).

[Uniswap commands 35-50% of spot DEX volume](https://blockchain.news/news/uniswap-leads-dex-market-august-2025) with $4.5 billion TVL. Every major DEX—Raydium CLMM, LFJ's Liquidity Book, Meteora DLMM, Orca's Whirlpools—runs variations of the same concentrated liquidity model.

Uniswap v4's hooks (processing [$100B+ cumulative volume across 2,500+ custom pools](https://www.dwf-labs.com/research/457-what-s-new-in-uniswap-v4-three-key-changes-and-two-new-protocols) since January 2025) lay foundations for more advanced AMMs: volatility tracking, dynamic fees, inventory awareness. But Uniswap V4 didn't invent hooks—Balancer V3 and Algebra Integral (calling them "Plugins") had them first. And critically, **Uniswap V4 is an infrastructure layer, not a full-fledged AMM protocol**: it ships no default hooks that consolidate liquidity or protect LPs. The fragmentation problem persists.

**The status quo has three fundamental flaws:**

### 3.1. Naive Price Discovery

Current AMMs derive prices directly from reserves through rigid invariant formulas:

| AMM | Invariant | Shape Parameters |
|-----|-----------|------------------|
| **Uniswap v2/v3** | x·y = k | Tick spacing only |
| **Curve v1** | An^n∑xᵢ + D = DAn^n + D^(n+1)/(n^n∏xᵢ) | [Amplification coefficient A](https://docs.curve.finance/cryptoswap-exchange/in-depth/) |
| **Curve v2** | Complex with K₀, K | [A + gamma (γ)](https://docs.curve.finance/cryptoswap-exchange/in-depth/) |
| **Gyroscope E-CLP** | Elliptical curve | [α, β bounds; c, s rotation; λ stretch](https://docs.gyro.finance/pools/e-clps) |

These parameters are **static**. Curve's A coefficient or Gyroscope's λ stretch factor don't respond to market volatility, price velocity, or inventory imbalance. They mechanically map reserves → price regardless of market conditions.

The result? AMMs price assets with the sophistication of a 1990s vending machine.

### 3.2. Fragmented Capital

The explosion of isolated pairs, fee tiers, and hooks has shattered liquidity across thousands of pools. A single token might have:
- 5+ fee tiers on Uniswap (0.01%, 0.05%, 0.3%, 1%, custom)
- Separate pools on each L2/rollup
- Fragmented TVL across hook-enabled variations
- Multiple competing ranges within each pool

Capital efficiency plummets. Traders face worse execution. LPs spread thin across competing pools.

### 3.3. The LVR Tax — The Hidden Drain on LP Capital

**Loss-Versus-Rebalancing (LVR)** is the hidden tax that costs LPs [5-7% of their liquidity annually](https://docs.cow.fi/cow-amm/concepts/the-problem-of-lvr)—hundreds of millions in aggregate. Research by [Milionis et al.](https://arxiv.org/pdf/2208.06046) established that AMMs systematically trade at worse-than-market prices, creating guaranteed arbitrage profits.

> *"Fees do not sufficiently compensate for arbitrage losses in most of the largest Uniswap liquidity pools—historically, returns from fees have been smaller than losses to arbitrageurs."* — [Measuring Arbitrage Losses and Profitability of AMM Liquidity (2024)](https://arxiv.org/html/2404.05803v1)

LVR accounts for more value extraction than frontrunning and sandwich attacks combined. Yet every major AMM—including Uniswap v4's hook ecosystem—remains fundamentally vulnerable because they share the same architectural flaw: **constant fees and stale reserves allow one-sided extraction in most market conditions**—whether the pool uses market price directly from reserves (CLMMs) or TWAP/VWAP derived (like Curve's EMA).

**Current solutions are insufficient.** Uniswap v4 and Balancer V3 hooks enable [LVR minimization experiments](https://github.com/sameshl/uniswap-v4-hooks-exploration), [volatility-based fee hooks](https://github.com/fewwwww/awesome-uniswap-hooks), and [impermanent loss hedging hooks](https://github.com/johnsonstephan/awesome-uniswap-v4-hooks). Hooks can override the entire pricing function—escaping x·y=k is technically possible. But doing so is anti-pattern: you're carrying the dead-weight of legacy architecture. The tick-size constraint exemplifies this: you want smaller tick sizes for concentrated stable pools, but gas consumption scales with ticks crossed, and tick spacing can't be changed after pool initialization.

We consider this design too heavy and inefficient. Better to build a lighter, more efficient AMM from scratch that natively supports volatility tracking, dynamic fees, inventory-aware pricing, multi-asset pools, and cooperative arbitrage—without the legacy baggage.

That said, hooks remain elegantly extensible. It's not excluded that the DAO deploys Uni V4 pools with hooks as proxies to BTR's Core instance.

---

## 4. The Evolution: From Strict Invariants to Inventory-First Pricing

We're not dropping invariants entirely—we're loosening the strict relationship between reserves and price. This relationship is now baked into the inventory skew, altered by the asset's liquidity profile, with market-aware spread (volatility, momentum) around the inventory-dictated price.

**How AIMM pricing works:**
1. **Reference price** = multi-timeframe TWAP of swap prices (internal oracle)
2. **Mid-price** = adjusted based on inventory skew (inventory risk management) along pair-specific liquidity profiles with volatility-scaled dispersion. See [Anchor-Tree Pricing](/docs/1.1.3-Anchor-Path-Pricing).
3. **Spread (fees)** = dynamic around mid-price, based on volatility and momentum (price deviation from TWAP)

All these components work together to provide better pricing, capital efficiency, and risk management for LPs and traders alike.

### 4.1. The ALM Foundation

The first attempt to break from invariant-based pricing came from **Platypus Finance** and later **Wombat Exchange**, which pioneered [Asset-Liability Management (ALM)](https://medium.com/platypus-finance/platypuss-asset-liability-management-eli5-92a1ee85b17) for AMMs.

Their innovation: **Coverage Ratio** = Assets / Liabilities

Instead of deriving price from reserve ratios, they track what the pool *owes* LPs (liabilities) separately from what it *holds* (assets). This enables single-sided deposits, natural supply/demand growth per token, and explicit tracking of pool health.

Both Platypus and Wombat demonstrated the viability of the ALM model for stableswap at scale, and started working on volatile extensions. **But they didn't go far enough.** They still use coverage ratio merely as a slippage modifier—the pricing mechanism remains fundamentally constrained by a modified, strict invariant bonding all pooled assets together.

> For a deeper analysis of ALM mechanics and how AIMM extends them, see [Foundations § Asset-Liability Management](/docs/Foundations#3-asset-liability-management-platypus--wombat).

---

## 5. Our Solution: AIMM

**AIMM (Adaptive Inventory Market Maker)** extends the liability model with four key innovations:

### 5.1. TWAP-Centered Spline Pricing

We **decouple price from reserves**, using oracle-derived reference prices instead.

Instead of invariant formulas, AIMM uses:

1. **Internal TWAPs** — Dual-window (5min fast, 1hr slow) exponential moving averages updated on every swap
2. **Multi-timeframe Volatility EMAs** — Real-time σ tracking for dynamic spread adjustment
3. **Customizable Spline Profiles** — Monotone cubic Hermite interpolation defining liquidity depth at any price point

By default, we use the internal oracle for TWAP. But by design, we support external oracles with fallback. A pool can therefore be:
- **Fully permissionless and autonomous** — relying only on internal oracle
- **Oracle-integrated** — for institutional market makers wanting to run advanced strategies with their own price feeds, or RWA issuers pegging illiquid tokens to off-chain assets with minimal slippage

**Liquidity profiles are infinitely flexible:**
- Concentrate liquidity around expected price ranges
- Create asymmetric depth for directional markets
- Update profiles based on multi-timeframe price density analysis

```
Traditional AMM:     reserves → invariant → price
AIMM:               oracle + spline + inventory → price
```

> Spline-based liquidity profiles appear to be [novel in DeFi AMM design](/docs/Foundations#11-spline-based-liquidity-profiles). For comparison with elliptical (Gyroscope) and parametric curves, see Foundations.

### 5.2. Multi-Asset Unified Pools

AIMM pools contain **any number of tokens** routed through an anchor tree topology.

**Important distinction**: Swaps don't all path through a single hub token. The first **common anchor** is used for pricing, and swaps are accounted in the pool's "base token" (numéraire). Two swaps might not share any anchor—but all anchors connect to the tree root (base token), enabling unified accounting and risk management.

**Why this matters:**
- **O(1) scaling**: Unlike pairwise designs (Curve v2 CryptoSwap, Wombat) where N-asset pools require N² compute for oracle upkeep, pool configuration, and invariant solving, our anchor-tree design scales linearly. This matches the compute efficiency of CCMMs (aka Orbital AMM)—the only two AMM designs reasonably scalable beyond 10-asset pools.
- **Triangulated quoting**: All assets quote vs anchors, not directly against each other. This reduces mispricing arbitrage surface and provides robustness against market fragmentation and price manipulation.
- **Capital consolidation**: One pool depth serves all pairs, eliminating fragmentation.

The anchor tree uses Least Common Ancestor (LCA) pathfinding—mathematically optimal routing with minimal on-chain computation. This mirrors [traditional FX market structure](/docs/Foundations#9-anchor-path-pricing) where cross-rates triangulate through major currency pairs (typically USD).

### 5.3. LVR-Aware Pricing

Here's where AIMM fundamentally diverges: we don't claim to eliminate LVR—**we're the first AMM to price it explicitly** rather than let LPs bleed silently.

**Inventory Skew Adjusts Mid-Price ([Avellaneda-Stoikov Framework](/docs/Foundations#2-inventory-based-market-making-the-avellaneda-stoikov-framework))**

When coverage ratio deviates from target:
- Pool accumulating token → quote tighter asks, wider bids
- Pool depleting token → quote tighter bids, wider asks

This is how every professional market maker in traditional finance operates. AIMM brings that intelligence on-chain. NB: inventory skew adjusts **mid-price**, not fees directly.

**Two-Factor Dynamic Fees**

Fees (spread around mid-price) respond to two independent signals:

| Factor | Formula | Purpose |
|--------|---------|---------|
| **Volatility** | ν × σ | Adverse selection premium |
| **Momentum** (deviation) | λ × Δ | Directional surcharge for toxic flow |

High volatility + high deviation = maximum spread. Stable conditions = tight execution.

(The third factor—inventory—affects mid-price positioning, making the complete pricing model "tri-factor." But fees themselves are two-factor.)

**The Result: Toxic Flow Gets Priced**

By applying directional fees based on volatility and price momentum, AIMM:
- Widens spreads when flow is likely toxic (high deviation, depleting inventory)
- Tightens spreads when flow is likely organic (improving coverage)
- Internalizes adverse selection costs that LPs would otherwise absorb

The pool behaves less like a "free option seller" and more like an informed counterparty that charges appropriately for risk.

### 5.4. Cooperative Arbitrage

The remaining challenge with reactive AMM pricing: **lagged prices** relative to external markets. Traditional solutions add centralization:
- High-frequency oracles (single point of failure)
- am-AMM style auctions (single manager, complex, and centralized)
- RFQ systems (whitelisted solvers, off-chain matching)

AIMM solves this through **Cooperative Arbitrage**—a whitelisted, reputation-based rebate program:

1. **Anonymous application**: Any stat-arb trader can apply to become a Cooperator at [btr.supply/coop](https://btr.supply/coop)
2. **All cooperators start equal** with the same rebate tier (e.g., 20%)
3. **Reputation = donations / rebates**: Higher reputation → higher rebate percentage (up to 80% max)
4. **Competition among cooperators** keeps prices tight and inventory balanced
5. **Low reputation (<0.9) → revocation**: Only aligned arbitrageurs remain

Cooperators are **arbitrageurs**—encompassing MEV searchers, HFT desks, and medium-frequency stat arb traders. There's overlap between MEV and statistical arbitrage; we don't distinguish.

**Why it works**:
- Rebates lower cooperators' profit threshold → they front-run unknown arbitrageurs → smaller LVR window
- High-reputation cooperators donate proceeds back → LPs recover LVR losses (internalization of LVR)
- Cooperators donate as much as they want while remaining profitable, but compete to top the reputation leaderboard for more rebates → virtuous cycle incentivizing maximum donations relative to rebates claimed
- DAO can run its own bot with 100% donation rate → maximal reputation → closes LVR loop
- Whitelisted but competitive—up to 50 cooperators at launch (governance-adjustable) race to rebalance
- No oracle dependency, no builder dependency, no manager trust risk

**The Emergent Advantage**: Any cooperator can compete on equal terms, but those who donate more earn better rebates. A DAO-run bot that donates 100% of arbitrage proceeds naturally achieves top reputation and front-runs adversarial extractors—returning all LVR to LPs.

The result: LVR mitigation comparable to oracle-based AMMs (UAMM, DODO, Swaap) and manager auctions (am-AMM), without oracle fragility or manager centralization.

We call these participants **"Cooperators"** or **"Cooperative Arbitrageurs"**.

See: [Cooperative Arbitrage](/docs/1.1.6-Toxic-Flow-Mitigation#36-cooperative-arbitrage)

---

## 6. Why Now?

### 6.1. The Hooks Experiment Proves Appetite

Uniswap v4's 2,500+ custom pools in 10 months prove there's massive appetite for AMM innovation. Developers are building LVR minimization, volatility fees, MEV redistribution—**all trying to solve problems that AIMM addresses architecturally**.

Hooks can technically skip tick traversal entirely, but it's anti-pattern—you're carrying the dead-weight of V3 architecture. And if integrators still need hooks, we've added minimal hook support for Uniswap V4 compatibility.

AIMM is what hooks *tried* to do but couldn't cleanly: a clean-slate architecture built for informed market making from the ground up.

### 6.2. The LP Experience Is Broken

Studies consistently show that **90%+ of Uniswap v3/v4 LPs lose money** when accounting for LVR. The concentrated liquidity model shifts complexity onto users who:
- Must actively manage positions (gas costs to move ranges)
- Suffer LVR passively from stale pricing—arbitrageurs extract value on every price move, even if prices revert
- Compete against sophisticated market makers
- Face [JIT liquidity attacks](/docs/Foundations#8-clmmdlmm-limitations) that dilute their fee earnings

There's a nostalgia for Uniswap V2 among early LPs. They were subject to impermanent loss, but it was far less brutal than current LVR under hyper-concentrated conditions—which effectively leverages the loss for uncertain reward, since LPs are more competitive than ever.

**Our design is opinionated:** Most liquidity providers don't want to run market-making strategies. They want to deposit capital and earn yield. AIMM optimizes liquidity profiles automatically—single range, professionally managed—which is net positive for the vast majority of LPs.

Beyond retail, our design also addresses institutions. Professional market makers aren't all hyper-profitable. A fair share of whales providing to ALMs like Kamino, Beefy CLM, Gamma, and Arrakis are unaware of their net loss—these applications display only short-term yield APRs without netting the LVR losses. It's a widespread blind spot that LPs fall for until it's too late and they realize the shiny APY correlates inversely with their position value.

### 6.3. DEXs Must Rival CEX Spot Markets

We want AIMM to be **the spot DEX for everything**: stables, yield-bearing assets, majors, alts, and all the existing markets arriving on-chain (FX, commodities, stocks, indices). Our design caters to all these asset classes and exhibits unique traits complementary with traditional order book exchanges.

For DeFi to capture meaningful spot volume from centralized exchanges, on-chain markets need:
- Tighter spreads on liquid pairs
- Transparent, predictable pricing
- Sustainable LP economics

AIMM delivers all three through structural design rather than governance bribes or unsustainable incentives.

---

## 7. Technical Architecture

### 7.1. Pricing Pipeline

```
Input: (tokenIn, tokenOut, amountIn)
       ↓
1. Calculate inventory skew from coverage ratio
       ↓
2. Get TWAP from internal/external oracle
       ↓
3. Calculate volatility σ = mean(fastVolEMA, slowVolEMA)
       ↓
4. Traverse spline by volume → execution price
       ↓
5. Apply fees (volatility + momentum) with protocol split
       ↓
Output: (amountOut, spreadBps, fees)
```

### 7.2. Coverage-Aware ALM

Building on Wombat's foundation, AIMM implements:

- **Coverage Ratio**: reserves/liabilities per token
- **Withdrawal Haircuts**: Quadratic penalty when coverage < 100%
- **Liability Decay**: Time-decay mechanism to restore coverage during prolonged undercollateralization
- **Critical Thresholds**: Circuit breakers at extreme coverage levels

### 7.3. Oracle Security

**Internal Oracle:**
- Dual-window TWAPs resist single-block manipulation
- Same-block update rejection prevents flash loan attacks
- Volatility EMAs detect abnormal price movement

**External Fallback (Optional):**
- Multi-source aggregation (Chainlink, Pyth, custom feeds)
- Outlier detection with configurable deviation bounds
- Circuit breakers at X% deviation from inventory-implied price

For low-volume pools, external oracle fallback prevents TWAP manipulation across block boundaries.

### 7.4. Extreme Volatility Handling

**What happens in a flash crash?**

1. **Volatility fees spike** — σ-based component widens spreads automatically
2. **Inventory skew caps at ±100** — bounded price adjustment prevents runaway
3. **Maximum slippage = liquidity profile edge** — dispersion-scaled worst-case execution
4. **Emergency pause available** — governance can freeze operations if needed

The pool becomes a price-taker at extremes, but with explicit bounds rather than unbounded loss.

### 7.5. Emergency Mechanics

When coverage drops below critical threshold (default 50%):

1. **Withdrawal haircuts scale quadratically** — early exiters pay less, protecting remaining LPs
2. **Liability decay activates** — gradual reduction of claims to restore solvency
3. **Maximum premium applies** — +100 skew discourages further withdrawals
4. **Manual pause available** — governance can halt operations

These mechanics prevent bank runs while giving LPs time to respond.

### 7.6. Gas Efficiency

AIMM is designed to be **gas-competitive with Uniswap v4**:

| Operation | AIMM | Uniswap v4 | Curve v2 |
|-----------|------|------------|----------|
| Spline traversal | ~3-5 knots | 10-50+ ticks | N/A |
| Price computation | Direct | Tick iteration | Newton iteration (~35k gas) |
| Storage pattern | EIP-1153 transient | EIP-1153 transient | Traditional SSTORE |

---

## 8. Governance & Curator Trust

### 8.1. The Spline Governance Challenge

Customizable liquidity profiles are powerful but create a trust question: who sets them, and how do LPs verify they're not being front-run?

**AIMM's Approach:**

1. **Time-locked parameter changes** — Profile updates have mandatory delay (hours), giving LPs time to exit if needed
2. **Algorithmic profile fitting** — Keeper-driven updates based on on-chain trade density analysis
3. **Transparent on-chain state** — All profile parameters publicly readable and auditable
4. **Per-pool curator model** — Pool deployers set initial profiles; ongoing management per pool governance

We're replacing strict invariant risk with bounded human discretion risk, with full transparency and exit rights.

### 8.2. White-Label Architecture

AIMM pools are **permissionlessly deployable**. Any protocol, asset manager, or market maker can:

- Deploy their own N-asset pool
- Set custom parameters and liquidity profiles
- Apply their own hooks and rules
- Manage and incentivize as they see fit

No need to fork the protocol. Don't like our parametrization? Deploy your own pool under your own rules.

---

## 9. Incentive Design

### 9.1. Beyond ve(3,3)

Aerodrome's success ($572M TVL, $16.5B monthly volume) proves that incentive design matters as much as AMM mechanics. But ve(3,3) has tradeoffs:
- LPs give up yield to receive emissions (aggressive dilution)
- Vote-direction creates governance complexity
- Bribe markets add friction

**AIMM's Alternative:**

| Feature | Aerodrome (ve3,3) | Curve Gauges | AIMM |
|---------|-------------------|--------------|------|
| LP yield to receive emissions | Must lock | Must lock | No lock required |
| Governance participation | Requires veToken | Requires veCRV | LP voting rights included |
| Emission boost | veToken staking | veCRV staking | sBTR ↔ sLP matching |
| Max supply | Inflationary | Inflationary | Fixed cap with buybacks |

**Key innovations:**
- LPs participate in governance without staking governance token
- Staking LP positions (sLP) paired with staked governance token (sBTR) maximizes rewards
- Fixed token supply with value accrual through protocol fee buybacks
- No gauge bribes needed—structural capital pooling instead

### 9.2. Trader Incentives (Exploring)

We're exploring dedicating ~20% of token emissions to traders who contribute to organic rebalancing (not toxic flow). Rewards for improving pool coverage create aligned incentives across all participants.

---

## 10. Target Market: Stablecoins First

### 10.1. Why Stables?

1. **Curve dominates but has 30bps+ spreads on smaller pairs** — room for improvement
2. **Wombat proved ALM works here** — coverage-ratio mechanics validated at scale
3. **Clean signal** — LVR is highest on volatile pairs; proving our model on stables demonstrates fundamental viability
4. **Clear benchmark** — "tightest spreads in DeFi" is measurable and marketable

### 10.2. Initial Launch

Launch pools: **USDC / USDT / USDS / USDe / USD1 / PYUSD / RLUSD / ...**

Once we own stables, expansion to ETH, BTC, and blue-chip pairs. Trying to be everything at launch dilutes the story.

### 10.3. Target Users

**Primary:** Sophisticated DeFi-native LPs (the "prosumer" market that experiments with new protocols)

**Secondary:** Protocols seeking white-label liquidity infrastructure

---

## 11. Deployment Plan

### 11.1. Target Chains

Most active and liquid EVMs:
- **Ethereum** — deepest liquidity, institutional presence
- **Base** — Coinbase ecosystem, growing rapidly
- **Arbitrum** — established DeFi hub
- **BNB Chain** — high retail volume

### 11.2. Security

- **Multiple third-party audits** — Pre-launch security review by professional auditors
- **Bug bounty program** — Immunefi or equivalent
- **Gradual TVL caps** — Staged rollout with limits

Safety is non-negotiable. We won't launch without comprehensive security coverage.

### 11.3. Composability Option

A Uniswap v4 hook can turn AIMM into a v4 back-end, providing:
- Instant composability with v4 ecosystem
- Proven security model (Uniswap audits)
- Direct benchmark: "X% more capital-efficient than vanilla v4"

This gives us optionality: standalone launch or v4 integration based on market reception.

---

## 12. Competitive Moat

### 12.1. Why Can't Uniswap Just Add This?

**The tick model is structurally incompatible.**

Uniswap's architecture requires:
- Discrete tick traversal for price discovery
- Per-position liquidity accounting
- LP-managed range selection

AIMM requires:
- Oracle-first pricing with spline depth
- Pool-level inventory accounting
- Curator-managed profiles

These are fundamentally different designs. Uniswap could build AIMM, but it would be a new protocol—not a v5 upgrade.

### 12.2. Unforkable Advantages

1. **Network effects in multi-asset pools** — More assets = deeper liquidity = better execution = more assets
2. **Operational expertise** — Profile optimization requires market-making knowledge, not just code
3. **Integrated incentive design** — Token model aligned with LP success, not governance extraction

---

## 13. The BTR Vision

We named ourselves **BTR (Bayesian True Range)** because our early research used Bayesian inference for price estimation. The philosophy persists: **we treat every swap as evidence updating our belief about fair price**.

- **Bayesian**: Beliefs (prices) update based on evidence (flow, volatility, coverage)
- **True Range**: We capture realistic dispersion of price outcomes, not idealized curves
- **Adaptive**: Every parameter responds to market conditions

The goal isn't to eliminate adverse selection entirely—that's impossible without eliminating trading. The goal is to **price it correctly** so LPs are compensated fairly and arbitrageurs can't extract risk-free profits.

Traditional AMMs treat every trade identically. AIMM treats every trade based on what it reveals about market conditions.

---

## 14. Summary

| Problem | Traditional AMMs | AIMM Solution |
|---------|-----------------|---------------|
| **Pricing** | Static invariant formulas | TWAP-centered + spline + inventory-aware |
| **Capital** | Fragmented across pairs/tiers | Unified multi-asset pools with O(1) scaling |
| **LP Risk** | 5-7% annual LVR (unpriced) | Explicit adverse selection fees + cooperative arbitrage |
| **Flexibility** | Fixed curve shapes | Customizable liquidity profiles per asset |
| **Intelligence** | Passive price taker | Dynamic fees (volatility + momentum) |

We're not building a better Uniswap. We're building what comes after the invariant era.

---

*BTR Protocol — Adaptive Inventory Market Making*

