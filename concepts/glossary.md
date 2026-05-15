---
title: "Glossary"
description: "Comprehensive definitions for DeFi, TradFi, and statistical terms used in AIMM documentation"
audience: both
type: explanation
status: live
phase: n/a
order: 6
lang: en
publish: true
---
# Glossary

> Comprehensive definitions for DeFi, TradFi, and statistical terms used in AIMM documentation

---

## A

### Agave {#agave}
Solana validator client maintained by Anza (formerly Solana Labs). Rust implementation providing standard execution and consensus (Tower BFT + Proof of History). The "vanilla" Solana client without MEV features, contrast with Jito-Solana which adds bundle auction capabilities. Most validators run Jito-Solana for MEV revenue, but Agave remains the reference implementation.

**See**: [Jito](#jito), [Firedancer](#firedancer), [Consensus](#consensus).

### Adverse Selection {#adverse-selection}
Risk that counterparties possess superior information about fair value. In AMMs, informed traders extract value by trading against stale prices, whether from reserves (Uniswap), internal oracles (Curve v2), or external feeds. Adverse selection is the mechanism behind [LVR](#lvr-loss-versus-rebalancing), ranging from benign [Statistical Arbitrage](#statistical-arbitrage-stat-arb) to devastating [Informed Order Flow](#informed-order-flow-alpha-toxicity).

**AIMM mitigation**: Directional fees charge more when trades worsen [Coverage Ratio](#coverage-ratio).

See [Arbitrage](#arbitrage), [Toxic Flow](#order-flow-toxicity), [LVR](#lvr-loss-versus-rebalancing), [Adversarial Behavior](#adversarial-behavior).

### Adversarial Behavior {#adversarial-behavior}
**Value extraction without protocol or LP consent.** Strategies that profit by exploiting information asymmetries, timing advantages, or system inefficiencies at the expense of other market participants. Adversarial actors maximize personal gain without regard for protocol health or LP profitability.

**Categories**:
- **Information-based**: [Frontrunning](#frontrunning), [Sandwich Attacks](#sandwich-attack), [Informed Order Flow](#informed-order-flow-alpha-toxicity)
- **Timing-based**: [Latency Arbitrage](#latency-arbitrage), [JIT Liquidity](#jit-just-in-time-liquidity)
- **Block-ordering**: [MEV](#mev-maximal-extractable-value), [Greasing](#greasing), [MEV Spoofing](#mev-spoofing)
- **Price-based**: [Statistical Arbitrage](#statistical-arbitrage-stat-arb) (when purely extractive), [Toxic Flow](#order-flow-toxicity)

**Economic impact**: Adversarial behavior imposes [Adverse Selection](#adverse-selection) costs on LPs, increases [LVR](#lvr-loss-versus-rebalancing), and widens spreads. While some extraction is unavoidable (stat arb), protocols aim to convert adversarial behavior into [Cooperative Behavior](#cooperative-behavior) through rebates and alignment mechanisms.

**Examples**: Sandwich attack on DEX trade (harmful), latency arbitrage against stale oracle (extractive), JIT liquidity frontrunning passive LPs (fee theft).

See [Cooperative Behavior](#cooperative-behavior), [MEV](#mev-maximal-extractable-value), [Arbitrage](#arbitrage), [Order Flow Toxicity](#order-flow-toxicity).

### AMMs (Automated Market Makers) {#amms-automated-market-makers}
**Master entry for AMM designs.** Protocols enabling decentralized trading via algorithmic pricing rather than order books. All AMMs trade off capital efficiency, slippage, and MEV exposure.

**Architecture Families**:

| Family | Examples | Capital Efficiency | Price Impact | Model |
|--------|----------|-------------------|----------|-------|
| **[CSMM](#csmm-constant-sum-market-maker)** | Component of hybrid CFMM eg. Curve | Very high | Zero | x+y=k linear |
| **[CPMM](#cpmm-constant-product-market-maker)** | Uniswap V1/V2, Raydium | Low | Convex | x·y=k hyperbolic |
| **[CFMM](#cfmm-constant-function-market-maker)** | Curve, Balancer, Gyroscope | Variable | Convex/Flat | Invariant F(x₁,...,xₙ)=k |
| **[CLMM](#clmm-concentrated-liquidity-market-maker)** | Uniswap V3/V4, Raydium CLMM, Orca | Very high | Tick-dependent | Tick-based ranges |
| **[DLMM](#dlmm-dynamic-liquidity-market-maker)** | LFJ V2, Meteora, Saros | Very high | Bin-dependent | Bin-based + dynamic fees |
| **[CCMM](#ccmm-circularorbital-constant-market-maker)** | Orbswap | Highest | Bounded | n-dimensional sphere |
| **[AIMM](#aimm-adaptive-inventory-market-maker)** | BTR | Highest | Spline-shaped | Anchor-path pricing + inventory |

**Core Concepts**:
- [Invariant](#invariant) - Mathematical constraint defining price curve
- [Liquidity Shaping](#liquidity-shaping) - Dynamic adjustment of depth
- [Liquidity Concentration](#liquidity-concentration) - Range/bin-based strategies
- [Price Impact](#price-impact) - Cost of trade size
- [Slippage](#slippage) - Deviation from expected price

#### CSMM (Constant Sum Market Maker)
CFMM with x + y = k, giving linear pricing with zero slippage inside the feasible region. Capital-inefficient for large price moves; typically used only near tight pegs or as a component in hybrid designs (e.g., constant-sum segment near peg in stableswap curves).

#### CPMM (Constant Product Market Maker)
Special case CFMM with two assets where x·y = k, giving a hyperbolic bonding curve. Marginal price is p = y/x (up to fees). Provides simple, path-independent pricing with convex slippage proportional to trade size relative to pool depth. Canonical design in Uniswap V2 and many Solana AMMs.

#### CFMM (Constant Function Market Maker)
General class of AMMs where the state (reserves) of a pool lies on a level set F(x₁,...,xₙ)=k, and trades move the state along this surface. CFMM includes constant-product (CPMM), constant-sum (CSMM), weighted, stableswap, and hybrid curves. Umbrella category for most modern AMMs.

#### CLMM (Concentrated Liquidity Market Maker)
CFMM where LP liquidity is allocated to user-chosen price ranges via ticks rather than spread over all prices. Within an LP's active range, the pool behaves like a local CPMM; outside the range, that LP's liquidity is inactive. Increases capital efficiency and fee density but requires active rebalancing and exposes LPs to out-of-range risk. Examples: Uniswap V3/V4, Raydium CLMM, Orca Whirlpools, PancakeSwap V3, Aerodrome (V3-compatible).

#### DLMM (Dynamic Liquidity Market Maker)
AMM where liquidity is organized into discrete price "bins" (rather than continuous [ticks](#tick)) with dynamic fee adjustments and liquidity repositioning. **Architecture**: Unlike [CLMM](#clmm-concentrated-liquidity-market-maker) where ranges of N ticks act as local [CPMM](#cpmm-constant-product-market-maker) between endpoints, each DLMM bin acts as independent [CSMM](#csmm-constant-sum-market-maker) with near-zero slippage inside the bin. LP positions are **fungible** per bin (not per individual tick). Volatility-linked fees raise spreads in volatile regimes. **Implementations**: [Liquidity Book](#liquidity-book) (Trader Joe LFJ V2, Avalanche), Meteora (Solana), Cetus (Sui/Aptos), Saros (Solana). See [Liquidity Range](#liquidity-range) for CLMM-DLMM comparison and [Concentrated Liquidity](#concentrated-liquidity) for architecture context.

#### CCMM (Circular/Orbital Constant Market Maker)
Advanced AMM design for **pegged-asset-only** multi-asset pools (2 to 10,000+ assets), using n-dimensional spherical geometry to concentrate liquidity around the peg. Sphere invariant: $\sum_{i=1}^{n}(r - x_i)^2 = r^2$. Polar ticks are hyperplanes $\vec{x}\cdot\vec{v} = k$ along the equal-price diagonal $\vec{v} = \frac{1}{\sqrt{n}}(1,\dots,1)$; combining interior and boundary ticks yields a torus invariant. Marginal price $\partial x_i/\partial x_j = (r-x_j)/(r-x_i)$. Reported capital efficiency vs Curve: ~15× at 0.90 depeg threshold, ~150× at 0.99 (n=5). **Intrinsic risk isolation**: sphere geometry asymmetrically drains a depegged asset rather than propagating loss to the rest of the pool — a depegged coin "becomes worthless much faster than a traditional AMM curve" while others keep trading at efficient prices. **Limitation**: invariant assumes correlated/pegged assets; cannot price volatile-vs-pegged pairs. Pioneered by Paradigm's Orbital research and implemented by [Orbswap](https://orbswap.org) (superellipse extension with tunable concentration). Direct competitor to Curve StableSwap for stables-only deployments; complementary to (not competing with) BTR AIMM for mixed-volatility pools. See [Foundations §10](/docs/concepts/foundations) for the sphere invariant, polar-tick math, and torus extension.

#### AIMM (Adaptive Inventory Market Maker)
BTR's [AMM](#amms-automated-market-makers) design that combines internal or external oracle mid-pricing, inventory-based mid-price adjustment, and spline-defined liquidity depth for market impact calculation. No pool-wide reserves/price invariant unlike strict invariant-based AMMs like Uniswap or Curve. **Status**: Phase 2 (post-Supply launch).

### ALM (Asset-Liability Management) {#alm-asset-liability-management}
Accounting framework tracking what a pool *owns* (assets/reserves) separately from what it *owes* (liabilities to LPs). Enables single-sided deposits and explicit undercollateralization tracking. See: [Coverage Ratio](#coverage-ratio).

### Anchor (Token) {#anchor-token}
In the anchor tree topology, an asset's parent node used for price routing. The base token (typically a stablecoin) is the root anchor with `anchor = address(0)`.

### Anchor Tree {#anchor-tree}
Hierarchical token topology where all assets connect through parent-child relationships to a root (base token). Enables multi-hop routing via Least Common Ancestor (LCA) algorithm.

### Anchor Path Pricing {#anchor-path-pricing}
Multi-hop pricing architecture that routes asset swaps through a tree of anchor relationships, separating **pricing** (how assets relate to each other) from **accounting** (measurement in base token denomination). Each asset has one **anchor** (pricing counterpart); swaps route through the **Lowest Common Ancestor (LCA)** path. **Edge hops** (entry/exit) apply full price impact and modify reserves; **intermediate hops** serve as pricing numeraires only. Solves volatility misrepresentation in correlated asset pairs by pricing wstETH↔WETH directly (2% volatility) rather than both against USDC individually (49% geometric mean). See [Anchor Tree](#anchor-tree), [Anchor (Token)](#anchor-token), and `docs/1. AIMM/1.2. Pricing/1.2.3. Anchor Path Pricing.md` for architecture details.

### APY (Annual Percent Yield) {#apy-annual-percent-yield}
The return on investment calculated over a one-year period, expressed as a percentage of the initial investment. Accounts for compounding effects, making it more accurate than simple APR for yield-bearing positions. Standard metric for staking yields, lending rates, and yield farming returns.

### Arbitrage {#arbitrage}
**Master entry for extraction taxonomy.** Strategies exploiting price discrepancies or information advantages to extract value. AIMM's approach: doesn't distinguish by trader intent, only by **coverage impact** (beneficial vs harmful trades).

**AIMM's Coverage-Based Model**:
- **Coverage-improving trades** (beneficial): Rebalance inventory, pay only base spread
- **Coverage-worsening trades** (harmful): Worsen imbalance, charged deviation surcharge

**Arbitrage Taxonomy** (Hierarchy of Toxicity):
1. [Statistical Arbitrage (Stat Arb)](#statistical-arbitrage-stat-arb) - Price discrepancies across venues. Most common, least harmful, economically benign.
2. [Latency Arbitrage](#latency-arbitrage) - Exploits speed/network lags. Infrastructure-dependent, controversial.
3. [Informed Order Flow (Alpha Toxicity)](#informed-order-flow-alpha-toxicity) - Superior information (news, insider). Rare, devastating, pure adverse selection.
4. [Markout Toxicity (Winner's Curse)](#markout-toxicity-winners-curse) - Losses in trends. LP systematically wrong side, not information-driven.
5. [Flow Internalization (PFOF)](#flow-internalization-pfof---payment-for-order-flow) - Retail routed elsewhere, toxic flow to AMM. Structural market design issue.
6. [JIT (Just-In-Time) Liquidity](#jit-just-in-time-liquidity) - Fee extraction via block-level timing. ~0.5-1% of DEX volume.

#### Statistical Arbitrage (Stat Arb)
Trading strategy exploiting temporary price discrepancies between correlated or identical assets across different venues (CEX vs DEX, L1 vs L2, or different blockchains). The arbitrageur observes that the asset trades at different prices across venues and simultaneously buys cheap and sells dear, capturing the spread. **Most common and least harmful form of [toxic flow](#order-flow-toxicity)**-it improves price consistency across markets and helps maintain pegs. Unlike [informed trading](#informed-order-flow-alpha-toxicity), stat arb relies solely on current price differences, not future information.

**Examples**:
- **CEX-DEX arb**: ETH at $2000 on Coinbase but $2010 on Uniswap → buy on Coinbase, sell on Uniswap
- **Cross-chain arb**: USDC at 1.00 on Ethereum, 0.995 on Arbitrum → buy on L2, sell on L1
- **Triangular arb**: Exploit pricing inefficiencies across multiple trading pairs

**Not MEV**: Statistical arbitrage is **distinct from [MEV](#mev-maximal-extractable-value)**, which manipulates block ordering. Stat arb simply reacts to existing price differences without altering transaction sequencing.

**Broader category**: Stat arb is a subset of general **arbitrage**, which also includes MEV searchers (block-level extraction), HFT desks (latency-based), and medium-frequency traders (trend-following). See [Arbitrage](#arbitrage) for the full taxonomy.

The DEX LP experiences stat arb as [adverse selection](#adverse-selection), but it's economically benign, markets become more efficient even as LPs are arbitraged. AIMM's oracle pricing and volatility-aware spreads reduce stat arb extractability by minimizing price staleness. Additionally, [Cooperative Arbitrage](#cooperative-arbitrage) aligns stat arb incentives with LP protection via reputation-based rebates.

#### Latency Arbitrage
Strategies that exploit speed advantages to trade ahead of slower participants when public information or quotes change. **Infrastructure-dependent form of [toxic flow](#order-flow-toxicity)**. Common across L1-L2 and cross-chain scenarios: when prices move on mainnet (L1), sequencers on L2s (Arbitrum, Optimism) are still processing old transactions, creating stale price windows. A bot observes the L1 price crash and sells on the stale-priced L2 AMM. Requires fast execution infrastructure and low-latency connections to exploit sequencer lag. More benign than [informed flow](#informed-order-flow-alpha-toxicity) (doesn't require secret information, just speed). See [Arbitrage](#arbitrage) for the toxicity hierarchy. Controversial for potential market fairness impacts.

#### Informed Order Flow (Alpha Toxicity)
Trading activity by participants possessing superior information about true fair value, insiders, arbitrageurs with news advantage, or those with fundamental alpha. Classic [adverse selection](#adverse-selection) problem from market microstructure theory (Glosten-Milgrom). **Rare but devastating form of [toxic flow](#order-flow-toxicity)**. Unlike [Statistical Arbitrage](#statistical-arbitrage-stat-arb) (which exploits price lags), informed flow trades *before* prices move. Example: A whale trading ahead of a forthcoming protocol hack or partnership announcement. The LP sells an underpriced asset just before the market reprices. This is pure adverse selection, the LP loses on both the up-move and down-move, making informed flow structurally unprofitable for passive LPs. Contrasts with [Flow Internalization](#flow-internalization-pfof---payment-for-order-flow) where flows are pre-selected. See [Arbitrage](#arbitrage) for the toxicity hierarchy. AIMM combats this via oracle-first pricing (reducing information asymmetry) and deviation-based surcharges on toxic flow.

#### Markout Toxicity (Winner's Curse)
Loss LPs incur in trending markets even without direct arbitrage. In a sustained price trend (e.g., ETH rising), every trade against the pool becomes "toxic in hindsight"-LPs continuously sell the appreciating asset while missing upside. **Trend-driven form of [toxic flow](#order-flow-toxicity)** distinct from [information-based extraction](#informed-order-flow-alpha-toxicity). Markout toxicity is measured by comparing execution price to the asset's price 5-10 minutes after trade: if markout is consistently negative (price moved favorable to the taker), flow is toxic. High markout losses indicate LPs are systematically on the wrong side of trend moves. This is the "Winner's Curse"-by trading, the LP unknowingly bets against the winner. See [Arbitrage](#arbitrage) for the toxicity hierarchy. AIMM addresses this via oracle pricing that adjusts preemptively for trends and directional skew that charges trending trades more.

#### Flow Internalization (PFOF - Payment for Order Flow)
Market structure where major wallets or aggregators (MetaMask Swaps, 1inch) use private mempools or intent systems, giving first look to market makers or solvers. **Structural market issue causing [adverse selection of flow](#adverse-selection)**, a distinct category in the [toxicity hierarchy](#arbitrage). Uninformed retail flow is filled by MMs at favorable terms; toxic or imbalanced flow is rejected and sent to public AMMs. Result: AMMs become "dumping grounds" for adversarial traders, receiving only the worst flows while missing benign retail trades. LPs on public AMMs experience structurally skewed flow composition compared to venues capturing both [informed](#informed-order-flow-alpha-toxicity) and uninformed order flow. Relates to [Latency Arbitrage](#latency-arbitrage) in that both exploit information/timing advantages, but PFOF is about flow routing rather than speed. See [Arbitrage](#arbitrage) for the toxicity hierarchy.

#### MEV Strategies (Maximal/Miner Extractable Value)
[MEV](#mev-maximal-extractable-value) are a technical subset of latency arbitrage consisting in extracting value from altering blocks as they are being built and appended to the blockchain. See [MEV](#mev-maximal-extractable-value) for more.

**Related**: [MEV](#mev-maximal-extractable-value) (block-ordering exploitation), [Adverse Selection](#adverse-selection) (information asymmetry), [Order Flow Toxicity](#order-flow-toxicity) (umbrella term), [MEV Strategies](#mev-strategies) (block-level extraction methods).

**AIMM Mitigation**: Oracle pricing reduces staleness; directional fees charge toxic flow more; [Flow Guard](#flow-guard) blocks JIT; deviation surcharge penalizes informed flow.

**See**: [MEV](#mev-maximal-extractable-value), [Adverse Selection](#adverse-selection), [Order Flow Toxicity](#order-flow-toxicity).

### Anonymity Set {#anonymity-set}
Total group of potential participants in a private transaction. Larger sets provide stronger anonymity by making it harder to identify any single participant. Critical metric for evaluating privacy protocol robustness.

### AppChain {#appchain}
Blockchain optimized for a specific application, deployed as a Layer 3 or sovereign rollup. Often uses shared security from a Layer 2, focusing on application-specific functionality rather than general-purpose computation.

### Archive Node {#archive-node}
See [Node Types](#node-types).

### Avellaneda-Stoikov Framework {#avellaneda-stoikov-framework}
Market-making model that dynamically adjusts bid/ask spreads to balance inventory risk and adverse selection costs. Computes optimal spreads as function of: (1) inventory position, (2) risk aversion, (3) adverse selection costs.

**AIMM implementation**: Applied via [Inventory Skew](#inventory-skew), adjusting price premiums/discounts based on [Coverage Ratio](#coverage-ratio) deviation. Unlike invariant-based pricing (x·y=k), AIMM's oracle-aware pricing explicitly models adverse selection cost through directional spreads.

**See**: [Inventory Skew](#inventory-skew), [Spread](#spread), `docs/1. AIMM/1.2. Pricing`.

---

## B

### B64 Float Encoding {#b64-float-encoding}
Compact 64-bit floating-point representation used for price storage:
- Bits [0-51]: Mantissa (52 bits)
- Bits [52-56]: Decimal places (5 bits)
- Bits [57-63]: Exponent + bias (7 bits)

Enables efficient on-chain storage of prices with varying magnitudes and precision.

### Base Token (Numéraire) {#base-token-num-raire}
Root asset in a pool's anchor tree, typically a stablecoin (USDC, USDT), WETH, or WBTC. All other assets price relative to it, serves as the pool's accounting basis.

**AIMM config**: `PoolStorage.baseToken`.

**See**: [Anchor Tree](#anchor-tree), [Anchor Path Pricing](#anchor-path-pricing).

### BLAKE2b {#blake2b}
Fast cryptographic hash function optimized for modern processors. Produces 512-bit output by default. Used in privacy protocols as alternative to SHA-3. Faster than MD5 while maintaining cryptographic security.

### BLAKE3 {#blake3}
Modern cryptographic hash function offering parallel hashing, incremental verification, and keyed hashing. Faster than BLAKE2b while maintaining security. Supports variable-length output and streaming.

### Block Builder {#block-builder}
Entity specializing in block construction within proposer/builder separation (PBS) architecture. Builders receive transactions and bundles from mempool and searchers, simulate for validity, order to maximize value, and construct blocks for validators to propose.

**See**: [MEV](#mev-maximal-extractable-value), [Bundle](#bundle).

### BPS (Basis Points) {#bps-basis-points}
Unit equal to 1/100th of 1% (0.01%).

**AIMM precision**:
- `BPS_PRECISION = 1,000,000` (0.0001% units)
- 10,000 BPS = 1%
- 100 BPS = 0.01%

### Brain Wallet {#brain-wallet}
Wallet derived from user-memorized passphrase. User-chosen entropy is typically weak; security depends on passphrase strength. Generally discouraged due to loss risk from weak passwords.

### Bridge {#bridge}
Protocol infrastructure enabling asset or data transfer between independent blockchains. Introduces trust assumptions and security considerations for cross-chain verification.

**Types**:
- **[Asset Bridge](#asset-bridge-token-bridge)**: Token/NFT transfers
- **[Messaging Bridge](#messaging-bridge-arbitrary-messaging-bridge---amb)**: Arbitrary data and contract calls

**Mechanisms**: Optimistic (fraud proofs), light-client (cryptographic verification), liquidity networks.

#### Asset Bridge (Token Bridge)
Bridges for moving digital assets between chains. Operates via lock-and-mint (lock source, mint wrapped on destination) or burn-and-release (burn wrapped, release from escrow). Canonical version typically on native chain.

#### Messaging Bridge (Arbitrary Messaging Bridge - AMB)
Bridges enabling arbitrary data and smart contract communication between chains. More flexible than asset bridges, enable cross-chain contract calls and governance execution.

**Examples**: LayerZero, Wormhole, OP Stack bridges.

**Verification**: Cryptographic proofs (zk, merkle), optimistic protocols (fraud proofs), or light clients.

### BTR Token {#btr-token}
AIMM protocol governance token with fixed maximum supply. Used for voting and reward boosting when staked as sBTR.

**See**: [Claim Power](#claim-power).

### Bulletproof {#bulletproof}
Zero-knowledge proof system for range proofs without trusted setup. Used in Solana's Confidential Transfers to encrypt amounts while revealing sender/receiver. More efficient than SNARKs for specific range-proof use cases.

### Bundle {#bundle}
Atomic sequence of transactions submitted as a unit (typically by MEV searchers). If any transaction fails, entire bundle reverts, atomicity is critical for coordinated front-running and back-running strategies. Submitted to builders or MEV relays.

**See**: [MEV](#mev-maximal-extractable-value), [Block Builder](#block-builder).

---

## C

### Catmull-Rom Spline {#catmull-rom-spline}
**See also** [Fritsch-Carlson Monotone Cubic Hermite Interpolation](#fritsch-carlson-monotone-cubic-hermite-interpolation) — the canonical AIMM method. Catmull-Rom was considered and rejected because it is **not monotone** in general (uses centered secants without sign-preservation or the `α² + β² ≤ 9` clamp). See [Spline (Cubic Interpolation)](#spline-cubic-interpolation).

### Certificate Authority (CA) {#certificate-authority-ca}
Trusted entity that issues and validates digital certificates for PKI systems. In blockchain context, used for identity verification in enterprise systems and secure key management infrastructure.

### Circuit Breaker {#circuit-breaker}
Emergency mechanism that halts operations when safety thresholds are violated. In AIMM:
- `FROZEN_BIT` flag halts all asset operations
- Critical coverage triggers maximum spread premium
- Oracle staleness prevents pricing

### Claim Power {#claim-power}
Multiplier applied to LP rewards based on sBTR coverage ratio. Ensures LPs stake governance tokens to maximize yield:
```
ClaimPower = min(1.0, sBTR_value × matching_ratio / sLP_value)
```

### CLOB (Central Limit Order Book) {#clob-central-limit-order-book}
Order-driven market where buy and sell limit orders are matched by price-time priority. Core market structure for traditional exchanges and some DEXes. Contrast with AMMs which use algorithmic pricing.

**See**: [AMMs](#amms-automated-market-makers), [Market Order](#market-order), [Limit Order](#limit-order).

### Commitment {#commitment}
Cryptographic hash of a secret value used in privacy protocols (e.g., mixers). User generates random secret, creates commitment hash, sends on-chain. The secret and commitment together form the deposit note for future withdrawal.

### Concentrated Liquidity {#concentrated-liquidity}
AMM design where LPs provide liquidity within specific price ranges rather than across all prices.

**Architectures**:
- **[CLMM](#clmm-concentrated-liquidity-market-maker)**: Tick-based (Uniswap V3/V4), ranges act as local CPMM
- **[DLMM](#dlmm-dynamic-liquidity-market-maker)**: Bin-based (Trader Joe, Meteora), each bin acts as CSMM

**Concentration strategies**:
- **[Quadratic Concentration](#quadratic-concentration)**: Gyroscope 2-CLP
- **[Elliptical Concentration](#elliptical-concentration)**: Gyroscope E-CLP (multi-asset)

**Tradeoffs**: Increases capital efficiency and fee density but requires active rebalancing.

**AIMM approach**: Uses spline-based [Liquidity Profiles](#liquidity-profile) managed at pool level, not individual LP ranges.

**See**: [Liquidity Shaping](#liquidity-shaping), [Liquidity Range](#liquidity-range).

#### Elliptical Concentration
Concentration mechanism for multi-asset pools using elliptical price curves. Transforms a "constant circle" via: stretch (λ), rotation (φ), displacement (α, β). Parameters fixed at deployment.

**Advantages**: Multi-asset concentrated liquidity more elegant than multiple 2-asset CLMM instances. Useful for stablecoin baskets and correlated assets.

**Implementation**: Gyroscope E-CLP.

**See**: [Quadratic Concentration](#quadratic-concentration), [CFMM](#cfmm-constant-function-market-maker).

#### Quadratic Concentration
Concentration for two-asset pools using quadratic functions within price range [α, β]. Tighter capital efficiency than CPMM (hyperbolic) while maintaining bounded slippage.

**Curve spectrum**: More flexible than CSMM (linear), more efficient than CPMM.

**Implementation**: Gyroscope 2-CLP.

**See**: [Elliptical Concentration](#elliptical-concentration), [Concentrated Liquidity](#concentrated-liquidity).

### Confidential Transfer {#confidential-transfer}
Transaction hiding amount while revealing sender/receiver addresses. Uses zero-knowledge proofs (e.g., Bulletproofs on Solana) to prove validity without revealing value. Provides value privacy without full anonymity.

### Consensus Client {#consensus-client}
Node software managing [consensus](#consensus) participation, block proposals, validator attestations, and finality. Post-Merge on Ethereum, operates as separate module from [execution client](#execution-client).

**Responsibilities**:
1. **Block proposal**: Selected validators propose new blocks (via [LMD-GHOST](#lmd-ghost) fork choice)
2. **Attestation**: Validators sign attestations confirming block validity and chain position
3. **Finality**: Accumulate checkpoint votes for [Casper FFG](#casper-ffg) finality (~12.8 min)
4. **Slashing detection**: Monitor for validator misbehavior and enforce penalties

**Ethereum consensus clients**:
| Client | Language | Market Share | Key Features | Use Case |
|--------|----------|--------------|--------------|----------|
| [Lighthouse](#lighthouse) | Rust | ~35% | Memory-efficient, slasher, low footprint | General purpose, validators |
| [Prysm](#prysm) | Go | ~30% | Feature-rich, monitoring, web UI | Validators with dashboards |
| Teku | Java | ~10% | Enterprise features, institutional support | Enterprise validators |
| Lodestar | TypeScript | ~5% | Research-friendly, accessible codebase | Development, research |
| [Nimbus](#nimbus) | Nim | ~5% | Ultra-lightweight, embedded-friendly | Low-resource environments |

**Communication with execution client**: Receives blocks from execution client via Engine API, validates them, participates in [Gasper](#gasper) [consensus](#consensus).

**Validator operation**: Consensus client runs alongside optional separate validator client (software that holds validator keys and signs attestations/proposals). Some clients bundle both (e.g., [Lighthouse](#lighthouse)), others separate (e.g., [Prysm](#prysm)).

See [Execution Client](#execution-client), [Validator](#validator), [Gasper](#gasper).

### Cooperative Behavior {#cooperative-behavior}
**Value-aligned extraction with protocol and LP benefit sharing.** Strategies where actors profit by providing services to the protocol while sharing gains with LPs or returning value to the protocol treasury. Cooperative actors optimize for long-term protocol health alongside personal profit.

**Categories**:
- **Rebate-based**: [Cooperative Arbitrage](#cooperative-arbitrage), [Cooperator](#cooperator)
- **Service provision**: [Backrunning](#backrun) (captures arb without harming preceding tx), beneficial JIT (provides depth for large orders with transparent fees)
- **Protocol-aligned**: Liquidations (maintain solvency), rebalancing arbitrage (bring pool to fair price)

**Characteristics**: Transparent operations, value sharing via rebates or protocol fees, reputation-based incentives, DAO-governed access, long-term alignment over short-term extraction.

**Economic impact**: Cooperative behavior reduces net [LVR](#lvr-loss-versus-rebalancing) for LPs through rebates, improves price discovery, and maintains protocol health. Converts [Adversarial Behavior](#adversarial-behavior) into mutualistic relationships.

**Examples**: Cooperator donating 50% of arb profits back to LPs (cooperative), backrunning liquidation without frontrunning victim (benign), rebalancing pool to CEX prices (beneficial stat arb).

See [Adversarial Behavior](#adversarial-behavior), [Cooperative Arbitrage](#cooperative-arbitrage), [Cooperator](#cooperator), [Statistical Arbitrage](#statistical-arbitrage-stat-arb).

### Cooperative Arbitrage {#cooperative-arbitrage}
> 🚧 **(roadmap, not yet implemented)** — Cooperative Arbitrage is proposed BTR DEX functionality; no on-chain Solidity exists in the current release. The mechanism below describes proposed behavior.

AIMM's **(proposed) whitelisted, reputation-based rebate program** that would transform adversarial CEX-DEX [Statistical Arbitrage](#statistical-arbitrage-stat-arb) into collaborative LP protection. Unlike traditional AMMs where arbitrageurs extract LVR unilaterally, Cooperative Arbitrage would align incentives via:

**Mechanism**:
1. **Whitelist application**: Arbitrageurs apply to DAO for Cooperator status
2. **Equal starting rebate**: All approved cooperators begin with same tier (e.g., 20%)
3. **Reputation tracking**: `reputation = total_donations / total_rebates`
4. **Tiered rebates**: Higher reputation → higher rebate % (up to 80% max)
5. **Revocation**: Low reputation (<0.9) → loss of Cooperator status
6. **Cooldown claiming**: Rebates accrue per-swap, claimable after cooldown (default: 1 week)

**Not MEV**: Targets **cross-exchange (CEX-DEX) statistical arbitrage**, not MEV (which manipulates block ordering). Cooperators are **arbitrageurs**-a broader term encompassing MEV searchers, HFT desks, and medium-frequency stat arb traders.

**How it fights LVR**:
- Rebates lower profit threshold → cooperators arbitrage faster → smaller LVR window
- High-reputation cooperators would donate proceeds → LPs would recover LVR losses directly
- DAO could run bot with 100% donation rate → maximal reputation → would close LVR loop

**Comparison with alternatives**:
- **vs. McAMM**: No builder dependency (works on any EVM), simpler (no auction mechanics)
- **vs. am-AMM**: Distributed competition (max 10 cooperators) instead of single manager
- **vs. UAMM/DODO/Swaap**: No oracle single-point-of-failure, internal TWAP primary

**Tradeoff**: Not fully permissionless (requires DAO curation), but avoids strong trust assumptions of oracle-first or manager-controlled designs.

**Parameters** (pool-level governance):
- `maxCooperators`: Max whitelisted cooperators (default: 10)
- `maxRebateBps`: Max rebate % (default: 8000 = 80%)
- `minReputation`: Min reputation threshold (default: 0.9)
- `rebateCooldown`: Rebate claim cooldown (default: 1 week)

See: [Toxic Flow Mitigation § Cooperative Arbitrage](/docs/1.1.6-Toxic-Flow-Mitigation#36-cooperative-arbitrage), [Arbitrage](#arbitrage), [Statistical Arbitrage](#statistical-arbitrage-stat-arb), [LVR](#lvr-loss-versus-rebalancing).

### Cooperator {#cooperator}
Whitelisted arbitrageur in AIMM's [Cooperative Arbitrage](#cooperative-arbitrage) program. Earns rebates proportional to reputation (`donations / rebates` ratio).

**Terminology**: Cooperative Arbitrageur. **Not** "Coop MEV Searcher"-cooperators perform [Statistical Arbitrage](#statistical-arbitrage-stat-arb), not [MEV](#mev-maximal-extractable-value).

**See**: [Cooperative Arbitrage](#cooperative-arbitrage), [Arbitrage](#arbitrage).

### Collateral {#collateral}
**Assets deposited as security or backing for financial obligations.** In DeFi, collateral secures loans, backs derivatives positions, or provides liquidity guarantees. The ratio of collateral to obligations determines solvency and risk exposure.

**Collateralization States**:

| State | Ratio | Meaning | DeFi Context |
|-------|-------|---------|--------------|
| **Over-collateralized** | >100% | More assets than obligations | Safe, low liquidation risk |
| **Fully collateralized** | =100% | Assets equal obligations | Equilibrium, moderate risk |
| **Under-collateralized** | <100% | Obligations exceed assets | Risky, high liquidation risk |

**Common DeFi Uses**:
- **Lending protocols**: Borrowers deposit collateral (ETH) to borrow (USDC). Over-collateralization required (e.g., 150%) to absorb price volatility.
- **Derivatives**: Traders post collateral to back leveraged positions. Under-collateralization triggers liquidation.
- **Stablecoins**: Algorithmic stablecoins backed by collateral (e.g., DAI backed by ETH/USDC).
- **AMM pools**: Reserves serve as collateral backing LP token claims. [Coverage Ratio](#coverage-ratio) measures collateralization health.

**AIMM context**: Pool reserves function as collateral for LP liabilities. [Coverage Ratio](#coverage-ratio) (reserves / liabilities) measures collateralization state. Under-collateralization (<100%) triggers [Haircuts](#haircut) on withdrawals and premium [Inventory Skew](#inventory-skew) pricing.

**Liquidation**: When collateral value falls below required threshold, positions are forcibly closed to protect lenders/protocol. Critical mechanism preventing bad debt and maintaining solvency.

See [Coverage Ratio](#coverage-ratio), [Reserves](#reserves), [Liabilities](#liabilities), [Haircut](#haircut), [Liquidation](#liquidation).

### Coverage Ratio {#coverage-ratio}
Core ALM metric measuring pool health as `reserves / liabilities`-how much actual token the pool holds per unit of LP claim.

**States**:
- **100% (= 1.0)**: Fully collateralized (equilibrium), zero skew
- **< 100%**: Undercollateralized → [Inventory Skew](#inventory-skew) charges premium, [Haircuts](#haircut) on withdrawals
- **> 100%**: Overcollateralized → skew offers discount, encourages rebalancing

**Role in pricing**:
- **Skew**: [Gamma (γ)](#gamma-γ) controls quadratic penalty on coverage deviation
- **Haircut**: Quadratic penalty when coverage < 100% prevents bank runs
- **Decay**: Emergency mechanism reduces liabilities if undercollateralized

**Thresholds**: Typical bounds 50% floor, 200% ceiling. Critical levels trigger max skew (±100 bps) and emergency mechanisms.

**See**: [Inventory Skew](#inventory-skew), [Haircut](#haircut), [Decay (Liability)](#decay-liability), [ALM](#alm-asset-liability-management), `docs/1. AIMM/1.2. Pricing`.

### CREATE3 {#create3}
Deterministic deployment pattern (via Solady) for sLP token addresses. Ensures predictable addresses across chains.

**See**: [LP Token](#lp-token).

### Consensus {#consensus}
Agreement mechanism by which distributed nodes determine a canonical ordering of transactions and state. Consensus protocols vary in finality guarantees, throughput, decentralization, and security assumptions.

**Protocol Families**:

| Family | Examples | Finality | Safety Threshold |
|--------|----------|----------|------------------|
| **Nakamoto (PoW/PoS)** | Bitcoin, [Ouroboros](#ouroboros) | Probabilistic | 50% honest |
| **Classical BFT** | [PBFT](#pbft-practical-byzantine-fault-tolerance), [Tendermint](#tendermint--cometbft), [HotStuff](#hotstuff) | Deterministic | 2/3 honest |
| **Hybrid PoS+BFT** | [Gasper](#gasper), [Tower BFT](#tower-bft) | Hybrid | 2/3 honest |
| **DAG-based** | [Hashgraph](#hashgraph), [Narwhal](#narwhal)+[Bullshark](#bullshark) | Deterministic | 2/3 honest |
| **Rollup-inherited** | Arbitrum, Base | L1-dependent | L1 threshold |

**By Chain**:
- **Ethereum L1**: [Gasper](#gasper) = [LMD-GHOST](#lmd-ghost) + [Casper FFG](#casper-ffg)
- **Solana**: [Tower BFT](#tower-bft) + [Proof of History](#proof-of-history)
- **Sui**: [Narwhal](#narwhal) + [Bullshark](#bullshark)/[Mysticeti](#mysticeti)
- **BNB Chain**: [PoSA](#posa-proof-of-staked-authority)
- **Polygon PoS**: [Tendermint](#tendermint--cometbft)-like (Heimdall/Bor)
- **Cardano**: [Ouroboros](#ouroboros) (Praos/Genesis)
- **Polkadot**: [BABE](#babe) + [GRANDPA](#grandpa)
- **NEAR**: [Nightshade](#nightshade) + [Doomslug](#doomslug)
- **Tezos**: [Tenderbake](#tenderbake)
- **Hedera**: [Hashgraph](#hashgraph)
- **Cosmos ecosystem**: [Tendermint](#tendermint--cometbft)/[CometBFT](#tendermint--cometbft)
- **Hyperliquid**: [HyperBFT](#hyperbft)
- **Arbitrum/Base**: Sequencer ordering + Ethereum L1 finality

**Key Concepts**: [Validator](#validator), [Block Builder](#block-builder), [Sequencer](#sequencer), [Finality](#finality), [Slashing](#slashing).


#### BABE (Blind Assignment for Blockchain Extension)
Polkadot's block production mechanism using VRF-based slot leader election. In each slot, validators run a Verifiable Random Function weighted by stake to determine if they're elected to produce a block. Multiple leaders may be elected per slot (unlike single-leader protocols), creating potential forks resolved by [GRANDPA](#grandpa) finality.

**Mechanism**: Each validator computes `VRF(slot, secret_key)` and compares output to a threshold proportional to their stake. If below threshold, they produce a block referencing the best chain tip.

**Security**: Assumes <50% adversarial stake for liveness (block production continues) and relies on [GRANDPA](#grandpa) for deterministic finality. Part of Polkadot's hybrid [consensus](#consensus). See [Ouroboros](#ouroboros) for Cardano's similar VRF-based approach.


#### Bullshark
[Consensus](#consensus) protocol for Sui that provides total ordering on top of [Narwhal](#narwhal)'s DAG-based data availability layer. A partially synchronous BFT protocol achieving deterministic finality with 2/3 honest stake assumption.

**How it works**: Narwhal builds a DAG of transaction batches with availability certificates. Bullshark then interprets this DAG to determine a total order, validators "vote" on anchor points in the DAG, and commits happen when anchors gather enough support across rounds.

**Properties**:
- **Latency**: ~2-3 seconds to finality under good conditions
- **Throughput**: Decoupled from consensus (Narwhal handles data dissemination)
- **Safety**: Deterministic, no forks once committed if <1/3 Byzantine

Sui has since evolved to [Mysticeti](#mysticeti) for improved latency. See [Narwhal](#narwhal), [Consensus](#consensus).


#### Casper FFG (Friendly Finality Gadget)
Ethereum's finality mechanism providing economic finality on top of [LMD-GHOST](#lmd-ghost) fork choice. Part of [Gasper](#gasper) [consensus](#consensus).

**How it works**: Validators vote on checkpoint blocks (first block of each epoch, ~6.4 min). A checkpoint is **justified** when >2/3 of total stake votes for it. A justified checkpoint becomes **finalized** when the next checkpoint is also justified, creating a chain of supermajority attestations.

**Security guarantee**: If two conflicting checkpoints are both finalized, at least 1/3 of total stake must have voted for both (equivocation), triggering [slashing](#slashing). This makes finality economically irreversible, reverting a finalized block requires burning ~$10B+ in stake.

**Finality timeline**:
- Slot 0: Block proposed
- Slot 32 (epoch boundary): Checkpoint justified (~6.4 min)
- Slot 64 (next epoch): Previous checkpoint finalized (~12.8 min)

See [Gasper](#gasper), [LMD-GHOST](#lmd-ghost), [Validator](#validator), [Consensus](#consensus).


#### Doomslug
NEAR Protocol's block production and fast finality mechanism providing optimistic 1-block confirmation. Part of NEAR's [Nightshade](#nightshade) sharded [consensus](#consensus).

**How it works**: Block producers are selected by stake-weighted VRF. A block at height `h` is considered "doomslug-final" once it receives endorsements from >50% of block producers for height `h` AND the next block producer builds on it. This gives practical finality in ~1-2 seconds under honest majority.

**Guarantee levels**:
- **Doomslug finality** (~1-2s): Optimistic, assumes honest next block producer
- **BFT finality** (~2-3s): Full 2/3 attestation, resistant to Byzantine block producers

**Security**: Doomslug finality can be violated if the next block producer is malicious (they could build on a different fork), but BFT finality cannot be violated with <1/3 Byzantine stake.

See [Nightshade](#nightshade), [Consensus](#consensus), [BABE](#babe) for similar VRF-based selection.

#### GRANDPA (GHOST-based Recursive Ancestor Deriving Prefix Agreement)
Polkadot's finality gadget providing deterministic finality on top of [BABE](#babe) block production. A BFT protocol where validators vote on chains rather than individual blocks.

**How it works**: Validators broadcast votes for the highest block they consider final. When >2/3 of stake votes for a chain containing block B, all ancestors of B are finalized. This allows finalizing multiple blocks in a single round.

**Key properties**:
- **Batch finality**: Can finalize hundreds of blocks in one round after network partition heals
- **Asynchrony tolerance**: Continues making progress even with delayed messages
- **Safety**: Deterministic finality with <1/3 Byzantine stake

**Finality time**: Usually 12-60 seconds under normal conditions; can finalize large backlogs after partitions.

Part of Polkadot's hybrid [consensus](#consensus). See [BABE](#babe), [Consensus](#consensus), [Casper FFG](#casper-ffg) for Ethereum's finality gadget.


#### Gasper
Ethereum's [consensus](#consensus) protocol combining [LMD-GHOST](#lmd-ghost) (fork choice) with [Casper FFG](#casper-ffg) (finality). The name is a portmanteau of "GHOST" and "Casper."

**Architecture**:
- **LMD-GHOST**: Determines which chain tip to follow based on latest validator attestations
- **Casper FFG**: Provides economic finality via checkpoint voting every epoch (~6.4 min)

**Security properties**:
- **Liveness**: Continues producing blocks with >50% honest stake
- **Safety**: No conflicting finalized blocks unless >1/3 stake equivocates (slashable)
- **Finality**: ~12.8 minutes (2 epochs) for economic finality

**Execution flow**:
1. Validator selected by RANDAO proposes block
2. Committee of validators attest to block's validity and chain position
3. Attestations feed into LMD-GHOST for fork choice
4. Checkpoint voting accumulates for Casper FFG finality

See [Casper FFG](#casper-ffg), [LMD-GHOST](#lmd-ghost), [Validator](#validator), [Consensus](#consensus).


#### Hashgraph
Hedera's [consensus](#consensus) protocol using gossip-about-gossip and virtual voting to achieve asynchronous BFT with mathematically proven fairness guarantees.

**How it works**:
1. **Gossip-about-gossip**: Nodes gossip events (transactions + metadata about what they've seen)
2. **DAG construction**: Events form a directed acyclic graph with cryptographic links
3. **Virtual voting**: From the DAG structure, nodes deterministically compute what votes "would have been cast" without explicit voting rounds
4. **Consensus**: Events become "famous witnesses" when enough of the network has seen them

**Properties**:
- **aBFT (Asynchronous BFT)**: Safety guaranteed even with arbitrary network delays
- **Fairness**: Transaction order reflects median arrival time across nodes
- **Finality**: Deterministic, typically 3-5 seconds

**Math**: Safety if <1/3 of total stake is Byzantine. Virtual voting computes supermajority (>2/3) agreement on event ordering without message overhead.

Contrast with [Narwhal](#narwhal) (another DAG-based approach). See [Consensus](#consensus), [Finality](#finality).


#### HotStuff
Foundational BFT [consensus](#consensus) protocol (Facebook/Diem, 2019) providing linear message complexity and pipelined block production. Basis for many modern blockchain consensus protocols.

**Three-phase commit**:
1. **Prepare**: Leader proposes block, collects votes
2. **Pre-commit**: Leader broadcasts prepare certificate, collects pre-commit votes
3. **Commit**: Leader broadcasts pre-commit certificate, block is committed

**Key innovations**:
- **Linear complexity**: O(n) messages per view vs O(n²) for PBFT
- **Optimistic responsiveness**: Commits as fast as network allows (not timeout-bound)
- **Pipelining**: Multiple blocks in different phases simultaneously

**Derivatives**: [HyperBFT](#hyperbft) (Hyperliquid), Jolteon, Diem BFT, Aptos BFT. Contrast with [PBFT](#pbft-practical-byzantine-fault-tolerance) (quadratic), [Tendermint](#tendermint--cometbft) (two-phase).

See [Consensus](#consensus), [PBFT](#pbft-practical-byzantine-fault-tolerance), [Finality](#finality).


#### HyperBFT
Hyperliquid's [consensus](#consensus) protocol, a modified [HotStuff](#hotstuff)-style BFT engine optimized for high-frequency trading workloads.

**Properties**:
- **Sub-second finality**: Deterministic commit in <1 second under good conditions
- **Pipelined throughput**: Multiple blocks in flight (prepare/pre-commit/commit)
- **Validator rotation**: Round-robin leadership with stake-weighted selection

**Security**: Standard BFT guarantees, safety if <1/3 of validator stake is Byzantine, with committed blocks irreversible. Requires known validator set and reasonable network synchrony.

**Architecture**: Shared between HyperCore (perpetuals engine) and HyperEVM (general EVM execution), providing unified ordering across both execution environments.

See [HotStuff](#hotstuff), [Consensus](#consensus), [Finality](#finality).


#### LMD-GHOST (Latest Message Driven Greediest Heaviest Observed SubTree)
Ethereum's fork choice rule determining which chain tip validators should build on. Part of [Gasper](#gasper) [consensus](#consensus).

**How it works**: When choosing between competing forks, follow the branch with the most recent attestation weight. Each validator's "latest message" (attestation) counts toward the weight of the block it references and all ancestors.

**Algorithm**:
```
function get_head(store):
    head = store.justified_checkpoint.root
    while True:
        children = get_children(head)
        if children is empty:
            return head
        head = max(children, key=lambda c: get_latest_attesting_balance(c))
```

**Properties**:
- **Adaptive**: Reacts quickly to new attestations (vs fixed depth rules)
- **Sybil-resistant**: Weighted by stake, not number of validators
- **Complementary**: Provides liveness; [Casper FFG](#casper-ffg) provides finality

See [Gasper](#gasper), [Casper FFG](#casper-ffg), [Consensus](#consensus), [Finality](#finality).


#### Mysticeti
Sui's current [consensus](#consensus) protocol, replacing [Bullshark](#bullshark) for improved latency. An optimized DAG-based BFT achieving sub-second finality while maintaining [Narwhal](#narwhal)'s data availability properties.

**Improvements over Bullshark**:
- **Reduced latency**: ~390ms median finality vs ~2s for Bullshark
- **Pipelining**: Leaders can propose without waiting for previous round completion
- **Uncertified DAG**: Removes certificate overhead for faster block propagation

**Architecture**:
1. **DAG construction**: Validators build a DAG of blocks referencing prior rounds
2. **Commit rule**: Blocks commit when they have sufficient "support" from subsequent rounds
3. **Total ordering**: Committed DAG vertices provide deterministic transaction order

**Security**: Deterministic BFT finality with <1/3 Byzantine stake. Achieves optimal 3-round latency for commit.

See [Narwhal](#narwhal), [Bullshark](#bullshark), [Sui Core](#sui-core), [Consensus](#consensus).


#### Narwhal
Sui's DAG-based mempool and data availability layer, developed by Mysten Labs. Unlike traditional mempools (gossip-based) or leader-based forwarding ([Gulf Stream](#gulf-stream)), Narwhal organizes pending transactions into a Directed Acyclic Graph (DAG) structure before consensus.

**Architecture**:
- **Workers**: Accept transactions from users, batch them into blocks
- **Primary**: Builds DAG by referencing blocks from workers and other primaries
- **Certificates**: Cryptographic proofs that a block was seen by 2f+1 validators

**MEV Characteristics**: MEV on Sui differs from other chains because:
1. **Object-based model**: Contention only occurs on "shared objects" (accessed by multiple transactions). Pure object transactions have no ordering dependencies.
2. **DAG parallelism**: Unrelated transactions execute in parallel, reducing ordering-based MEV for independent trades.
3. **Concentrated contention**: MEV opportunities cluster around shared objects (e.g., AMM pools), where ordering still matters.

**Contrast with**:
- [Public Mempool](#mempool) (Ethereum): All transactions gossiped, anyone can observe all pending txs
- [Gulf Stream](#gulf-stream) (Solana): Leader-based forwarding, leader controls ordering
- [Sequencer](#sequencer) (Arbitrum/OP): Single entity orders all transactions

Narwhal is often paired with **Bullshark** (consensus protocol) or **Tusk** (total ordering) to achieve final transaction ordering.

See [Mempool](#mempool), [MEV](#mev-maximal-extractable-value), [Gulf Stream](#gulf-stream).


#### Nightshade
NEAR Protocol's sharded [consensus](#consensus) architecture providing horizontal scaling through parallel shard processing with unified security.

**How it works**:
1. **Chunk producers**: Each shard has validators producing "chunks" (shard blocks)
2. **Block producers**: Aggregate chunks into blocks on the main chain
3. **State sharding**: Each shard maintains independent state, with cross-shard receipts for communication

**Consensus components**:
- [Doomslug](#doomslug) for fast block finality
- BFT voting for cross-shard consistency
- Stake-weighted validator selection

**Security**: Validators are shuffled across shards to prevent adaptive adversary attacks. Security scales with total network stake, not per-shard stake.

See [Doomslug](#doomslug), [Consensus](#consensus), [Finality](#finality).


#### Ouroboros
Cardano's Proof-of-Stake [consensus](#consensus) protocol family (Ouroboros Praos, Ouroboros Genesis). Named after the mythological snake eating its own tail, symbolizing circular time divisions (epochs).

**How it works**:
1. **Slot leadership**: In each slot, a validator is elected by stake-weighted VRF to produce a block
2. **Longest chain**: Validators follow the densest/longest chain (similar to Nakamoto PoS)
3. **Probabilistic finality**: Reorg probability decreases exponentially with block depth

**Security**:
- **Honest majority**: <50% adversarial stake for liveness
- **Finality**: Probabilistic; k blocks deep gives reorg probability ~2^(-k)
- **In practice**: ~15-20 blocks (~5 minutes) considered "effectively final"

**Variants**:
- **Ouroboros Praos** (2017): Standard BFT with predictable leaders
- **Ouroboros Genesis** (2018): Improves bootstrapping and chain selection for recovering nodes
- **Ouroboros Leios** (upcoming): Adds separate input endorsers and block producers for higher throughput

Part of Cardano's layered consensus model. See [BABE](#babe) (Polkadot's similar VRF approach), [Consensus](#consensus), [Finality](#finality).


#### PBFT (Practical Byzantine Fault Tolerance)
Foundational classical BFT [consensus](#consensus) protocol (1999, Castro & Liskov) with quadratic message complexity. Guarantees safety and liveness with <1/3 Byzantine stake.

**Three-phase commit**:
1. **Pre-prepare**: Leader assigns sequence numbers to client requests
2. **Prepare**: Backups acknowledge receipt and promise to commit in this order
3. **Commit**: Backups execute the request when they see a quorum committing

**Properties**:
- **Deterministic finality**: Committed requests never reverted
- **Quadratic complexity**: O(n²) messages per view, problematic for large validator sets
- **View changes**: Designated leader changes on timeout, incurring complexity

**Successors**: [HotStuff](#hotstuff) (linear), [Tendermint](#tendermint--cometbft) (two-phase), [Baboon](#hashgraph) (gossip-based).

See [Consensus](#consensus), [HotStuff](#hotstuff), [Finality](#finality).


#### PoSA (Proof of Staked Authority)
BNB Chain's [consensus](#consensus) mechanism using a small validator set with stake-weighted voting. Effective BFT with centralization trade-offs.

**How it works**:
1. **Validator set**: ~21 active validators elected by staking and voting
2. **Block production**: Validators take turns producing blocks in a rotation
3. **Finality**: Block is committed when 2/3+ of validators have sealed it

**Security**:
- **Honest majority**: <1/3 of active validator stake can be Byzantine
- **Finality**: ~1-2 seconds (consensus-generated blocks)
- **Centralization**: Small validator set reduces decentralization but improves throughput

**Contrast**: Larger validator sets like [Consensus](#consensus) (Ethereum: 900k+ validators, lower throughput) vs smaller (BNB: 21 validators, higher throughput).

See [Consensus](#consensus), [Tendermint](#tendermint--cometbft), [Finality](#finality).


#### Proof of History (PoH)
Solana's cryptographic clock providing timestamps and ordering before [Tower BFT](#tower-bft) [consensus](#consensus). Not consensus itself, but a verifiable ordering mechanism.

**How it works**:
1. **Sequential hashing**: Leader continuously hashes (hash chain): H₀ → H₁ → H₂ → ...
2. **Tick encoding**: After N hashes, leader signs the hash and timestamp
3. **Transaction ordering**: Transactions are inserted into the hash chain in order received

**Properties**:
- **Verifiable**: Anyone can verify the chain of hashes (deterministic)
- **Order proof**: Timestamp proves transaction occurred between tick T and T+N
- **Throughput**: Allows pipelining, validators can begin processing before consensus completes

**Not consensus**: PoH alone doesn't prevent double-spending; [Tower BFT](#tower-bft) provides finality.

**Analogy**: Like a notary's timestamp, proving "this document existed at this time," but not proving agreement among multiple parties.

See [Tower BFT](#tower-bft), [Consensus](#consensus).


#### Tendermint / CometBFT
Classical BFT [consensus](#consensus) protocol (2014, Tendermint Core) using two-phase consensus (propose/commit). Used as the consensus engine for Cosmos ecosystem chains.

**Three-round voting**:
1. **Propose**: Leader proposes a block
2. **Pre-vote**: Validators vote on the proposal (or nil if invalid)
3. **Pre-commit**: Validators pre-commit when >2/3 pre-voted for the same block

**Properties**:
- **Deterministic finality**: Committed blocks cannot be reverted (vs Nakamoto)
- **Two-phase**: Simpler than [PBFT](#pbft-practical-byzantine-fault-tolerance) (3-phase) but adds one round
- **Partial synchrony**: Works under bounded network delays (not asynchronous)
- **Liveness**: Continues making progress with >2/3 honest stake

**Advantages**: Proven, widely implemented, clear safety/liveness guarantees.

**Disadvantages**: Requires known validator set, lower throughput than pipelined protocols.

**Adoption**: Cosmos, Polygon PoS, Berachain, Hedera (modified), Tezos (predecessor).

**Modern name**: CometBFT (renamed/refactored by Interchain Foundation in 2023).

See [Consensus](#consensus), [PBFT](#pbft-practical-byzantine-fault-tolerance), [HotStuff](#hotstuff), [Finality](#finality).


#### Tower BFT
Solana's [consensus](#consensus) protocol layered on top of [Proof of History](#proof-of-history). Uses exponentially increasing lockouts to achieve fast finality with stake-weighted voting.

**How it works**:
1. **Lockouts**: When a validator votes for a block, they become locked out of voting for competing forks for an exponentially growing duration
2. **Fork resolution**: Validators switch forks only if they see a competing fork with >2/3 support
3. **Finality**: When a validator chain receives sufficient support (lockouts accumulate), earlier blocks become irreversible

**Security properties**:
- **Safety**: If <1/3 of stake violates lockouts, they can be slashed
- **Liveness**: Continues producing blocks with >50% honest stake
- **Latency**: Very low (~400ms) assuming good network conditions

**Assumptions**:
- Tight network synchrony (validators see blocks within ~300ms of production)
- Honest majority of block producers
- Complex timing, deviations from assumptions can cause stalls

**Contrast**: [Gasper](#gasper) (Ethereum's hybrid approach) uses separate fork choice + finality; Tower BFT integrates both.

See [Proof of History](#proof-of-history), [Consensus](#consensus), [Gulf Stream](#gulf-stream), [Finality](#finality).

---

## D

### Dex adapter {#dex-adapter}
Concrete contract at `alm/evm/src/adapters/Dex.sol`. A multi-holder shim that wraps a single third-party concentrated-liquidity pool (Uniswap v3/v4, PancakeSwap V3/Infinity, Algebra, Ramses, Aerodrome Slipstream). Holders (Vaults, EOAs, multisigs) deposit principal; the adapter manages LP positions on the underlying pool and exposes a uniform NAV / `assetValue` surface.

NOT to be confused with **Dex** (the BTR-native AIMM product, source tree `dex/evm/`). "Dex adapter" = the alm-side integration shim; "Dex" (capitalized, no qualifier) = the BTR DEX product.

### Dark Pool {#dark-pool}
Private trading venue where transactions are hidden from public order book and mempool until settlement. Enables large trades without revealing volume, reducing market impact and information leakage.

**AIMM context**: Not currently implemented, but architecture supports privacy extensions.

**See**: [MEV](#mev-maximal-extractable-value), [Encrypted Mempool](#encrypted-mempool).

### Data Availability (DA) {#data-availability-da}
In rollups and Layer 2s, the requirement that transaction data be published and retrievable from L1, enabling fraud proofs and preventing censorship. Critical for rollup security, without DA, users cannot verify state or recover funds.

**See**: [Rollup](#rollup), [Modular Blockchain](#modular-blockchain).

### Decay (Liability) {#decay-liability}
Emergency mechanism gradually reducing LP claims when coverage remains below threshold. Protects remaining LPs from bank runs:

```
decayAmount = liabilities × decaySlope × dt / WAD
```

**AIMM parameters**: `decaySlope` (time-based decay rate), `decayThreshold` (coverage level triggering decay).

**See**: [Coverage Ratio](#coverage-ratio), [Haircut](#haircut), [ALM](#alm-asset-liability-management).

### Delegatecall {#delegatecall}
EVM opcode executing external contract code in caller's storage context.

**AIMM use**: none. Phase 42H removed the Diamond/proxy pattern; all cross-contract calls are now normal external calls.

### Deviation (Oracle) {#deviation-oracle}
Disagreement between multi-timeframe price signals:

```
Δ = max(|fastOffset - slowOffset|, |fastOffset - 0|)
```

High deviation indicates price uncertainty, triggering wider spreads in AIMM.

**See**: [Internal Oracle](#internal-oracle), [Feed](#feed-oracle).

### Diamond-Lite (historical) {#diamond-lite-historical}
A Diamond-lite proxy pattern was used by AIMM prior to **Phase 42H**. Replaced by standalone singletons + EIP-1167 `Pool` clones (no `delegatecall`, no ERC-7201 namespacing, no module trust). Retained here only as a glossary marker for historical references in audit / decision records.

### Dispersion {#dispersion}
The price range over which liquidity is distributed in AIMM's liquidity profiles. Dispersion scales dynamically with volatility, higher volatility widens the curve, lower volatility tightens it. This is AIMM's core innovation for volatility-responsive depth.

**Also known as**: range breadth, depth curve width. Sometimes described as "dynamic bonding curve" in comparison to traditional AMM invariant curves.

**Formula**:
```
dispersion = clamp(baseDispersion + σ × vega / MULT_BASE, minDispersion, maxDispersion)
```

**Parameters**:
- `minDispersion`: Floor (prevents free options during low volatility)
- `maxDispersion`: Ceiling (prevents illiquidity during extreme volatility)
- `vega`: Sensitivity to volatility

See [Liquidity Shaping](/docs/1.1.2-Liquidity-Shaping) for detailed mechanics.

---

## E

### ECDSA (Elliptic Curve Digital Signature Algorithm) {#ecdsa-elliptic-curve-digital-signature-algorithm}
Public-key cryptographic algorithm used for digital signatures. Based on elliptic curve mathematics. Ethereum and most blockchains use ECDSA with secp256k1 curve for transaction signing. Provides authentication and non-repudiation without revealing private keys.

### Ed25519 {#ed25519}
Modern elliptic curve signature scheme based on Curve25519. Faster and simpler than ECDSA while maintaining equivalent security. Used by Solana, some privacy protocols, and as alternative in protocols seeking better performance. Produces 64-byte signatures.

### EOA (Externally Owned Account) {#eoa-externally-owned-account}
Ethereum account controlled by a private key, as opposed to a smart contract account. EOAs can initiate transactions but cannot execute arbitrary code. Standard user accounts and validator accounts are EOAs. Account abstraction (EIP-4337, EIP-7702) blurs the distinction between EOAs and contract accounts.

### Execution Client {#execution-client}
Node software responsible for executing transactions, running smart contracts, and maintaining the state of accounts and storage. Post-Merge on Ethereum, execution clients operate as separate modules from [consensus clients](#consensus-client), communicating via the **Engine API** (JSON-RPC bridge).

**Responsibilities**:
1. **Transaction validation**: Check nonce, gas, signature, balance before accepting
2. **Block execution**: Apply transactions in order, updating state
3. **State management**: Maintain account balances, contract storage, nonce counters
4. **RPC serving**: Provide JSON-RPC interface for wallets and dApps (eth_call, eth_sendTransaction, etc.)

**Ethereum execution clients**:
| Client | Language | Market Share | Key Features | Use Case |
|--------|----------|--------------|--------------|----------|
| [Geth](#geth-go-ethereum) | Go | ~45% | Dominant, widely trusted, stable | General purpose, validators |
| [Nethermind](#nethermind) | C# | ~20% | MEV-Boost, JSON-RPC extensions | Enterprise, MEV infrastructure |
| [Reth](#reth-rust-ethereum) | Rust | <1% | Modular, high performance | Next-gen infrastructure |
| Besu | Java | ~5% | Enterprise features, private networks | Permissioned chains |
| Erigon | Go | ~10% | Archive-optimized, efficient storage | Block explorers, analytics |

**Communication with consensus client**: Engine API defines methods like `engine_newPayload` (execute block), `engine_forkchoiceUpdated` (update fork choice), enabling consensus client to request execution without knowing implementation details.

See [Consensus Client](#consensus-client), [Validator](#validator), [Full Node](#full-node).

### EIP (Ethereum Improvement Proposal) {#eip-ethereum-improvement-proposal}
**Master entry for Ethereum protocol improvements.** Formal design documents proposing changes to the Ethereum protocol. EIPs are peer-reviewed, numbered, and tracked through development stages (draft, review, last call, final). Covers core protocol upgrades (consensus, networking), client specifications, and application standards. **Every [ERC](#erc-ethereum-request-for-comment) is an EIP, but not every EIP is an ERC.**

**Key EIP Categories**:
- **Core**: Protocol-level changes (consensus, networking, VM)
- **Interface**: Application-level standards (most become ERCs)
- **Meta**: Process and governance improvements

See [ERC](#erc-ethereum-request-for-comment) for application-level standards.

#### EIP-712 (Typed Structured Data Hashing and Signing)
Standard for hashing and signing typed structured data on-chain. Users see human-readable typed data (domain, types, values) instead of opaque hex strings. Prevents signature reuse across domains and enables secure off-chain signature generation. Used by [ERC-2612](#erc-2612-erc-20-permit), [ERC-3009](#erc-3009-transfer-with-authorization), and meta-transaction systems.

#### EIP-1153 (Transient Storage)
Transaction-scoped storage cleared automatically at transaction end. Gas-efficient for temporary data. AIMM uses for reentrancy guards and oracle price caching (saves ~2,100 gas per hit vs persistent storage).

#### EIP-1559 (Fee Market Change)
Fee model with base fee (burned) and priority fee (to validator). Makes gas fees more predictable, reduces ETH supply via burn. Implemented in London upgrade (2021).

#### EIP-3675 (The Merge)
Transition from Proof-of-Work to Proof-of-Stake (September 2022). Reduced energy consumption by 99.95%, eliminated mining, introduced validators and staking.

#### EIP-4337 (Account Abstraction)
Enables account abstraction without core protocol changes. Smart contract wallets function like EOAs through EntryPoint contract. Enables gas payment in ERC-20 tokens, social recovery, transaction batching.

#### EIP-4844 (Proto-Danksharding)
Blob-carrying transactions for off-chain data storage. Reduces Layer 2 costs by separating transaction data from execution data. Foundation for full danksharding.

#### EIP-7201 (Namespaced Storage)
Storage layout standard preventing collisions in upgradeable contracts. Each module uses unique slot:
```solidity
bytes32 constant SLOT = keccak256("pool.storage.base.v1") - 1;
```

#### EIP-7702 (Delegated Execution)
Enables EOAs to temporarily delegate execution to smart contracts. Allows EOAs to behave as smart contract wallets for single transactions, enabling gas sponsorship and batching.

### ERC (Ethereum Request for Comment) {#erc-ethereum-request-for-comment}
**Master entry for Ethereum application-level standards.** Classification within the EIP system focused on smart contract standards and interfaces. ERCs define rules and interfaces for tokens, NFTs, and dApps, ensuring ecosystem interoperability. All ERCs are EIPs, but ERCs are application-focused rather than protocol-focused.

**Major ERC Categories**:
- **Token Standards**: ERC-20, ERC-721, ERC-1155
- **Token Extensions**: ERC-2612, ERC-3009, ERC-3156
- **Vault Standards**: ERC-4626, ERC-7540
- **Crosschain**: ERC-7802
- **Advanced**: ERC-8004

See [EIP](#eip-ethereum-improvement-proposal) for core protocol improvements.

#### ERC-20 (Token Standard)
Standard interface for fungible tokens. Defines transfer(), approve(), balanceOf(). Foundation for stablecoins, governance tokens, and utility tokens.

#### ERC-721 (NFT Standard)
Standard interface for non-fungible tokens. Each token has unique ID and metadata. Used for digital collectibles, art, gaming items.

#### ERC-1155 (Multi-Token Standard)
Standard managing both fungible and non-fungible tokens in single contract. Commonly used in gaming and metaverses.

#### ERC-2612 (ERC-20 Permit)
ERC-20 extension adding permit() for off-chain signature approvals. Eliminates separate approval transactions, saving gas. Uses [EIP-712](#eip-712-typed-structured-data-hashing-and-signing) for structured signing.

#### ERC-3009 (Transfer with Authorization)
Enables single-transaction token transfers using off-chain signatures. Uses [EIP-712](#eip-712-typed-structured-data-hashing-and-signing) for secure signature generation.

#### ERC-3156 (Flash Loan Standard)
Standardized interface for flash loans. Defines `IERC3156FlashLender` (lender) and `IERC3156FlashBorrower` (borrower callback). Unifies incompatible implementations from Aave, dYdX.

**AIMM**: Implements flash loan interface for capital-efficient arbitrage and liquidations.

See [Flash Loan](#flash-loan).

#### ERC-4626 (Yield-Bearing Vault Standard)
Universal interface for tokenized vaults (lending pools, yield aggregators). Standardizes deposit/withdrawal/share mechanics, improving DeFi composability.

#### ERC-7540 (Asynchronous Vault Standard)
Extension of [ERC-4626](#erc-4626-yield-bearing-vault-standard) adding asynchronous deposits and redemptions. Critical for real-world asset (RWA) protocols and cross-chain operations with delayed finality.

#### ERC-7802 (Crosschain Token Interface)
Minimal interface for tokens to communicate cross-chain. Enables bridges with mint/burn rights to relay transfers securely. Standardizes crosschain token operations across multiple blockchains.

#### ERC-8004 (Trustless Agents)
Standard for discovering and interacting with autonomous agents across organizational boundaries. Defines pluggable trust models (reputation, stake-secured re-execution, zkML proofs, TEE oracles) proportional to value at risk. Enables agent economies composing trust mechanisms matching task criticality.

### Encrypted Mempool {#encrypted-mempool}
Mempool where transaction details are hidden from block proposers until ordering is finalized, using threshold encryption. Prevents frontrunning and sandwiching, but may leak metadata (size, target) enabling speculative attacks. Active research area.

See [Mempool](#mempool), [MEV](#mev-maximal-extractable-value), [Threshold Encryption](#threshold-encryption).

### Execution Price {#execution-price}
See [Prices](#prices).

---

## F

### Fair Value {#fair-value}
See [Prices](#prices).

### Full Node {#full-node}
See [Node Types](#node-types).

### Feed (Oracle) {#feed-oracle}
Data structure containing price and volatility information:
```solidity
struct FeedData {
    uint64 lastPriceB64;    // Spot price (B64)
    int32 fastOffset;       // Fast TWAP vs spot
    int32 slowOffset;       // Slow TWAP vs spot
    uint32 fastVolEMA;      // Fast volatility
    uint32 slowVolEMA;      // Slow volatility
    uint32 updatedAt;       // Timestamp
    uint16 ttl;             // Staleness threshold
    uint8 confidence;       // 0-100
}
```

### Flash Loan {#flash-loan}
Uncollateralized loan that must be repaid within the same atomic transaction. If not repaid by end of transaction, entire transaction reverts. Enabled by EVM's atomicity and smart contract callbacks. Standardized by [ERC-3156](#erc-3156-flash-loan-standard), which provides universal interfaces (IERC3156FlashLender, IERC3156FlashBorrower) enabling interoperability across protocols. Enables liquidations, arbitrage, and composable DeFi strategies. AIMM charges `flashFeeBps` as fee.

### Fill Ratio {#fill-ratio}
Fraction of a resting order's size that ultimately executes before cancellation or expiry. Used to evaluate posting strategies and predict order completion probability. High fill ratios indicate strong liquidity demand at quoted prices.

### Finality {#finality}
The guarantee that a transaction or block cannot be reverted or reorganized. Different [consensus](#consensus) mechanisms provide different finality types:

| Type | Guarantee | Time | Examples |
|------|-----------|------|----------|
| **Probabilistic** | Reorg probability decreases exponentially with depth | Minutes-hours | Bitcoin PoW, [Ouroboros](#ouroboros) |
| **Economic** | Reorg requires burning slashable stake | ~10-15 min | [Gasper](#gasper) (Ethereum) |
| **Deterministic** | Mathematically impossible to revert | Seconds | [Tendermint](#tendermint--cometbft), [HotStuff](#hotstuff) |
| **Soft (L2)** | Sequencer-guaranteed, not L1-backed | Instant | Arbitrum, Base pre-batch |
| **Hard (L2)** | L1-finalized batch commitment | 10-30 min | Arbitrum, Base post-batch |

**Security threshold**: BFT-style finality requires <1/3 Byzantine stake; Nakamoto-style requires <50% adversarial hashpower/stake.

See [Consensus](#consensus), [Validator](#validator), [Sequencer](#sequencer).

### Firedancer {#firedancer}
Next-generation Solana [validator client](#validator) developed by Jump Crypto in C/C++. Designed for extreme performance with custom networking stack, memory management, and parallelization.

**Why it matters**:
- **Client diversity**: Reduces network risk from single-client bugs (currently ~85% [Jito](#jito)/[Agave](#agave))
- **Performance**: Target 1M+ TPS on commodity hardware through aggressive optimization
- **Independence**: Clean-room implementation, not a fork of Agave

**Status**: Frankendancer (hybrid using Firedancer networking + Agave execution) running on mainnet; full Firedancer validator in development.

See [Agave](#agave), [Jito](#jito), [Consensus](#consensus), [Validator](#validator).

### FCFS / FIFO (First-Come-First-Served / First-In-First-Out) {#fcfs-fifo-first-come-first-served-first-in-first-out}
Transaction ordering policy where transactions are processed in the order they arrive at the [sequencer](#sequencer) or validator. Used by some rollup sequencers (original Arbitrum model) to provide fair ordering without explicit [MEV](#mev-maximal-extractable-value) auctions. **Limitations**: FCFS is vulnerable to latency games, actors with faster connections to the sequencer can still front-run by submitting transactions microseconds earlier. Many chains have moved to hybrid models combining FCFS with priority fees or explicit MEV auctions ([Timeboost](#timeboost)). Contrast with [price-time priority](#price-time-priority) (CLOB ordering) and [gas auction](#gas-auction) (Ethereum L1 ordering). See [Sequencer](#sequencer), [Mempool](#mempool).

### Flow Guard {#flow-guard}
**Master entry for forbidden call flow protections.** Security mechanisms preventing illegitimate or malicious sequences of calls at transaction, block, or epoch level. Flow guards monitor and block call patterns that violate protocol invariants or enable attacks.

**Scope Levels**:
- **Transaction-level**: Single transaction execution ([Reentrancy Guard](#reentrancy-guard))
- **Block-level**: Multiple transactions within same block (time-based cooldowns)
- **Epoch-level**: Longer timeframes across multiple blocks (staking unbonding periods)

**Common Flow Guard Types**:

| Type | Scope | Protection | AIMM Implementation |
|------|-------|------------|---------------------|
| **[Reentrancy Guard](#reentrancy-guard)** | Transaction | Prevents recursive calls mid-execution | Transient storage (EIP-1153) |
| **Time-based Cooldown** | Block/Epoch | Enforces minimum time between operations | 15s deposit→withdraw cooldown |
| **Circuit Breaker** | Epoch | Halts operations during extreme conditions | Volatility/deviation thresholds |

**AIMM-Specific Flow Guards**:
- `deposit()` → `withdraw()`: Prevents instant LP fee extraction
- `stakeGov()` → `unstakeGov()`: Prevents instant voting/reward capture
- `stakeLP()` → `unstakeLP()`: Prevents instant reward farming

Default cooldown: 15 seconds. Configurable via `setFlowCooldown()`.

See [Reentrancy Guard](#reentrancy-guard), [Circuit Breaker](#circuit-breaker), [Flow Guards](/docs/3.4-Flow-Guards).

---

## G

### Geth (Go-Ethereum) {#geth-go-ethereum}
Dominant Ethereum [execution client](#execution-client) written in Go. ~45% market share. Standard full node setup: `geth --http --syncmode snap`. Requires pairing with [consensus client](#consensus-client).

See [Execution Client](#execution-client) for comparison table.

### Gamma (γ) {#gamma}
Per-asset sensitivity parameter controlling how aggressively pricing responds to [inventory skew](#inventory-skew) deviations. Higher gamma amplifies the quadratic penalty/discount curve when coverage deviates from target. **Tuning guidance**:
- **5000 (0.5x)**: Flat curve, gradual rebalancing incentives (conservative, smooth pricing)
- **10000 (1.0x)**: Linear-ish response (default, balanced)
- **20000 (2.0x)**: Steep curve, aggressive rebalancing (volatile assets, high deviation penalty)

**Formula**: `skew = sign × 100 × ((coverage - target) / (bound - target))^(γ/10000)`

**Economic role**: Gamma is the **risk aversion coefficient** from [Avellaneda-Stoikov](#avellaneda-stoikov-framework). Higher gamma means the pool is more risk-averse and charges steeper premiums for inventory imbalance. **Relation to LVR**: Higher gamma reduces [LVR](#lvr-loss-versus-rebalancing) by charging toxic flow more aggressively. See [Inventory Skew](#inventory-skew), [Vega (ν)](#vega-ν), [Lambda (λ)](#lambda-λ), and `docs/1. AIMM/1.2. Pricing/1.2.5. Parametrization.md` for implementation.

### Gas Auction {#gas-auction}
Transaction ordering mechanism on Ethereum L1 where transactions compete for block inclusion based on gas price (priority fee). Higher gas prices result in earlier inclusion, creating an implicit auction for block space and ordering priority. This is how [MEV](#mev-maximal-extractable-value) extraction occurs on L1-[searchers](#mev-searcher) bid up gas prices to ensure their transactions land in favorable positions (e.g., before a victim's transaction for [front-running](#frontrunning)). **Contrast with**:
- [FCFS/FIFO](#fcfs--fifo-first-come-first-served--first-in-first-out): Order by arrival time (some rollup sequencers)
- [Timeboost](#timeboost): Explicit auction for express lane priority (Arbitrum)
- [Price-Time Priority](#price-time-priority): CLOB ordering (exchanges)

Gas auctions create an adversarial environment where sophisticated actors with better gas estimation and faster infrastructure can consistently outbid retail users. This is why [MEV-protected RPCs](#mev-protected-rpc) exist, to bypass the public gas auction entirely. See [Mempool](#mempool), [Block Builder](#block-builder).

### Gas Optimization {#gas-optimization}
Techniques to reduce transaction costs:
- B64 encoding (75% storage reduction)
- Transient caching (2,100 gas/hit saved)
- Packed structs (single-slot reads)
- LCA without path caching

### Grace Period {#grace-period}
Time window after timelock expires during which operation can be executed. Default: 7 days. Prevents indefinitely pending operations.

### Greasing {#greasing}
See [MEV Strategies](#mev-strategies).

### Gulf Stream {#gulf-stream}
Solana's transaction forwarding protocol that eliminates traditional mempools by forwarding transactions directly to the current and upcoming slot leaders. Instead of broadcasting to all nodes (Ethereum-style gossip), validators forward transactions to the leader expected to produce the next few blocks, enabling sub-400ms transaction confirmation. **MEV implications**: While there's no public mempool to observe, the slot leader (and attached infrastructure like [Jito](#jito)) still controls transaction ordering and can extract MEV. Transactions arrive at the leader in rough order, but the leader has full discretion over final ordering. **Contrast with**:
- [Public Mempool](#mempool) (Ethereum): Global gossip, anyone can observe
- [Sequencer Queue](#sequencer) (Arbitrum/OP): Single sequencer controls ordering
- [Narwhal](#narwhal) (Sui): DAG-based mempool with different MEV characteristics

See [Mempool](#mempool), [Jito](#jito), [Sequencer](#sequencer).

---

## H

### Haircut {#haircut}
Penalty applied to withdrawals when coverage < 100%:
```
haircut = (1 - coverage)^p
actualAmount = grossAmount × (1 - haircut)
```
Where `p = 1 + (suppressor / 10000)`

### Helios {#helios}
Ethereum [light client](#light-node) written in Rust, providing trustless RPC access without running a full node. Verifies block headers and state proofs cryptographically.

**How it works**: Syncs with the beacon chain sync committee (512 validators), verifies their BLS signatures, and uses Merkle proofs to verify any state query against the signed state root.

**Use cases**:
- Wallets verifying balances without trusting Infura/Alchemy
- Mobile apps with limited resources
- Bridging verification

**Trust model**: Trusts that >2/3 of the sync committee is honest (same as full node assumption, but sampled subset).

See [Light Node](#light-node), [Consensus Client](#consensus-client), [Nimbus](#nimbus).

### Hash Collision {#hash-collision}
Two different inputs that produce the same hash output. While theoretically possible with cryptographic hash functions, finding meaningful data pairs that collide is computationally infeasible with SHA-256 or Keccak-256, making them secure for blockchain use.

### Hashing {#hashing}
**Master entry for cryptographic hash functions.** Process converting arbitrary-size input data into fixed-size hash output. Fundamental to blockchain security, data integrity, and privacy proofs.

**Hash Function Properties**:
- **Deterministic**: Same input always produces same output
- **One-way**: Computationally infeasible to reverse (find input from hash)
- **Collision-resistant**: Infeasible to find two different inputs with same output
- **Avalanche effect**: Tiny input change → completely different output

**Hash Functions by Use Case**:

| Hash | Output | Used In | Resistance | Notes |
|------|--------|---------|-----------|-------|
| **[Keccak-256](#keccak-256)** | 256 bits | Ethereum, Solidity | Strong | EVM native |
| **[SHA-256](#sha-256-secure-hash-algorithm)** | 256 bits | Bitcoin, proofs | Strong | Industry standard |
| **[SHA3-256](#sha3-256)** | 256 bits | Standards | Very strong | Official SHA3 |
| **[BLAKE2b](#blake2b)** | 512 bits | Zcash, Rust | Very strong | Fast, proven |
| **[BLAKE3](#blake3)** | 256 bits | Next-gen | Very strong | Fastest, modern |
| **[Poseidon](#poseidon)** | Variable | ZK proofs | Strong | Circuit-optimized |

**Blockchain Applications**:
- **Transaction hashing**: Hash(tx data) → tx ID
- **Block hashing**: Hash(block header) → block ID
- **Merkle trees**: Hash(data) + Hash(data) → Hash(parent) for proof structures
- **State roots**: Hash of entire state at block height
- **Commitments**: Hash(data) to commit without revealing data
- **Signatures**: Hash(message) signed with [ECDSA](#ecdsa-elliptic-curve-digital-signature-algorithm) or [Ed25519](#ed25519)

**ZK-Specific Hashes**:
- [Poseidon](#poseidon) - Optimized for constraint-based circuits
- MIMC, Rescue - Circuit-friendly alternatives
- Trade: Speed in circuits vs cryptographic margins

**Related**: [Merkle Tree](#merkle-tree), [Merkle Proof](#merkle-proof), [Zero-Knowledge Proof](#zero-knowledge-proof-zkp), [Signature](#signature-cryptographic), [ECDSA](#ecdsa-elliptic-curve-digital-signature-algorithm), [Ed25519](#ed25519).

### HD Wallet (Hierarchical Deterministic Wallet) {#hd-wallet-hierarchical-deterministic-wallet}
Wallet that derives multiple keypairs from a single seed phrase using a standardized derivation path. Enables deterministic, non-custodial address generation and recovery. Users back up one seed phrase to recover all derived addresses. BIP32/BIP39/BIP44 standards define the derivation hierarchy.

### Hooks {#hooks}
Extensibility points in AIMM lifecycle:
- PRE/POST_DEPOSIT
- PRE/POST_WITHDRAW
- PRE/POST_SWAP
- PRE/POST_FLASH_LOAN
- PRE/POST_DONATE

Controlled via bitmask flags per asset.

---

## I

### Impermanent Loss (IL) {#impermanent-loss-il}
Loss LPs experience when asset prices diverge from deposit ratio, compared to simply holding the assets. Called "impermanent" because if prices return to original ratio, the loss disappears. IL is a fundamental property of AMMs that continuously rebalance, arbitrage merely realizes the loss by trading against the pool. AIMM's oracle pricing and coverage-based depth reduce IL exposure.

### Internal Oracle {#internal-oracle}
AIMM's built-in price tracking system updated automatically on every swap. Uses dual-window TWAPs (5min/1hr) and volatility EMAs.

### Invariant {#invariant}
Mathematical constraint defining [AMM](#cfmm-constant-function-market-maker) price curves. **Invariant-based designs**:
- [CPMM](#cpmm-constant-product-market-maker): x × y = k (Uniswap V2)
- [CSMM](#csmm-constant-sum-market-maker): x + y = k (linear pricing)
- [CFMM](#cfmm-constant-function-market-maker) (general): F(x₁,...,xₙ) = k
- Stablecoin curves (e.g., Curve's StableSwap)

**Alternatives**: [CLMM](#clmm-concentrated-liquidity-market-maker) (tick-based ranges), [DLMM](#dlmm-dynamic-liquidity-market-maker) (bin-based), [CCMM](#ccmm-circularorbital-constant-market-maker) (n-dimensional). **Concentration variants**: [Quadratic Concentration](#quadratic-concentration) (Gyro 2-CLP: quadratic within range), [Elliptical Concentration](#elliptical-concentration) (Gyro E-CLP: elliptical with transformation params). AIMM breaks from invariant-based pricing entirely, using oracle-aware pricing instead. See [Price Impact](#price-impact), [Concentrated Liquidity](#concentrated-liquidity), and [Liquidity Range](#liquidity-range) for architectural variants.

### Index Price {#index-price}
See [Prices](#prices).

### Internal Transaction {#internal-transaction}
Smart contract-to-smart contract interaction recorded on-chain as a distinct event from a primary transaction. Triggered by function calls within contract execution. Visible on block explorers but uses the parent transaction's nonce and fee. Important for tracing execution flow.

### Inventory Skew {#inventory-skew}
Price adjustment based on [coverage ratio](#coverage-ratio) deviation, derived from the [Avellaneda-Stoikov Framework](#avellaneda-stoikov-framework). Applies a quadratic penalty/discount to incentivize rebalancing:
$$\begin{aligned} \text{skew} &\in [-100, +100] \text{ bps} \\ \text{skew} &= \operatorname{sign} \cdot 100 \cdot \left(\frac{\text{coverage} - \text{target}}{\text{bound} - \text{target}}\right)^{\gamma/10000} \end{aligned}$$

where:
- $\text{target}$ = 100% coverage (equilibrium)
- $\text{bound}$ = 50% (lower floor) or 200% (upper ceiling)
- $\gamma$ = gamma sensitivity parameter (default 10000 = 1.0x)
- $\text{sign}$ = positive if coverage < target (premium), negative if coverage > target (discount)
**Mechanism**: When undercollateralized (coverage < 100%), pool charges a premium to discourage further withdrawals. When overcollateralized (coverage > 100%), pool offers a discount to encourage users to sell assets back. **Economic interpretation**: Acts as a cost for inventory risk borne by remaining LPs. Higher [gamma](#gamma-γ) creates steeper penalty curves. **Relation to Avellaneda-Stoikov**: Replicates the academic model's dynamic spread adjustment as a function of inventory position. See [Coverage Ratio](#coverage-ratio), [Gamma (γ)](#gamma-γ), and `docs/1. AIMM/1.2. Pricing/1.2.5. Parametrization.md` for tuning guidance.

---

## J

### Jito {#jito}
Solana's dominant MEV infrastructure, providing a modified validator client and bundle auction system. **Architecture**: Jito validators accept transaction bundles from [searchers](#mev-searcher), auction ordering priority, and share MEV revenue with stakers. Unlike Ethereum's [Flashbots](#block-builder) (which operates at the builder layer), Jito integrates directly into the validator client because Solana has no separate builder role, the slot leader is both proposer and builder.

**Components**:
- **Jito-Solana**: Modified validator client that accepts bundles
- **Block Engine**: Auctions bundle placement within blocks
- **Relayer**: Routes bundles to the current slot leader
- **JitoSOL**: Liquid staking token sharing MEV revenue with stakers

**MEV on Solana**: Despite [Gulf Stream](#gulf-stream) eliminating public mempools, MEV persists because: (1) the slot leader controls final ordering, (2) Jito provides a mechanism for searchers to bid for favorable positions, and (3) some MEV (liquidations, arbitrage) doesn't require mempool observation.

**Free Public Endpoints**: Helius (Solana RPC provider) offers optional Jito integration for MEV protection on outbound transactions.

See [Gulf Stream](#gulf-stream), [MEV](#mev-maximal-extractable-value), [MEV Searcher](#mev-searcher), [Mempool](#mempool).

---

## K

### Knot (Spline) {#knot-spline}
Control point in liquidity profile defining price offset at specific depth level. Profiles have 8-16 knots with weights summing to 200.

### Keccak-256 {#keccak-256}
Cryptographic hash function standardized in Ethereum and used throughout EVM chains. Produces 256-bit hash. Ethereum uses Keccak-256 (not SHA-3) for hashing transactions, blocks, and state. Different from NIST SHA-3 standard.

---

## L

### Lighthouse {#lighthouse}
Ethereum [consensus client](#consensus-client) written in Rust by Sigma Prime. ~35% market share. Memory-efficient with built-in slasher for detecting validator misbehavior. Pairs with execution client via Engine API.

See [Consensus Client](#consensus-client) for comparison table.

### Lambda (λ) {#lambda}
Per-asset sensitivity parameter controlling how much additional spread is charged when oracle signals disagree (price uncertainty). **Tuning guidance**:
- **5000 (0.5x)**: Gentle response to price uncertainty (dampen volatility response)
- **10000 (1.0x)**: Linear response (default, balanced)
- **15000 (1.5x)**: Aggressive uncertainty penalty (volatile/trending assets)

**Formula**: `deviationSurcharge = maxSpread × λ / (100 × 10000)`

where `maxSpread = max(|fastOffset - slowOffset|)` measures disagreement between fast (5-min) and slow (1-hr) [TWAP](#twap-time-weighted-average-price).

**Economic role**: Lambda implements the **adverse selection cost** from [Avellaneda-Stoikov](#avellaneda-stoikov-framework) by charging more when price signals conflict, indicating market uncertainty. Higher lambda protects LPs during trending markets (when fast and slow prices diverge maximally) by widening spreads. **Relation to Markout Toxicity**: Lambda helps mitigate [markout toxicity](#markout-toxicity-winners-curse) by charging trending trades more. See [Deviation (Oracle)](#deviation-oracle), [Volatility (σ)](#volatility-σ), [Vega (ν)](#vega-ν), and `docs/1. AIMM/1.2. Pricing/1.2.5. Parametrization.md` for tuning.

### LCA (Least Common Ancestor) {#lca-least-common-ancestor}
Algorithm finding shortest path between two nodes in anchor tree. Used for multi-hop swap routing.

### Light Node {#light-node}
See [Node Types](#node-types).

### Liabilities {#liabilities}
LP claims against pool reserves. Tracks what the pool owes depositors:
```solidity
struct Asset {
    uint128 reserves;      // What pool holds
    uint128 liabilities;   // What pool owes
}
```

### Liquidity Index {#liquidity-index}
Accrued interest multiplier for yield-bearing assets. Updated on interactions.

### Limit Order {#limit-order}
See [Order Types](#order-types) → [Limit Order](#limit-order).

### Liquidity Provider (LP) {#liquidity-provider-lp}
Agent that supplies executable quotes or inventory to a market, either through limit orders in a [CLOB](#clob-central-limit-order-book) or deposits into an [AMM](#cfmm-constant-function-market-maker) pool. Earns the spread and any fees minus [adverse selection](#adverse-selection) costs and rebalancing costs. **LP risks**: [Impermanent Loss](#impermanent-loss-il), [LVR](#lvr-loss-versus-rebalancing), extraction via [Order Flow Toxicity](#order-flow-toxicity) ([Statistical Arbitrage](#statistical-arbitrage-stat-arb), [JIT](#jit-just-in-time-liquidity), [Informed Order Flow](#informed-order-flow-alpha-toxicity), [Markout Toxicity](#markout-toxicity-winners-curse)). **Related**: [Market Maker](#market-maker) (CLOB equivalent), [Liquidity Taker](#liquidity-taker), [Maker-Taker Fees](#maker-taker-fees), [Price Discovery](#price-discovery). See [Arbitrage](#arbitrage) for extraction taxonomy.

### Liquidity Taker {#liquidity-taker}
Trader who executes immediately against resting liquidity by hitting bids or lifting offers in a CLOB, or by swapping against an AMM pool. Pays the spread and market impact costs.

### Liquidity Profile {#liquidity-profile}
Pool-level spline-based depth curve defining price impact and capital concentration across price ranges. AIMM's architecture:
```solidity
struct LiquidityProfile {
    uint8[16] weights;     // Segment weights (sum = 200)
    int8[17] knots;        // Price offset knots (weights.length + 1 points)
}
```
**Related architectures**: [Concentrated Liquidity](#concentrated-liquidity) (LP chooses range), [Liquidity Shaping](#liquidity-shaping) (dynamic adjustments), [Bonding Curve](#bonding-curve) (price discovery). Contrasts with [CLMM](#clmm-concentrated-liquidity-market-maker) (tick-based ranges) and [DLMM](#dlmm-dynamic-liquidity-market-maker) / [Liquidity Book](#liquidity-book) (bin-based segments). See [Invariant](#invariant) and [Price Impact](#price-impact) for curve mechanics.

### Liquidity Shaping {#liquidity-shaping}
Dynamic adjustment of [liquidity profile](#liquidity-profile) weights and offsets in response to market conditions, volatility, or trading activity. Enables pool operators or automated systems to concentrate liquidity where trading occurs most, improving capital efficiency and fee capture. Unlike static [bonding curves](#bonding-curve), liquidity shaping allows real-time rebalancing without modifying the [invariant](#invariant). Related to [Liquidity Bootstrapping Pool (LBP)](#liquidity-bootstrapping-pool-lbp) (time-weighted shaping) and [DLMM](#dlmm-dynamic-liquidity-market-maker) (bin-level dynamic fees).

### Bonding Curve {#bonding-curve}
Mathematical function defining the relationship between token supply and price. Traditional AMMs use fixed bonding curves (x·y=k for CPMMs, StableSwap invariant for Curve). AIMM replaces fixed invariant curves with spline-based [liquidity profiles](#liquidity-profile) that have dynamic [dispersion](#dispersion)-the curve shape responds to volatility rather than being static.

**Traditional bonding curves**: [CPMM](#cpmm-constant-product-market-maker) (hyperbolic x·y=k), [CSMM](#csmm-constant-sum-market-maker) (linear), StableSwap (concentrated around peg).

**AIMM approach**: Liquidity profiles + volatility-scaled dispersion replace bonding curves. See [Dispersion](#dispersion) for how curve width responds to market conditions.

### Liquidity Bootstrapping Pool (LBP) {#liquidity-bootstrapping-pool-lbp}
Pool design for fair token price discovery with time-weighted liquidity shaping. Early in launch, pool weights are skewed toward the existing asset (e.g., 95% USDC / 5% new token), making new tokens cheaper. Over time, weights shift (time-decreasing LBP) to equalize (50/50), creating a smooth price discovery process. Prevents whales from accumulating all tokens at launch price. **Mechanism**: [Bonding Curve](#bonding-curve) implicit in weight trajectory. **Related**: [Liquidity Shaping](#liquidity-shaping), [Price Discovery](#price-discovery), [Concentrated Liquidity](#concentrated-liquidity).

### Liquidity Range {#liquidity-range}
Price interval within which an LP allocates concentrated liquidity in [concentrated liquidity](#concentrated-liquidity) pools. **In tick-based pools**: Range = set of ticks; within range, acts as local [CPMM](#cpmm-constant-product-market-maker) between endpoints (continuous). **In bin-based pools**: Range = set of bins; each bin independently acts as [CSMM](#csmm-constant-sum-market-maker) (discrete). **Key trade-offs**: Tighter range = higher capital leverage but more rebalancing required and greater out-of-range risk. Wider range = lower leverage but less rebalancing. See [Concentrated Liquidity](#concentrated-liquidity) for pool type comparison and [Impermanent Loss](#impermanent-loss-il) for risk context. **Implementations**: Uniswap V3 ([CLMM](#clmm-concentrated-liquidity-market-maker)), Trader Joe V2 ([DLMM](#dlmm-dynamic-liquidity-market-maker)).

### Liquidity Book {#liquidity-book}
[DLMM](#dlmm-dynamic-liquidity-market-maker) architecture using discrete price bins rather than continuous ticks. Each bin acts as a local [CSMM](#csmm-constant-sum-market-maker) with near-zero slippage inside the bin. LP positions are **fungible per bin** (users hold position tokens per bin, not per individual tick). **Distinction from CLMM**: In [CLMM](#clmm-concentrated-liquidity-market-maker), ranges of N ticks act as [CPMM](#cpmm-constant-product-market-maker) between endpoints (continuous). In Liquidity Book, each bin acts independently as CSMM (discrete). Dynamic fee adjustments respond to volatility and bin utilization. See [Liquidity Range](#liquidity-range) for CLMM-DLMM comparison and [DLMM](#dlmm-dynamic-liquidity-market-maker) for architecture details. **Implementations**: Trader Joe V2 (Avalanche), Meteora (Solana), Cetus (Sui/Aptos), Saros (Solana).

### LP Token {#lp-token}
Fungible claim on deposited liquidity. AIMM tracks per-user, per-asset LP balances rather than pool-wide shares.

### LVR (Loss-Versus-Rebalancing) {#lvr-loss-versus-rebalancing}
The difference between LP returns and a theoretical portfolio that rebalances continuously at true market prices. Unlike IL, LVR is **permanent**-it occurs because AMMs trade at stale prices between oracle updates. Arbitrageurs exploit this lag: when external price rises, they buy cheap from the pool; when it falls, they sell dear. Even if prices revert, the pool lost on both legs. Research shows 5-7% annual LP capital loss on major AMMs. AIMM addresses LVR via oracle-first pricing (reducing staleness) and directional fees (charging for toxic flow).

---

## M

### Market Making {#market-making}
Providing liquidity by continuously posting firm bid and ask quotes, standing ready to buy and sell. Market makers earn the bid-ask spread plus any venue rebates. Market making can be manual (CLOB) or automated (AMM).

### Market Maker {#market-maker}
Participant that continuously posts firm bid and ask quotes, standing ready to buy and sell, earning the bid-ask spread and any rebates. Bears inventory risk and adverse selection costs. Foundation of market liquidity.

### Market Order {#market-order}
See [Order Types](#order-types) → [Market Order](#market-order).

### Market Depth {#market-depth}
Measure of market's ability to absorb large buy or sell orders without significantly affecting price. Represented visually by the order book, deeper markets can handle larger orders with less impact. Critical metric for evaluating trading venue liquidity. Also called order book depth.

### Maker-Taker Fees {#maker-taker-fees}
Fee schedule where liquidity providers (makers) receive rebates or pay lower fees, while liquidity takers pay higher fees per executed trade. Incentivizes passive liquidity provision over aggressive trading. Standard model on most CEXes and some DEXes.

### Moving Average {#moving-average}
**Master entry for time-series smoothing techniques.** Methods calculating averages over rolling time windows to reduce noise and identify trends. Different weighting schemes serve different purposes in trend detection and volatility estimation.

**Types**:

| Type | Weighting | Response | AIMM Use |
|------|-----------|----------|----------|
| **[SMA](#sma-simple-moving-average)** | Equal | Slow | TWAP gives SMA mathematically |
| **[EMA](#ema-exponential-moving-average)** | Exponential decay | Fast | Volatility tracking (dual windows) |
| **[LWMA](#lwma-linear-weighted-moving-average)** | Linear decay | Medium | Alternative to EMA |
| **[TWAP](#twap-time-weighted-average-price)** | Time-weighted | Window-dependent | Internal oracle pricing |

**AIMM implementation**: EMA for volatility (prioritizes recent spikes), TWAP for oracle pricing (manipulation-resistant).

**See**: [Volatility](#volatility-σ), [Internal Oracle](#internal-oracle).

#### EMA (Exponential Moving Average)
Weighted average prioritizing recent data via exponential decay.

**Formula**:
```
EMA_new = EMA_old + α × (value - EMA_old)
where α = smoothing factor
```

**AIMM implementation**: Dual EMAs for volatility:
- **Fast EMA** (5-min): Captures short-term spikes
- **Slow EMA** (1-hr): Baseline estimate
- **Breadth**: max(fast, slow) → responsive to sudden changes

**See**: [Moving Average](#moving-average), [Volatility](#volatility-σ).

#### LWMA (Linear Weighted Moving Average)
Linearly weighted average with recent data prioritized. Weights decrease linearly going back in time.

**Properties**: Balanced responsiveness (between SMA and EMA), predictable decay, more expensive than EMA.

**AIMM context**: Not currently used.

**See**: [Moving Average](#moving-average), [EMA](#ema-exponential-moving-average).

#### SMA (Simple Moving Average)
Arithmetic mean over fixed window, all data points weighted equally.

**Formula**: `SMA = (P₁ + P₂ + ... + Pₙ) / n`

**Properties**: Smoothest, slowest to react, no decay bias.

**AIMM context**: TWAP accumulator produces SMA behavior mathematically.

**See**: [Moving Average](#moving-average), [TWAP](#twap-time-weighted-average-price).

#### TWAP (Time-Weighted Average Price)
Accumulator-based average price over time window. Mathematically equivalent to SMA but manipulation-resistant.

**Formula**: `TWAP = Σ(Price_i × Duration_i) / Total_Duration`

**AIMM implementation**:
- Maintains cumulative `price × time` accumulator
- Updates on every swap: accumulates `price × Δt`
- Query: `(accumulator_now - accumulator_past) / time_elapsed`

**Properties**: Manipulation-resistant (attacker must sustain price over time), gas-efficient, smooth.

**See**: [Moving Average](#moving-average), [Internal Oracle](#internal-oracle).

### MEV (Maximal/Miner Extractable Value) {#mev-maximal-miner-extractable-value}
The total profit a block producer can capture by strategically controlling transaction ordering, inclusion, or suppression within a block. MEV arises because transaction order confers informational advantage, actors can profit by reordering pending transactions. Different strategies carry different externalities: sandwich attacks harm users, frontrunning exploits information, and backrunning captures benign arbitrage. Encompasses multiple extraction types: [Frontrunning](#frontrunning) (harmful), [Sandwich Attacks](#sandwich-attack) (most harmful), [Backrunning](#backrun) (benign), and [MEV Spoofing](#mev-spoofing) (encrypted-mempool specific). Related to [Arbitrage](#arbitrage) taxonomy but focused on block-ordering control rather than market-structure exploitation. Measured as percentage of total block value and a significant source of LP losses in AMMs. See [MEV Searcher](#mev-searcher) for execution agents.

### MEV Strategies {#mev-strategies}

#### Front-running
[MEV](#mev-maximal-extractable-value) extraction where an attacker observes a high-value pending transaction and places their own transaction ahead of it. **Information-based extraction** (uses private information about mempool). Common on DEXes where buying an asset first increases slippage on the victim's swap. Extracts value from the victim's transaction impact at the victim's expense. Economically related to [Informed Order Flow](#informed-order-flow-alpha-toxicity) but operates at block-ordering level. See [Arbitrage](#arbitrage) for the broader extraction taxonomy.

#### Back-running
A [MEV](#mev-maximal-extractable-value) extraction strategy where a transaction is strategically placed after an existing transaction to capture arbitrage opportunities generated by price movements from that transaction. **Benign form of extraction** (no harm to the preceding transaction). Unlike [sandwich attacks](#sandwich-attack), backrunning simply capitalizes on market inefficiencies created by legitimate trading activity, it's economically equivalent to [Statistical Arbitrage](#statistical-arbitrage-stat-arb). Common in liquidation capture and DEX arbitrage. See [Arbitrage](#arbitrage) for the toxicity spectrum.

#### Sandwich Attack
Coordinated [MEV](#mev-maximal-extractable-value) strategy involving transactions placed both before and after a victim's transaction. The attacker first moves the market to their advantage (buying an asset to raise its price), then the victim's transaction executes at worse price, then the attacker captures the spread by reversing position. **Most damaging MEV strategy for DEX users** (harmful to victims). Combines [frontrunning](#frontrunning) and [backrunning](#backrun) but with intentional adverse market movement. See [Arbitrage](#arbitrage) and [Order Flow Toxicity](#order-flow-toxicity) for extraction context.

#### JIT (Just-In-Time) Liquidity
Strategy where sophisticated actors atomically add liquidity before a large swap and remove it immediately after, capturing fees without bearing price risk. **Fee-extraction form of [toxic flow](#order-flow-toxicity)** (harms LPs by frontrunning their inventory). Research shows JIT accounts for only ~1% of Uniswap v3/v4 volume, dominated by 1-20 professional MEV bots only. Distinct from [Statistical Arbitrage](#statistical-arbitrage-stat-arb) in that it exploits fee timing rather than price discrepancies. See [Arbitrage](#arbitrage) for extraction taxonomy. AIMM's [Flow Guard](#flow-guard) mechanism blocks this attack vector by enforcing cooldowns on specific call flows: deposit->withdrawal and stake->unstake.

#### MEV Spoofing
Speculative attack that attempts to extract [MEV](#mev-maximal-extractable-value) from [encrypted mempools](#encrypted-mempool) without knowing transaction content. The attacker observes encrypted transaction metadata (size, target address) and speculatively submits their own encrypted transaction hoping to land in a favorable position. **Information-incomplete extraction** (requires threshold encryption breaking). If the position is unfavorable, the attacker withholds their decryption key share, preventing transaction execution and effectively "spoofing" the system by consuming block space without committing. Distinct from other MEV strategies in that it trades on metadata rather than full transaction visibility. See [Arbitrage](#arbitrage) for extraction taxonomy. Mitigated by penalties for non-decryption and advanced protocol designs combining encryption with transaction permutation or threshold schemes.

#### Greasing
MEV extraction where actors submit transactions that don't execute but consume blockspace and fees to influence market conditions or extract value from the block's structure. Related to [MEV Spoofing](#mev-spoofing) in that transactions may be submitted speculatively without full commitment to execution. Value extraction comes from block construction itself rather than transaction content.

**Aliases**: Greasing, block-stuffing (when used defensively).

**Related**: [MEV](#mev-maximal-extractable-value) (block-ordering exploitation), [Adverse Selection](#adverse-selection) (information asymmetry), [Order Flow Toxicity](#order-flow-toxicity) (umbrella term), [MEV Strategies](#mev-strategies) (block-level extraction methods).

**AIMM Mitigation**: Oracle pricing reduces staleness → less stat arb; directional fees charge toxic flow more; [Flow Guard](#flow-guard) blocks JIT; deviation surcharge on uncertainty penalizes informed flow.

### Mempool {#mempool}
Buffer of pending transactions waiting to be included in a block. Architecture varies significantly by chain:

**Public Mempool** (Ethereum L1, BNB Chain): Transactions are gossiped across all nodes in the network. Anyone running a node can observe pending transactions, enabling classic [MEV](#mev-maximal-extractable-value) via public observation of orderflow. [Block builders](#block-builder) and [searchers](#mev-searcher) monitor the mempool to identify profitable reordering opportunities.

**Private Mempool / Sequencer Queue** (Arbitrum, Base, Optimism, most OP Stack rollups): Users send transactions directly to a centralized [sequencer](#sequencer), which immediately orders them without public gossip. Reduces third-party MEV but concentrates ordering power in the sequencer. The sequencer still sees all pending transactions and can extract value if malicious.

**Leader-Based Forwarding** (Solana via [Gulf Stream](#gulf-stream)): Transactions are forwarded directly to the current and upcoming slot leaders rather than gossiped globally. No traditional queryable mempool, but the leader (and attached infrastructure like [Jito](#jito)) controls ordering and can extract MEV.

**DAG-Based Mempool** (Sui via [Narwhal](#narwhal)): Transactions are gossiped into a Directed Acyclic Graph structure before consensus, with MEV concentrated around "shared object" contention rather than traditional front-running.

See [Sequencer](#sequencer), [Block Builder](#block-builder), [MEV](#mev-maximal-extractable-value), [Encrypted Mempool](#encrypted-mempool), [MEV-Protected RPC](#mev-protected-rpc).

### MEV-Protected RPC {#mev-protected-rpc}
[RPC](#rpc-remote-procedure-call) endpoint that hides user transactions from the public [mempool](#mempool) to prevent [front-running](#frontrunning) and [sandwich attacks](#sandwich-attack). Transactions are sent directly to trusted [block builders](#block-builder) or private relays that commit to fair ordering.

**How it works**: Instead of broadcasting to the public mempool, the RPC sends transactions to a private relay or builder that either: (1) includes transactions without reordering against them, (2) auctions MEV opportunities to searchers but returns extracted value to users, or (3) uses [threshold encryption](#threshold-encryption) to hide transaction content until ordering is finalized.

**Free Public Endpoints** (Ethereum L1):
- MEV Blocker: `https://rpc.mevblocker.io`
- Flashbots Protect: `https://rpc.flashbots.net/fast`
- Merkle: `https://eth.merkle.io`
- Llama Nodes: `https://eth.llamarpc.com`
- BlockRazor: `https://eth.blockrazor.xyz`

**Free Public Endpoints** (BNB Chain):
- 48 Club: `https://rpc.48.club`
- Merkle: `https://bsc.merkle.io`
- Llama Nodes: `https://binance.llamarpc.com`
- BlockRazor: `https://bsc.blockrazor.xyz`

**Free Public Endpoints** (Base):
- Llama Nodes: `https://base.llamarpc.com`

**Private RPC Networks** (multi-chain, require account):
- **Alchemy**: MEV protection always-on for all supported chains
- **BlockPI Network**: Optional MEV protection (enable in dashboard)
- **dRPC**: Optional MEV protection (enable in dashboard)
- **Helius** (Solana): Optional Jito integration for MEV protection

**Research & Further Reading**:
- [Private MEV Protection RPCs (CoW DAO)](https://arxiv.org/pdf/2505.19708v1) - Analysis of private RPC effectiveness
- [MEV on Polygon](https://arxiv.org/pdf/2508.21473v1) - Impact study
- [Cross-chain Sandwich Attacks](https://arxiv.org/pdf/2511.15245v1) - Multi-chain MEV vectors
- [UniswapX Price Improvement](https://blog.uniswap.org/UniswapX_PI.pdf) - Order flow auction design
- [Protected Order Flow Relay](https://arxiv.org/pdf/2408.02303) - Relay architecture

See [Slippage](#slippage), [MEV](#mev-maximal-extractable-value), [Mempool](#mempool), [Sequencer](#sequencer).

### Multiparty Computation (MPC) {#multiparty-computation-mpc}
Cryptographic technique enabling multiple parties to jointly compute a function over private inputs without revealing those inputs to each other. In blockchain contexts, used for threshold cryptography (requiring k-of-n parties to cooperate), distributed key management, and some MEV-resistant protocols for transaction encryption.

### Merkle Tree {#merkle-tree}
Cryptographic data structure for efficient verification. Used by Distributor for reward claims.

### Merkle Patricia Trie {#merkle-patricia-trie}
Data structure combining Merkle tree and Patricia trie properties, used by Ethereum to store state (accounts, balances, contract storage). Enables efficient updates and cryptographic verification of any historical state without storing the entire tree.

### Merkle Proof {#merkle-proof}
Cryptographic proof demonstrating that a specific data element exists in a Merkle tree without revealing the entire tree. Requires only logarithmic hash values. Used in privacy protocols to prove membership (e.g., proving funds were deposited) without revealing which specific element (which user/transaction) the prover represents.

### Mixer {#mixer}
Privacy protocol that pools funds from many users and mixes them, breaking the link between origin and destination addresses. Users deposit funds and later withdraw to different addresses. Uses zero-knowledge proofs and Merkle trees to prove a valid deposit existed without revealing which specific deposit was theirs. Enables transaction privacy on public blockchains.

### Mid-Price {#mid-price}
See [Prices](#prices).

### Mark Price {#mark-price}
See [Prices](#prices).

### Market Making & DeFi Specific Metrics {#market-making-defi-specific-metrics}
**Master entry for liquidity provision and DeFi-specific performance metrics.** Measures pool utilization, liquidity depth, and capital efficiency unique to decentralized trading infrastructure.

| Metric | Definition | Use Case | AIMM Application |
|--------|-----------|----------|------------------|
| **[Bid-Ask Spread](#spread)** | Difference between highest bid and lowest ask | Transaction cost for traders | Dynamic spread based on [volatility](#volatility-σ) and [coverage ratio](#coverage-ratio) |
| **[Loss vs Rebalancing (LVR)](#lvr-loss-versus-rebalancing)** | LP loss from rebalancing due to price moves | Quantify passive LP costs | Alternative IL metric capturing realized losses |
| **[Total Value Locked (TVL)](#total-value-locked-tvl)** | Total dollar value of assets in protocol | Protocol size and capital attraction | Pool size determines depth and execution quality |
| **[Utilization Ratio](#utilization-ratio)** | Percentage of liquidity actually used | Capital efficiency and depth adequacy | Higher utilization = better capital use but higher price impact |
| **[Volume](#volume)** | Total traded notional value in period | Liquidity activity and fee generation | More volume = more fee revenue for LPs |

**Key Insights**:
- **Spread narrowness**: Tight spreads (lower transaction costs) require deep liquidity
- **LVR vs IL**: LVR better captures actual realized losses during active trading
- **TVL size matters**: Larger pools handle larger orders with less impact
- **Utilization sweet spot**: 100% utilization means order reaches pool boundaries (high impact); 0% means wasted capital
- **Volume predictability**: Patterns help forecast fee revenue and optimal capital deployment

**Related**: [Volatility](#volatility-σ), [Price Impact](#price-impact), [Impermanent Loss](#impermanent-loss-il), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics).

### LVR (Loss Versus Rebalancing) {#lvr-loss-versus-rebalancing}
**Quantifies the loss an LP experiences from passively holding a position as prices move, compared to continuously rebalancing.** Alternative to [impermanent loss](#impermanent-loss-il) that captures realized losses from price movement.

**Difference from IL**:
- **IL**: Compares pool value to "if LP held tokens separately" at current prices
- **LVR**: Compares LP's actual payoff to a continuously-rebalanced strategy (never holds pool position)

**Calculation**:
```
LVR = Value_RebalancedStrategy - Value_HoldingLiquidityPosition
Positive LVR = loss (LP would be better off rebalancing)
```

**Example**:
- LP deposits $10k (5 ETH @ $2k + 5 USDC equivalent)
- ETH rises to $3k; BTC not provided by this pool
- Rebalanced strategy: Constantly sell ETH as it rises → earns $5k profit
- Passive LP position: Gets squeezed, ends with ~6.15 ETH + 0 USDC ≈ $18.45k total (less than rebalancing)
- LVR ≈ $1.55k loss

**AIMM context**: Oracle pricing and [coverage ratio](#coverage-ratio)-based spreads reduce LVR by earning fees that offset the loss from price divergence.

See [Impermanent Loss](#impermanent-loss-il), [Market Making & DeFi Specific Metrics](#market-making--defi-specific-metrics).

### Total Value Locked (TVL) {#total-value-locked-tvl}
**Total dollar value of all assets deposited into a protocol.** Primary metric for measuring protocol size, capital attraction, and liquidity depth.

**Calculation**:
```
TVL = Σ(each_asset_balance × oracle_price)
```

**Interpretation**:
- **Large TVL** ($100M+): Sufficient liquidity for most trades, tight spreads possible
- **Small TVL** ($1M): Limited liquidity, wide spreads, high price impact for medium trades
- **Growing TVL**: Indicates user confidence and capital inflow
- **Declining TVL**: Risk signal (users withdrawing due to low returns, governance issues, or security concerns)

**AIMM context**: Larger TVL means deeper liquidity and lower [price impact](#price-impact). Pool operators use TVL forecasts to plan capital requirements and spread competitiveness.

**Related**: [Utilization Ratio](#utilization-ratio), [Volume](#volume), [Liquidity Depth](#liquidity-depth).

### Utilization Ratio {#utilization-ratio}
**Measure of capital efficiency across DeFi primitives.** Quantifies how much deposited capital is actively deployed versus sitting idle. Definition varies by protocol type based on how capital generates yield.

**Common DeFi Formulations**:

| Protocol Type | Utilization Formula | Interpretation |
|---------------|---------------------|----------------|
| **Money Markets** | Borrowed / Total Supplied | % of deposits actively lent out |
| **Lending Protocols** | Utilized / Total Available | % of capital earning interest |
| **AMMs / Pools** | Trading Volume / TVL | Capital turnover efficiency |
| **Derivatives** | Open Interest / Collateral | % of collateral backing positions |
| **Staking** | Staked / Circulating Supply | % of tokens locked |

**General interpretation**:
- **High utilization**: Capital efficiently deployed, earning maximum yield
- **Medium utilization**: Balanced between earning and liquidity buffer
- **Low utilization**: Excess idle capital, wasted opportunity cost

**AIMM-Specific Definition**:
```
Utilization = (Liquidity_ActivelyTraded / Total_Available_Liquidity) × 100%
```

**AIMM context**: Liquidity shaping adjusts weights to increase utilization near expected trading zones. Higher utilization = more fee generation per deployed capital.

**Interpretation (AIMM)**:
- **High utilization (>80%)**: Liquidity concentrated near market price; good capital efficiency but orders hitting boundaries (high impact)
- **Medium utilization (40-80%)**: Balanced capital deployment; most trader needs met without wasteful depth
- **Low utilization (<20%)**: Too much liquidity at extreme prices; wasted capital not earning fees

**Trade-off**: Tight utilization reduces [price impact](#price-impact) for typical orders but leaves no buffer for large ones. Applies across DeFi: money markets balance utilization against withdrawal liquidity, AMMs balance between capital efficiency and slippage protection.

See [Liquidity Depth](#liquidity-depth), [Liquidity Shaping](#liquidity-shaping), [Total Value Locked](#total-value-locked-tvl).

### Volume {#volume}
**Total notional value of assets traded in a period (day, week, month).** Primary revenue driver for LP fees.

**Calculation**:
```
Daily_Volume = Σ(each_swap_amountIn × priceAtSwap)
```

**Relationship to fees**:
```
Daily_Fee_Revenue = Daily_Volume × Fee_Rate
Example: $10M volume × 0.05% fee = $5k daily revenue
```

**High-volume assets**: Generate substantial LP fee revenue, compensating for [IL](#impermanent-loss-il).

**Low-volume assets**: LPs earn minimal fees, making IL drag more significant.

**AIMM context**: Forecast expected LP yields based on historical volume patterns. Volume seasonality (weekday vs weekend, US vs Asian hours) affects spread strategy and coverage ratio targets.

**Related**: [Spread](#spread), [Utilization Ratio](#utilization-ratio), [Total Value Locked](#total-value-locked-tvl).

### Modular Blockchain {#modular-blockchain}
Blockchain architecture where consensus, data availability, and settlement are handled by separate layers or protocols. Enables horizontal scaling by specializing components. Contrasts with monolithic blockchains that handle all functions within a single layer.

### Monolithic Blockchain {#monolithic-blockchain}
Blockchain architecture where all functions, consensus, transaction execution, and data availability, are handled by a single protocol layer. Examples: Bitcoin, original Ethereum. More coupled but simpler to reason about; scaling requires vertical optimization.

### Makima Spline {#makima-spline}
See [Spline (Cubic Interpolation)](#spline-cubic-interpolation).

### Monotone Cubic Hermite Interpolation {#monotone-cubic-hermite-interpolation}
See [Spline (Cubic Interpolation)](#spline-cubic-interpolation).

### MULT_BASE {#mult-base}
Multiplier precision constant = 10,000. Used for sensitivity parameters (gamma, vega, lambda).

---

## N

### Native Token {#native-token}
Chain's native currency (ETH on Ethereum). Represented by sentinel address `0xEeee...EEeE` (EIP-7528). Automatically wrapped/unwrapped by pool.

### Nethermind {#nethermind}
Ethereum [execution client](#execution-client) written in C#/.NET. ~20% market share. Enterprise-focused with MEV-Boost integration and extensive JSON-RPC extensions.

See [Execution Client](#execution-client) for comparison table.

### Nimbus {#nimbus}
Ethereum client in Nim providing both [consensus](#consensus-client) and [light client](#light-node) functionality. ~5% market share. Ultra-lightweight, suitable for embedded devices and mobile.

See [Consensus Client](#consensus-client) for comparison table and [Light Node](#light-node).

### Node Types {#node-types}
**Master entry for blockchain network participants.** Nodes maintain and validate blockchain state, differing in data retention, resource requirements, and capabilities.

| Type | Storage | Verification | Resources | Use Case |
|------|---------|--------------|-----------|----------|
| **[Full Node](#full-node)** | Current state + recent history | Independent | Medium | RPC providers, validators |
| **[Archive Node](#archive-node)** | All historical states | Independent | High | Block explorers, analytics |
| **[Light Node](#light-node)** | Headers + selective proofs | Trust majority | Low | Wallets, mobile apps |
| **[Validator Node](#validator-node)** | Full state + consensus duties | Independent | Medium-High | Block production, staking |

See [Node Operator](#node-operator), [Execution Client](#execution-client), [Consensus Client](#consensus-client).

#### Full Node
Complete blockchain node that downloads, validates, and stores current blockchain state and transaction history. Full nodes independently verify all transactions and blocks according to consensus rules. Require significant storage but provide strong security guarantees and can serve RPC requests.

**Aliases**: Full node, full validator.

See [Archive Node](#archive-node), [Light Node](#light-node), [Validator Node](#validator-node).

#### Archive Node
Full node maintaining complete history of all ledger states at every block height. Unlike standard full nodes that prune old data, archive nodes retain all historical state changes, enabling queries of past balances and contract states. Essential for block explorers and analytics services.

**Storage requirements**: Multi-TB (Ethereum mainnet: ~12TB+).

See [Full Node](#full-node).

#### Light Node
Lightweight client downloading only block headers and selective proofs rather than full blockchain state. Light nodes trust validator majority to follow consensus rules honestly and use cryptographic proofs to verify specific data without storing everything. Consume minimal resources, suitable for wallets and mobile applications.

**Examples**: [Helios](#helios) (Ethereum), [Tinydancer](#tinydancer) (Solana).

See [Full Node](#full-node), [Helios](#helios), [Tinydancer](#tinydancer).

#### Validator Node
Network participant staking cryptocurrency as collateral to participate in consensus and block production. Validators earn rewards for honest participation but face penalties (slashing) for misbehavior. Runs full node plus consensus client for block proposal and attestation duties.

See [Validator](#validator), [Full Node](#full-node), [Consensus Client](#consensus-client), [Slashing](#slashing).

### Node Operator {#node-operator}
Individual or entity running validator software to secure a Proof of Stake network. Node operators earn staking rewards for honest participation but face penalties (slashing) for misbehavior. In staking pools, node operators may also contribute capital alongside delegators.

See [Node Types](#node-types), [Validator](#validator).

### Nonce {#nonce}
Number used once, in blockchain context, a transaction counter incremented with each transaction from an account to prevent replay attacks and ensure ordering. Also used in smart contracts (e.g., randomness) and as a proof-of-work parameter in mining.

### Nullifier {#nullifier}
Special hash of a secret commitment, sent with withdrawal requests in privacy protocols. Protocol checks that the nullifier has never been used before, preventing double-spending of the same commitment. Without nullifiers, users could withdraw multiple times from the same deposit. Critical security mechanism in mixers and private transaction systems.

### Order Book Depth {#order-book-depth}
Distribution of resting volume across price levels on both bid and ask sides, often summarized as a depth profile or ladder. Deeper order books indicate stronger liquidity and lower market impact per unit traded. Key measure of market microstructure health.

### Order Flow Toxicity {#order-flow-toxicity}
Measure of how informative or adverse trade flow is to liquidity providers. **Umbrella term covering all types of extraction** ([Statistical Arbitrage](#statistical-arbitrage-stat-arb), [Latency Arbitrage](#latency-arbitrage), [Informed Order Flow](#informed-order-flow-alpha-toxicity), [Markout Toxicity](#markout-toxicity-winners-curse), [Flow Internalization](#flow-internalization-pfof---payment-for-order-flow)). Toxic flow indicates informed trading, speed advantages, or flow composition biases that extract value from market makers. Often proxied by metrics like VPIN or short-term price reversion patterns. See [Arbitrage](#arbitrage) for the toxicity hierarchy. Low toxicity favors passive liquidity provision.

---

## O

### Oracle {#oracle}
**Master entry for price feed sources.** Oracles provide fair value estimates to smart contracts and trading systems. AIMM uses oracle prices to compute [inventory skew](#inventory-skew) and reduce staleness-based [arbitrage](#arbitrage).

**Oracle Types**:

| Type | Source | Update | Latency | Trust | Use |
|------|--------|--------|---------|-------|-----|
| **[Internal Oracle](#internal-oracle)** | Pool swaps | Automatic | Very low | Permissionless | AIMM primary |
| **[External Push](#external-oracle)** | Chainlink, Pyth | Push-based | Variable | High (audited) | Fallback, L2s |
| **[Custom Feed](#feed-oracle)** | Backend | Manual | Custom | Project-specific | Specialized |

**Oracle Mechanism**:
1. **Data sources**: CEX prices, on-chain activity, off-chain compute
2. **Aggregation**: Multiple sources → consensus value (median, TWAP, etc.)
3. **Signing**: Oracle provider signs feed (enables verification)
4. **Update frequency**: Fast (1-5 min) for AIMM, slower for settlement

**AIMM's Internal Oracle**:
- **Fast TWAP**: Short-window average (e.g., 5 min) - responsive to market moves
- **Slow TWAP**: Long-window average (e.g., 1 hour) - filters noise
- **Agreement**: Deviation between fast/slow → increases [spread](#spread)

**Related Concepts**:
- [Deviation](#deviation-oracle) - Disagreement between oracle sources
- [Feed](#feed-oracle) - Oracle data structure + metadata
- [Threshold Encryption](#threshold-encryption) - MEV-resistant oracle delivery
- [Fair Value](#fair-value) - Oracle-derived "true" price
- [Price Discovery](#price-discovery) - How markets discover fair value

See [Arbitrage](#arbitrage), [Liquidity Shaping](#liquidity-shaping), [Spread](#spread) for AIMM applications.

### Order Types {#order-types}
**Master entry for trading order instructions.** Order types specify execution conditions, price constraints, and timing rules for trade execution in both CLOBs and AMMs.

**Basic Order Categories**:

| Category | Price Control | Execution | Use Case |
|----------|---------------|-----------|----------|
| **[Market](#market-order)** | None (current price) | Immediate | Fast execution, high certainty |
| **[Limit](#limit-order)** | Exact price or better | Conditional | Price control, queue position |
| **[Stop](#stop-orders)** | Trigger-activated | Conditional | Risk management, automation |

**Advanced Order Types**:

| Type | Description | Execution | Use Case |
|------|-------------|-----------|----------|
| **[Iceberg](#iceberg-order)** | Partial visibility order | Sequential chunks | Large trades, avoid impact |
| **[OCO](#oco-one-cancels-other)** | Paired conditional orders | One executes → other cancels | Bracket orders, dual scenarios |

See [Time-In-Force](#time-in-force-tif) for duration specifications (FOK, IOC, AON, GTC, Day Order), [Market Order](#market-order), [Limit Order](#limit-order), [Stop Orders](#stop-orders).

#### Market Order
Order to buy or sell immediately at the best available current market price. Prioritizes execution speed over price certainty. Fills at current market rate, which may differ from expected price due to [slippage](#slippage).

**Trade-offs**: High execution certainty but no price control. Subject to market impact and slippage in volatile conditions.

See [Limit Order](#limit-order), [Slippage](#slippage), [Price Impact](#price-impact).

#### Limit Order
Order to buy or sell at a specified price or better. Provides price certainty but execution is not guaranteed. Remains in order book until fully filled, partially filled, or manually canceled. Core instrument in CLOBs.

**Trade-offs**: Price control but no execution certainty. May never fill if market doesn't reach limit price.

See [Market Order](#market-order), [CLOB](#clob-central-limit-order-book), [Time-In-Force](#time-in-force-tif).

#### Iceberg Order
Large order where only a small portion (the "tip") is visible in the order book at any time. As each visible portion fills, the next chunk is automatically revealed. Hides true order size to minimize market impact and information leakage.

**How it works**:
1. Submit large order (e.g., 10,000 tokens)
2. Display small portion (e.g., 500 tokens)
3. When 500 fills → next 500 appears
4. Repeat until full order fills

**Advantages**:
- ✅ Reduces market impact (large orders move price)
- ✅ Prevents front-running (traders can't see full size)
- ✅ Better execution price (less signal to adversaries)

**Disadvantages**:
- ⚠️ Slower execution (sequential chunks)
- ⚠️ Sophisticated traders can detect patterns
- ⚠️ Higher fees (multiple order placements)

See [Order Types](#order-types), [Price Impact](#price-impact), [Front-Running](#front-running).

#### OCO (One-Cancels-Other)
Paired order where execution of one automatically cancels the other. Allows setting both upside (take-profit) and downside (stop-loss) exits simultaneously. Common pattern: bracket order around current price.

**How it works**:
1. Place two linked orders (e.g., stop-loss at $95, take-profit at $105)
2. Current price: $100
3. If price hits $105 → take-profit executes, stop-loss cancels
4. If price hits $95 → stop-loss executes, take-profit cancels

**Use cases**:
- **Bracket orders**: Set profit target and loss limit simultaneously
- **Breakout trading**: Buy if price breaks above resistance OR sell if breaks below support
- **Risk management**: Automate exit strategy in both directions

See [Stop Orders](#stop-orders), [Take-Profit Order](#take-profit-order), [Stop-Loss Order](#stop-loss-order).

### Overcollateralized {#overcollateralized}
Coverage > 100%. Pool has more reserves than liabilities. Maximum negative skew (-100) incentivizes selling.

---

## P

### Prysm {#prysm}
Ethereum [consensus client](#consensus-client) written in Go by Prysmatic Labs. ~30% market share. Feature-rich with Web3Signer support, monitoring integrations, and web UI.

See [Consensus Client](#consensus-client) for comparison table.

### Prices {#prices}
**Master entry for price types.** Different price concepts serve different purposes in trading, risk management, and protocol design.

**Categories**: Quote prices (what traders see), Execution prices (what actually happens), Valuation/Risk prices (for risk management), AIMM-specific prices (protocol internals).

See [Price Impact](#price-impact), [Slippage](#slippage), [Volatility](#volatility-σ), [Deviation](#deviation-oracle).

#### Bid Price
Highest price that a buyer (individual, market maker, bot, or AMM) is willing to pay for an asset. Buyers offer bids at the bid price; the best (highest) bid constitutes the bid side of the market.

See [Ask Price](#ask-price), [Spread](#spread).

#### Ask Price
Lowest price that a seller (individual, market maker, bot, or AMM) is willing to accept for an asset. Sellers offer asks at the ask price; the best (lowest) ask constitutes the ask side of the market.

See [Bid Price](#bid-price), [Spread](#spread).

#### Mid-Price
Fair value estimate before spread application. In AIMM:
```
midPrice = oraclePrice × (1 + skewOffset)
```

Represents the theoretical fair price after inventory adjustment but before adding bid-ask spread. Used as reference point for quote generation.

See [Bid Price](#bid-price), [Ask Price](#ask-price), [Spread](#spread).

#### Execution Price
Final price a trader receives after accounting for:
1. Mid-price from oracle
2. Inventory skew offset
3. Spline traversal (price impact)
4. Fee deduction

Represents the actual price paid/received, including all protocol adjustments and fees.

See [Price Impact](#price-impact), [Slippage](#slippage).

#### Fair Value
Theoretical correct price of an asset. In AIMM, approximated by oracle TWAP with inventory adjustment. Represents the "true" market price independent of temporary imbalances or manipulations.

See [Oracle](#oracle), [TWAP](#twap-time-weighted-average-price).

#### Mark Price
For perpetual futures, the reference price used for calculating unrealized PnL and triggering liquidations. Typically derived from index price with funding rate adjustments to prevent manipulation. Differs from last-traded price to reduce liquidation cascade risk.

See [Index Price](#index-price), [Liquidation](#liquidation).

#### Index Price
For derivatives, the median or volume-weighted average price of an asset across multiple spot markets or CEXes. Used to prevent index manipulation and determine mark price. Provides fair reference price independent of any single exchange.

See [Mark Price](#mark-price), [Oracle](#oracle).

### Price Impact {#price-impact}
The deterministic price movement caused by a trade's size relative to available liquidity. Unlike [slippage](#slippage) (unexpected deviations), price impact is **predictable** from the [AMM](#amms-automated-market-makers) curve or order book depth before execution.

**Calculation basis**:
- **[AMM](#amms-automated-market-makers) pools**: Derived from bonding curve ([CPMM](#cpmm-constant-product-market-maker): x·y=k gives convex impact; [CLMM](#clmm-concentrated-liquidity-market-maker): tick-dependent)
- **Order books**: Sum of volume consumed across [price](#prices) levels to fill
- **AIMM**: Spline traversal through [liquidity profile](#liquidity-profile) gives fine-grained depth control

**Formula (simplified [CPMM](#cpmm-constant-product-market-maker))**:
```
priceImpact = tradeSize / (poolReserve + tradeSize)
```

**Distinction from Slippage**:
- **Price impact**: Known before trade, function of size/depth
- **[Slippage](#slippage)**: Unknown deviation from quote due to market movement, [MEV](#mev-maximal-extractable-value), or timing

**AIMM's approach**: Rather than using a fixed invariant curve, AIMM uses [spline-based liquidity profiles](#liquidity-profile) that allow pool operators to shape price impact across different trade sizes. This enables:
- Tighter impact for small trades (retail-friendly)
- Wider impact for large trades (LP protection)
- Dynamic adjustment based on [coverage ratio](#coverage-ratio) and [volatility](#volatility-σ)

See [Slippage](#slippage), [Spread](#spread), [Liquidity Profile](#liquidity-profile), [Dispersion](#dispersion).

### Protocol Fee {#protocol-fee}
Portion of spread retained by protocol:
```
protocolFee = totalFee × protoShare / 100
lpFee = totalFee - protocolFee
```

### Partial Fills {#partial-fills}
Standard execution policy where any executable portion of an order is filled immediately, with the remainder staying open until fully executed or canceled. Allows incremental order fulfillment instead of all-or-nothing execution.

### Price-Time Priority {#price-time-priority}
Matching rule in CLOBs where orders at better prices execute first, and among equal prices, earlier timestamps (earlier queue position) execute first. Determines execution priority and incentivizes liquidity provision early in price levels.

### Price Discovery {#price-discovery}
Process by which trading activity and order flow incorporate information into prices. Market-driven price formation through competition among participants. Critical for efficient markets and information dissemination.

### Quoting Strategy {#quoting-strategy}
Algorithmic policy determining where and when to place, update, or cancel quotes given current inventory, volatility, adverse selection, and order flow forecasts. Core component of market maker operations. Examples: inventory-targeting, adverse-selection adjusting, volatility-responsive.

### Queue Position {#queue-position}
Relative rank of a resting order within its price level, determining execution priority under price-time rules. Earlier positions execute first. Queue positioning strategies aim to secure favorable rank without unnecessary cancellations.

---

## R

### Reentrancy Guard {#reentrancy-guard}
Transaction-level [Flow Guard](#flow-guard) preventing recursive calls exploiting mid-execution state. Blocks functions from calling themselves or other protected functions during execution, preventing reentrancy attacks where malicious contracts recursively call back into the protocol before state updates complete.

**AIMM implementation**: Uses transient storage ([EIP-1153](#eip-1153-transient-storage)) for gas-efficient guards (~2,100 gas savings vs persistent storage).

See [Flow Guard](#flow-guard), [EIP-1153](#eip-1153-transient-storage).

### Relayer {#relayer}
Service that observes events or messages on one blockchain and relays them to another, particularly for cross-chain bridges. Relayers are economically incentivized via fees to faithfully transmit information. Bridge security typically depends on relayer honesty and the protocol's ability to penalize misbehavior.

### Reserves {#reserves}
Actual tokens held by pool. Updated on deposits, withdrawals, and swaps. Reserves serve as [Collateral](#collateral) backing LP token claims, with [Coverage Ratio](#coverage-ratio) measuring the collateralization level (reserves / liabilities).

See [Collateral](#collateral), [Coverage Ratio](#coverage-ratio), [Liabilities](#liabilities).

### Risk Config {#risk-config}
Per-asset safety parameters:
```solidity
struct RiskConfig {
    uint16 decayStartRatioBps;  // Decay activation threshold
    uint16 coverageMin;         // Critical minimum coverage (parametric)
    uint16 coverageMax;         // Critical maximum coverage (parametric)
    uint32 decaySlope;          // Decay rate
    uint16 depthAmplifier;      // Virtual depth at low coverage
    uint16 flags;               // Feature flags
}
```

### RLP (Recursive-Length Prefix Encoding) {#rlp-recursive-length-prefix-encoding}
Serialization format used by Ethereum to encode arbitrary nested data structures into byte sequences. Efficiently encodes lists, strings, and integers. Used internally for transaction and block serialization, state trees, and data storage.

### RPC (Remote Procedure Call) {#rpc-remote-procedure-call}
Network protocol enabling client-server communication where clients (wallets, dApps, services) request data or submit transactions to servers (blockchain nodes) over a network. The primary interface for users and applications to interact with the blockchain. RPC endpoints enable transaction submission, account balance queries, block retrieval, and smart contract execution calls.

### Reth (Rust Ethereum) {#reth-rust-ethereum}
Next-generation Ethereum [execution client](#execution-client) written in Rust by Paradigm. <1% market share but rapidly growing. Modular architecture with high performance and developer-friendly APIs. Production-ready.

See [Execution Client](#execution-client) for comparison table.

### Risk-Adjusted Performance Metrics {#risk-adjusted-performance-metrics}
**Master entry for risk and performance measurement.** Comprehensive metrics for quantifying uncertainty, downside exposure, return efficiency, and the relationship between asset movements and market benchmarks.

| Metric | Definition | Use Case | AIMM Application |
|--------|-----------|----------|------------------|
| **[Volatility (σ)](#volatility-σ)** | Annualized standard deviation of returns | Measure price uncertainty | Base [vega](#vega-ν) parameter, higher volatility → wider spread |
| **[Standard Deviation](#standard-deviation)** | Square root of variance (consistency measure) | Basis for volatility calculations | Compute from 20-day rolling window of price changes |
| **[Variance / Covariance](#variance--covariance)** | Squared deviation from mean / joint variation | Portfolio risk aggregation | Correlate asset price movements (not used directly, foundation) |
| **[Value at Risk (VaR)](#value-at-risk-var)** | Max loss at X% confidence over Y period | Regulatory and risk thresholds | Monitor loss bounds under extreme scenarios |
| **[Conditional VaR (CVaR)](#conditional-var-expected-shortfall)** | Expected loss beyond VaR threshold | Tail risk assessment | What happens beyond worst expected loss |
| **[Drawdown](#drawdown)** | Peak-to-trough decline in portfolio value | Measure suffered losses | Track cumulative impermanent loss for LPs |
| **[Beta (β)](#beta-β)** | Systematic risk relative to market | Compare volatility to S&P 500 or benchmark | Asset-to-market sensitivity (applied indirectly via oracle prices) |
| **[Skewness](#skewness)** | Asymmetry in return distribution | Detect fat tails (extreme events more common) | Detect when price changes cluster on one side (bearish vs bullish regime) |
| **[Kurtosis](#kurtosis)** | Heavy-tailedness (frequency of extreme events) | Measure tail risk beyond normal distribution | High kurtosis = frequent black swan events (increase spreads) |

**Key Insights**:
- **Volatility drives spreads**: Higher σ → AIMM charges wider spread (compensates LPs for risk)
- **Variance is squared volatility**: VAR(X) = σ², used in portfolio theory
- **VaR limitations**: Doesn't capture tail risk; CVaR does
- **Skewness signal**: Negative skew = downside crashes (bearish market structure); positive = upside explosions
- **Kurtosis detection**: Market regime change if kurtosis spikes (tail events becoming more common)

**Related**: [Volatility](#volatility-σ), [Vega](#vega-ν), [Price Impact](#price-impact), [Spread](#spread), [Statistical Concepts](#statistical-concepts).


#### Volatility Metrics
**Measures of price uncertainty and distribution characteristics.** Quantifies how much and in what pattern prices fluctuate.

##### Volatility (σ)
**Annualized standard deviation of asset returns.** Quantifies price uncertainty; higher volatility = less predictable prices = greater risk and trading costs.

**Calculation** (annualized from daily returns):
```
σ_annual = √(252) × σ_daily
where σ_daily = sqrt(mean((R_i - R_mean)²))
R_i = daily return = (Price_today - Price_yesterday) / Price_yesterday
```

**AIMM implementation**: Dual-window volatility EMAs (5-min and 1-hr) updated on every swap via internal oracle. Used to compute [vega](#vega-ν) component of spread:
```
spread_width = volatility_band × (1 + deviationSurcharge)
```

**Interpretation**:
- **Low volatility (5-10%)**: Stable assets (stablecoins, LRTs), tight spreads safe
- **Normal volatility (15-30%)**: Most crypto tokens, typical trading
- **High volatility (>50%)**: Altcoins, pump-and-dump cycles, wide spreads necessary

**Related**: [Standard Deviation](#standard-deviation), [Vega](#vega-ν), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics), [Spread](#spread).


##### Standard Deviation
**Square root of variance.** Measures how spread out returns are around their mean. Unit: same as original data (returns expressed as %).

**Formula**:
```
σ = √(1/N × Σ(x_i - μ)²)
where μ = mean, N = number of observations
```

**Relationship to variance**: σ = √(VAR)

**AIMM context**: 20-day rolling standard deviation of price changes forms the basis of volatility measurement. If σ suddenly doubles, market regime has changed.

See [Statistical Concepts](#statistical-concepts), [Volatility](#volatility-σ).


##### Variance / Covariance
**Variance**: Average squared deviation from mean (σ²). Measures dispersion.

**Covariance**: Joint variation between two variables. Positive covariance = variables move together; negative = inverse movement.

**Formula**:
```
VAR(X) = σ² = mean((X - μ)²)
COV(X,Y) = mean((X - μ_x)(Y - μ_y))
```

**Portfolio context**: Multi-asset portfolios require covariance matrix to compute aggregate risk. If two assets are highly correlated (COV > 0.9), diversification benefit is minimal.

**AIMM context**: Multi-token pools benefit from low correlation, when one asset drops, others may rise, reducing IL.

See [Risk & Volatility Metrics](#risk-adjusted-performance-metrics).


##### Skewness
**Asymmetry in the distribution of returns.** Measures whether extreme events cluster on one side (upside or downside).

**Interpretation**:
- **Skewness ≈ 0**: Symmetric distribution (normal, fair)
- **Positive skewness**: Right tail longer (occasional huge gains, common small losses)
- **Negative skewness**: Left tail longer (occasional crashes, common small gains)

**Example**:
- Lotto ticket: Huge positive skew (99.9% lose $1, 0.1% win $1M)
- Most trading: Slight negative skew (frequent small gains, rare crashes)

**Market regime signals**:
- **Negative skew rising**: Bearish (crashes becoming more likely)
- **Positive skew rising**: Bullish or speculative (upside tail events priced in)

**AIMM context**: Rising negative skew detected via oracle updates → increase spreads (market expects downside risk).

See [Kurtosis](#kurtosis), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics).


##### Kurtosis
**Heavy-tailedness of the distribution.** Measures frequency of extreme events relative to normal distribution.

**Interpretation**:
- **Kurtosis ≈ 3** (excess kurtosis = 0): Normal distribution (frequent extreme events are rare)
- **Kurtosis > 3**: Leptokurtic (fat tails, black swans more common than normal predicts)
- **Kurtosis < 3**: Platykurtic (thin tails, smoother than normal)

**Practical meaning**: High kurtosis = "weird things happen more often than expected"

**Example**:
- Normal stock market: Extreme 5-sigma moves happen ~1× per 3.5M years
- Crypto markets: Extreme moves happen ~1× per month (higher kurtosis)

**AIMM context**: Monitor kurtosis via oracle; spike indicates market regime change (circuit breaker trigger threshold).

**Related**: [Skewness](#skewness), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics), [Statistical Concepts](#statistical-concepts).



#### Risk Metrics
**Measures of downside exposure and systematic risk.** Quantifies potential losses and market sensitivity.

##### Value at Risk (VaR)
**Maximum expected loss at X% confidence level over Y time period.** For example, "1-day VaR at 95% confidence = $100,000" means there's only a 5% chance of losing more than $100k in the next day.

**Calculation**:
```
VaR_95 = Percentile(sorted_losses, 95th)
```

**Limitations**:
- Does NOT tell you what happens beyond the threshold (tail risk)
- Assumes historical distribution continues (ignores regime changes)
- Can be gamed (false confidence)

**AIMM context**: Pool operators monitor VaR to ensure coverage ratio stays above emergency thresholds. If portfolio value drops beyond VaR, [circuit breakers](#circuit-breaker) may activate.

**Related**: [Conditional VaR (Expected Shortfall)](#conditional-var-expected-shortfall), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics).


##### Drawdown
**Peak-to-trough decline in portfolio value.** Measures the largest cumulative loss from a historical high point.

**Example**:
- Portfolio peak at $1,000,000 (day 10)
- Portfolio low at $700,000 (day 20)
- Drawdown = -30%

**Maximum Drawdown**: Worst drawdown in entire history. Indicates the worst case experience an investor faced.

**AIMM context**: LPs experience drawdowns from [impermanent loss](#impermanent-loss-il) when asset prices diverge. AIMM's [coverage ratio](#coverage-ratio)-based depth and oracle pricing reduce realized drawdown.

**Related**: [Impermanent Loss](#impermanent-loss-il), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics).


##### Beta (β)
**Systematic risk relative to a market benchmark.** Measures how an asset moves relative to the overall market (e.g., S&P 500).

**Formula**:
```
β = COV(Asset, Market) / VAR(Market)
```

**Interpretation**:
- **β = 1**: Moves in line with market
- **β > 1**: More volatile than market (amplified swings)
- **β < 1**: Less volatile than market (smoother)
- **β < 0**: Inverse movement (hedge)

**Examples**:
- BTC β ≈ 1.0-1.2 (slightly more volatile than broader crypto)
- USDC β ≈ 0.0 (no correlation to market)
- Inverse ETF β ≈ -1.0 (perfect hedge)

**Crypto context**: Beta typically measured vs BTC or broader market cap index. Altcoin β > 1.0 (more leverage to market moves).

See [Risk & Volatility Metrics](#risk-adjusted-performance-metrics), [Volatility](#volatility-σ).



#### Return Metrics
**Measures of return efficiency and asset relationships.** Quantifies risk-adjusted performance and correlation patterns.

##### Expected Return
**Probability-weighted average of all possible outcomes.** Used to forecast portfolio performance.

**Formula**:
```
E[R] = Σ(probability_i × return_i)
```

**Example**: Asset with 60% chance of +10% return and 40% chance of -5%:
```
E[R] = 0.6 × 10% + 0.4 × (-5%) = 6% - 2% = 4%
```

**AIMM context**: Expected LP return = (spread × volume) - IL - fees paid. Forecast depends on:
- **Estimated transaction volume** (from historical patterns)
- **Spread estimate** (based on vega, deviation, coverage)
- **IL estimate** (from price movement distribution)

**Limitation**: Based on historical assumptions; fails during market regime changes.

See [Return & Performance Metrics](#risk-adjusted-performance-metrics), [Expected Return](#expected-return).


##### Sharpe Ratio
**Excess return per unit of volatility.** Risk-adjusted performance measure comparing an investment's return to its volatility.

**Formula**:
```
SR = (Portfolio_Return - Risk_Free_Rate) / Portfolio_Volatility
```

Where Risk-Free-Rate is typically 0-5% (US Treasury, lending rates).

**Interpretation**:
- **SR < 0.5**: Weak returns relative to risk (avoid)
- **0.5 < SR < 1.0**: Moderate performance (acceptable)
- **SR > 1.0**: Good risk-adjusted returns (desirable)
- **SR > 2.0**: Excellent performance (rare, may indicate backtest bias)

**Example**: Strategy A has 12% return, 10% volatility → SR = 1.2; Strategy B has 8% return, 5% volatility → SR = 1.6 (better despite lower absolute return).

**AIMM context**: Use Sharpe ratio to compare AIMM performance (with varying slippage, fees, IL) against passive holding or alternative AMMs.

See [Volatility](#volatility-σ), [Return & Performance Metrics](#risk-adjusted-performance-metrics).


##### Alpha (α)
**Excess return beyond what's explained by market risk (beta).** Represents manager skill or strategy outperformance.

**Formula** (Capital Asset Pricing Model):
```
α = Actual_Return - Expected_Return
where Expected_Return = Risk_Free_Rate + β × (Market_Return - Risk_Free_Rate)
```

**Interpretation**:
- **α ≈ 0**: Strategy matches benchmark (not beating market)
- **α > 0**: Outperformance (manager/strategy has skill or edge)
- **α < 0**: Underperformance (losing to benchmark)

**Example**: AIMM achieves 8% return with β=0.8 in a 10% bull market:
```
Expected return = 2% (risk-free) + 0.8 × (10% - 2%) = 8.4%
Actual return = 8%
α = 8% - 8.4% = -0.4% (slight underperformance)
```

**AIMM context**: Compare AIMM's actual LP returns vs oracle-passive baseline. Positive α indicates superior pricing or fee structure.

See [Beta (β)](#beta-β), [Return & Performance Metrics](#risk-adjusted-performance-metrics).


##### Correlation
**Measure of joint movement between two variables, normalized to [-1, +1] range.**

**Formula**:
```
CORR(X, Y) = COV(X, Y) / (σ_x × σ_y)
```

**Interpretation**:
- **CORR = +1**: Perfect positive (move together)
- **CORR = 0**: Independent (no relationship)
- **CORR = -1**: Perfect negative (inverse movement / hedges)

**Examples**:
- BTC & ETH: CORR ≈ 0.85 (highly correlated, poor diversification benefit)
- USDC & volatile altcoin: CORR ≈ 0.1 (low correlation, good diversification)
- USDC & reverse perpetual futures: CORR ≈ -0.9 (hedging pair)

**AIMM context**: Multi-token pools benefit from low-correlation pairs. ETH-BTC pool: high IL. ETH-USDC pool: lower IL (stablecoin stabilizes value).

**Related**: [Variance / Covariance](#variance--covariance), [Return & Performance Metrics](#risk-adjusted-performance-metrics).


##### Autocorrelation / Autoregression
**Self-correlation of a time series with its own lagged values.** Detects patterns: does a price change today predict tomorrow's change?

**Formula** (lag-1 autocorrelation):
```
AC_1 = CORR(X_t, X_{t-1})
```

**Interpretation**:
- **AC > 0 (positive)**: Momentum (today's move predicts similar tomorrow → trending markets)
- **AC ≈ 0**: Random walk (no pattern)
- **AC < 0 (negative)**: Mean reversion (overshoots correct → oscillating markets)

**Example**:
- Strong uptrend: AC > 0.5 (continuation likely)
- Oscillating between bounds: AC < -0.3 (reversal likely)

**Autoregression (AR)**: Forecasting model predicting future values from lagged history:
```
X_t = α + β × X_{t-1} + ε_t
```

**AIMM context**: Detect market regime:
- **Positive AC**: Trending market, may need wider spreads (adverse selection harder to detect)
- **Negative AC**: Mean-reversion market, tight spreads work better
- **Switching AC**: Market regime changing (circuit breaker trigger)

**Related**: [Stationarity](#stationarity), [Return & Performance Metrics](#risk-adjusted-performance-metrics).


### Conditional VaR (Expected Shortfall) {#conditional-var-expected-shortfall}
**Average loss in the worst X% of outcomes.** If VaR is the threshold, CVaR is the mean loss beyond that threshold, captures tail risk.

**Formula**:
```
CVaR = mean(losses where loss > VaR)
```

**Example**: VaR_95 = $100k, CVaR_95 = $150k means: in the worst 5% of days, the *average* loss is $150k (not just $100k).

**Advantage**: Incorporates tail events; better for risk management.

**AIMM context**: CVaR better reflects black swan risk (protocol hacks, oracle failures, extreme market moves). Higher CVaR suggests need for larger [coverage ratio](#coverage-ratio) buffer.

See [Value at Risk (VaR)](#value-at-risk-var), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics).

### Portfolio Optimization {#portfolio-optimization}
**Master entry for mathematical techniques allocating capital to maximize returns while controlling risk.** Foundation for institutional portfolio management and multi-asset protocol design.

| Technique | Goal | Inputs | Output | AIMM Relevance |
|-----------|------|--------|--------|-----------------|
| **[Mean-Variance Optimization](#mean-variance-optimization)** | Efficient capital allocation | Expected returns, volatility, correlation | Optimal weights | Design LP liquidity allocation across assets |
| **[Efficient Frontier](#efficient-frontier)** | Visualize risk-return trade-off | Portfolio weights, returns, risk | Curve of optimal portfolios | Understand Pareto-optimal risk levels |
| **[Diversification](#diversification)** | Risk reduction via uncorrelated assets | Correlation matrix | Weight allocation | Multi-token pool design (low-corr pairs) |
| **[Cointegration](#cointegration)** | Detect mean-reverting pairs | Time series of two assets | Cointegration coefficient | Identify pairs that drift together then revert |
| **[Principal Component Analysis (PCA)](#principal-component-analysis-pca)** | Dimensionality reduction | Large correlation matrix | Linearly independent components | Simplify multi-asset risk analysis |

**Key Insights**:
- **Efficient frontier**: No portfolio is better than all others on the frontier, only trade-offs
- **Diversification benefit**: Depends on correlation; uncorrelated assets reduce portfolio volatility
- **Cointegration pairs**: Long-short pairs that earn return from spread mean-reversion
- **PCA components**: First PC captures most variance; subsequent PCs capture residual patterns
- **In DeFi**: Multi-token pools optimize for low IL via correlation, not market returns

**Related**: [Risk & Volatility Metrics](#risk-adjusted-performance-metrics), [Return & Performance Metrics](#risk-adjusted-performance-metrics).

### Mean-Variance Optimization {#mean-variance-optimization}
**Mathematical framework allocating capital to maximize expected return for a given volatility target (or minimize volatility for a given return target).** Foundation of Modern Portfolio Theory by Harry Markowitz.

**Optimization problem**:
```
Maximize: E[R_p] = Σ(w_i × E[R_i])
Subject to: VAR[R_p] = Σ(w_i² × VAR[R_i]) + Σ(2 × w_i × w_j × COV[R_i, R_j]) ≤ σ_target
            Σ(w_i) = 1 (weights sum to 100%)
```

**Solution**: Portfolio on the [Efficient Frontier](#efficient-frontier).

**AIMM context**: Multi-token pools select weights to minimize [impermanent loss](#impermanent-loss-il) for a given set of assets. Low-correlation pairs (w1=50% BTC, w2=50% ETH) have lower IL than high-correlation pairs.

**Limitation**: Assumes returns are stationary and normally distributed; fails during regime changes.

See [Efficient Frontier](#efficient-frontier), [Portfolio Optimization](#portfolio-optimization).

### Efficient Frontier {#efficient-frontier}
**Curve representing the set of optimal portfolios that maximize return for each level of risk (or minimize risk for each level of return).** All portfolios on the frontier are equally desirable, choices depend on risk tolerance.

**Properties**:
- **Points below**: Inefficient (can achieve same return with less risk elsewhere)
- **Points above**: Unattainable (no combination achieves that risk-return trade-off)
- **Concave shape**: Reflects diminishing diversification benefits as you add more assets

**Example**:
```
Efficient Frontier portfolios for stocks:
- Portfolio A: 100% bonds, return 3%, volatility 2%
- Portfolio B: 70% stocks/30% bonds, return 8%, volatility 8%
- Portfolio C: 100% stocks, return 12%, volatility 18%
All on frontier; no portfolio is "best"-choice depends on risk appetite.
```

**AIMM context**: Multi-token pool selection moves along the efficient frontier as volatility changes. Tight spreads assume low-risk conditions (left side); wide spreads for high-volatility (right side).

See [Mean-Variance Optimization](#mean-variance-optimization), [Diversification](#diversification).

### Diversification {#diversification}
**Reducing portfolio risk by holding assets that don't move perfectly together (low correlation).** Core principle: "Don't put all eggs in one basket."

**Benefit magnitude**:
```
Diversification = 1 - (Portfolio_Volatility / Weighted_Individual_Volatility)
If assets uncorrelated: Portfolio_Volatility = √(Σ(w_i² × σ_i²))
```

**Example**:
- Hold 50% BTC (σ=80%) + 50% USDC (σ=0%): Portfolio σ ≈ 40% (large reduction)
- Hold 50% BTC (σ=80%) + 50% ETH (σ=75%, ρ=0.85): Portfolio σ ≈ 60% (less reduction due to correlation)

**Limitations**:
- Breaks during crisis: All assets correlate during market crashes (correlation → 1.0)
- Over-diversification: Too many assets increases operational complexity without benefit
- False diversification: Holding 100 correlated assets isn't truly diversified

**AIMM context**: LP pools benefit from low-correlation asset pairs. BTC-USDC pool has lower IL than BTC-ETH pool (lower correlation = less adverse selection).

See [Correlation](#correlation), [Efficient Frontier](#efficient-frontier).

### Cointegration {#cointegration}
**Statistical relationship where two non-stationary time series move together over time, with deviations mean-reverting to equilibrium.** Different from [correlation](#correlation) which measures synchronous movement.

**Example**:
- Stock A trades at $100, Stock B at $150 (ratio 1:1.5)
- Both drift higher over years: A → $200, B → $300 (still ratio 1:1.5)
- If B suddenly spikes to $350, ratio becomes 1:1.75 (deviation from equilibrium)
- Mean reversion strategy: Sell B, buy A, wait for ratio to normalize

**Test**: Johansen cointegration test detects whether two series are cointegrated.

**DeFi context**: Stablecoin pairs (USDC-USDT-DAI) are cointegrated, they can diverge (0.99-1.01) but always revert. LP spreads can be tighter for cointegrated pairs (less unpredictable divergence).

**Related**: [Stationarity](#stationarity), [Correlation](#correlation).

### Principal Component Analysis (PCA) {#principal-component-analysis-pca}
**Dimensionality reduction technique converting highly correlated variables into a smaller set of uncorrelated "principal components."** Each PC captures variance from original data, with PC1 capturing the most.

**Use case**: 10 correlated asset returns → 3 principal components capturing 90% of variance, simplifying analysis.

**Process**:
1. Compute correlation matrix of all assets
2. Extract eigenvectors (directions) and eigenvalues (variance captured)
3. Project data onto top N eigenvectors (principal components)
4. PC1 represents the direction explaining most portfolio variance

**Interpretation**:
- **PC1** (market risk): General up/down movement affecting all assets
- **PC2** (sector risk): Style factor differentiating asset classes
- **PC3+** (residual): Idiosyncratic risks

**AIMM context**: Simplify multi-token pool analysis. If pool has 20 tokens, PCA might show that 3 principal components explain 95% of risk, focus hedging/allocation on those 3.

**Related**: [Variance / Covariance](#variance--covariance), [Correlation](#correlation).

---

## S

### Sui Core {#sui-core}
Sui's [execution](#execution-client) and [consensus](#consensus) layer written in Rust. Integrates object-centric storage, [Narwhal](#narwhal) data availability, and [Mysticeti](#mysticeti) consensus.

**Responsibilities**:
1. **Execution**: Processes transactions, updates object state (Move language)
2. **Data availability**: Participates in [Narwhal](#narwhal) batch commitments
3. **Consensus**: Participates in [Mysticeti](#mysticeti) DAG consensus

**Unique properties**:
- **Object-centric**: State is "objects" (Move resources), not accounts/storage slots
- **Parallelization**: Independent transactions can execute in parallel if accessing different objects
- **Storage model**: Uses versioned objects with causal histories (DAG of object versions)

**Development**: Actively maintained by Mysten Labs with continuous optimization.

See [Mysticeti](#mysticeti), [Narwhal](#narwhal), [Consensus](#consensus), [Execution Client](#execution-client).

### sBTR (Staked BTR) {#sbtr-staked-btr}
Receipt token for staked governance tokens. Required for voting and reward boosting. Subject to transfer cooldown.

### MEV Searcher {#mev-searcher}
Independent network participants who identify and execute [MEV](#mev-maximal-extractable-value) opportunities using sophisticated algorithms and infrastructure. Searchers continuously monitor the mempool for profitable trades, construct transaction bundles, and submit them to block builders or MEV relays. They participate across the full spectrum of [extraction strategies](#arbitrage): from harmful ([Sandwich Attacks](#sandwich-attack), [Frontrunning](#frontrunning)) to benign ([Backrunning](#backrun), [JIT](#jit-just-in-time-liquidity), liquidations, [Statistical Arbitrage](#statistical-arbitrage-stat-arb)). See [Arbitrage](#arbitrage) and [MEV Strategies](#mev-strategies) for the taxonomy of extraction types.

### SAFT (Simple Agreement for Future Tokens) {#saft-simple-agreement-for-future-tokens}
Fundraising instrument where investors agree to receive tokens at a future date upon specified events (typically network launch). Used to raise capital before token issuance without immediate regulatory classification as a security. Converts to tokens at future launch.

### Secp256k1 {#secp256k1}
Elliptic curve used in ECDSA for Bitcoin and Ethereum transaction signing. Chosen for efficiency and security. Each private key corresponds to unique public key on secp256k1 curve. Standard across most blockchains for digital signatures.

### SHA-256 (Secure Hash Algorithm) {#sha-256-secure-hash-algorithm}
256-bit cryptographic hash function standardized by NIST. Produces 256-bit output. Used in Bitcoin consensus and many other protocols. Slower than Keccak-256 but widely standardized and trusted.

### SHA3-256 {#sha3-256}
256-bit variant of NIST's SHA-3 standard (finalized 2015). Different from Ethereum's Keccak-256, which is the original Keccak submission. SHA-3 uses sponge construction for flexibility and security.

### SHA2-256 {#sha2-256}
Another name for SHA-256, indicating it's part of the SHA-2 family (SHA-256, SHA-512, etc.). Produces 256-bit hash. The industry standard before SHA-3 finalization.

### Sequencer {#sequencer}
Entity controlling transaction ordering and block production in rollups ([Arbitrum](#arbitrum), Optimism, Base, etc.). Acts as the leader in the rollup's [consensus](#consensus), submitting ordered transactions and state commitments to the base layer (Ethereum L1).

**Responsibilities**:
1. **Transaction ordering**: Receives user transactions from [mempool](#mempool), decides ordering
2. **Block production**: Groups ordered transactions into blocks, executes them
3. **State commitment**: Submits block headers/state roots to L1 smart contract
4. **Soft finality**: Provides sequencer-guaranteed block ordering (~instant)
5. **Hard finality**: Waits for L1 batch confirmation (~10-30 min)

**Finality hierarchy**:
- **Sequencer finality** (soft): Sequencer promises to include transaction (not reversible without sequencer being malicious)
- **Batch finality** (hard): State root committed to L1 smart contract (reversible only via L1 reorg)
- **L1 finality** (final): L1 batch lands in finalized Ethereum block (~12.8 min for [Casper FFG](#casper-ffg))

**MEV on rollups**:
- **Arbitrum**: Originally FCFS ordering, now [Timeboost](#timeboost) auctions for priority
- **Optimism/Base**: FCFS + priority fee model (sequencer profitable to include high-fee txs)
- **Sequencer profit**: Transaction fees + MEV from reordering (if allowed)

**Centralization concerns**:
- **Single sequencer**: Centralized ordering = single point of MEV extraction
- **Censorship risk**: Sequencer could block transactions
- **Decentralization roadmap**: Most rollups planning decentralized sequencer sets

**Comparison**:
- **L1 [Validator](#validator)**: Proposes blocks in [consensus](#consensus), earns base reward + MEV
- **L2 Sequencer**: Receives transactions, orders them, earns transaction fees + MEV, submits to L1

See [Consensus](#consensus), [Mempool](#mempool), [MEV](#mev-maximal-extractable-value), [Block Builder](#block-builder), [Timeboost](#timeboost).

### Skew {#skew}
See [Inventory Skew](#inventory-skew).

### Slashing {#slashing}
Penalty mechanism in Proof of Stake where validators lose a portion of staked collateral for misbehavior (double voting, offline, consensus violations). Enforces honest validator participation by making dishonesty economically costly. Varies by protocol (1-100% of stake).

### Staking Pool {#staking-pool}
Smart contract pool allowing multiple ETH holders to combine capital to collectively meet validator staking requirements (typically 32 ETH). Pool participants earn proportional staking rewards minus pool operator fees. Enables small-scale staking without running hardware.

### sLP (Staked LP) {#slp-staked-lp}
Receipt token for staked LP positions. Deployed deterministically via CREATE3.

### Slippage {#slippage}
The difference between a quoted/expected price and the actual execution price of a trade. Slippage occurs due to:
1. **Market movement**: Price changes between quote time and execution (general volatility)
2. **Price impact**: Large trades moving the market against the trader (liquidity depth)
3. **Third-party activity**: [Front-running](#frontrunning), [sandwich attacks](#sandwich-attack), or other [MEV](#mev-maximal-extractable-value) extraction between quote and execution

**Key distinction from [Price Impact](#price-impact)**: Price impact is the deterministic cost of trade size against available liquidity (predictable from the order book/AMM curve). Slippage is the *unexpected* deviation from quoted price due to external factors.

**Slippage can be positive or negative**: If the market moves favorably between quote and execution, the trader receives *positive slippage* (better price than expected). This is rare on public [mempools](#mempool) due to MEV extraction.

**AIMM Protection**: The `minAmountOut` parameter in `swap()` enforces maximum acceptable slippage, if execution price would result in less than `minAmountOut`, the transaction reverts:
```solidity
if (finalAmountOut < minAmountOut) revert ThresholdViolation(finalAmountOut, minAmountOut);
```

**Mitigation strategies**:
- Set tight `minAmountOut` (accept only small deviations)
- Use [MEV-protected RPCs](#mev-protected-rpc) to hide transactions from [searchers](#mev-searcher)
- Trade during low-volatility periods
- Split large orders to reduce [price impact](#price-impact)

See [Price Impact](#price-impact), [Spread](#spread), [MEV](#mev-maximal-extractable-value), and [MEV-Protected RPC](#mev-protected-rpc) for related concepts.

### Statistical Concepts {#statistical-concepts}
**Master entry for mathematical and statistical properties.** Fundamental concepts used in analyzing price behavior, designing curve parameters, and validating market assumptions.

| Concept | Statistical Definition | AIMM Application | Example |
|---------|----------------------|------------------|---------|
| **[Monotonicity](#monotonicity)** | Function always increases or decreases (never reverses) | Liquidity depth must increase monotonically within price segments | Spline curve cannot have "dips" where deeper liquidity exists below |
| **[Linearity](#linearity)** | Proportional relationship (f(ax + by) = af(x) + bf(y)) | Price-depth relationship not perfectly linear (curved via spline) | Doubling trade size ≠ double execution cost due to price impact |
| **[Stationarity](#stationarity)** | Statistical properties (mean, variance) constant over time | Market assumptions requiring stationary price distributions | Oracle prices should have stable volatility regimes (detect regime changes) |
| **[Normalization](#normalization)** | Scaling data to fixed range (typically [0,1]) | Price normalization by oracle reference before spline evaluation | Normalize order size relative to pool liquidity for consistent depth |
| **[Standardization](#standardization)** | Transform to mean=0, std=1 (Z-scores) | Volatility standardization (volatility / historical mean) | Compare today's volatility to 20-day MA, express as standard deviations |
| **[Density Analysis](#density-analysis)** | Measure concentration of observations around center | Liquidity density profiling using TWAP-weighted distribution | Shape spline weights based on where most trading volume clusters |

**Key Insights**:
- **Monotonicity**: Critical for spline stability, non-monotonic depth creates negative liquidity
- **Linearity violations**: Price-depth relationship is curved due to AMM properties and [price impact](#price-impact)
- **Stationarity**: Assumes market behavior is consistent, violations trigger oracle [circuit breakers](#circuit-breaker)
- **Normalization vs Standardization**: Both rescale data; normalization bounds values [0,1], standardization centers on mean with unit variance
- **Density analysis**: Identifies where liquidity should concentrate (high-volume price zones need tighter depth shaping)

**Related**: [Volatility](#volatility-σ), [Price Impact](#price-impact), [Liquidity Profile](#liquidity-profile), [Fritsch-Carlson Monotone Cubic Hermite Interpolation](#fritsch-carlson-monotone-cubic-hermite-interpolation), [Makima Spline](#makima-spline).

### Monotonicity {#monotonicity}
Mathematical property where a function is always non-decreasing or non-increasing over its domain. In trading, **monotonic depth** means liquidity gets consistently deeper (or shallower) as you move away from market price, never reverses. Critical for spline-based profiles to prevent "dips" where a shallower price zone lies between deeper ones (which would create arbitrage opportunities).

**AIMM context**: Spline weights must be arranged so that depth increases monotonically from center outward. Violation causes pricing inconsistency.

See [Statistical Concepts](#statistical-concepts), [Spline (Cubic Interpolation)](#spline-cubic-interpolation).

### Linearity {#linearity}
In statistics: proportional relationship where doubling input doubles output (f(2x) = 2f(x)). In blockchain/trading: the **assumption is false**-price depth relationship is non-linear due to [price impact](#price-impact) and curve characteristics.

**Statistical definition**: f(ax + by) = af(x) + bf(y) (superposition property)

**AIMM context**: Price function is non-linear because depth changes curve shape. Small trades use tight spread, large trades face wider impact.

See [Statistical Concepts](#statistical-concepts), [Price Impact](#price-impact).

### Stationarity {#stationarity}
Statistical property where the mean, variance, and autocorrelation structure remain constant over time. A **stationary price process** has consistent statistical properties across time periods, enabling prediction and model stability.

**Test**: Augmented Dickey-Fuller (ADF) test determines if a time series is stationary or has a unit root (non-stationary).

**AIMM context**: Oracle prices should exhibit stationarity within regimes. If [volatility](#volatility-σ) doubles and persists (regime change), AIMM should detect and adapt. Non-stationary behavior triggers [circuit breakers](#circuit-breaker).

**Example**: If volatility spikes and stays high, prices are in a new regime (non-stationary relative to old mean). Algorithms must recalibrate parameters.

See [Statistical Concepts](#statistical-concepts), [Volatility](#volatility-σ).

### Normalization {#normalization}
Process of scaling data to a fixed range, typically [0, 1] or [-1, 1]. Preserves the shape of distribution while fitting within bounds. Useful for comparing variables with different units.

**Formula**:
```
normalized_x = (x - min(x)) / (max(x) - min(x))  → [0, 1]
```

**AIMM context**: Order size normalized relative to pool liquidity depth for consistent pricing across different assets.

**Difference from [Standardization](#standardization)**: Normalization bounds to range; standardization centers with zero mean.

See [Statistical Concepts](#statistical-concepts), [Standardization](#standardization).

### Standardization {#standardization}
Process of transforming data to have mean = 0 and standard deviation = 1, producing **Z-scores**. Enables comparison of variables with different scales.

**Formula**:
```
Z = (X - μ) / σ
where μ = mean, σ = standard deviation
```

**AIMM context**: Volatility standardized relative to historical baseline (current vol / 20-day MA). Expressed as "2.5 standard deviations above mean" triggers higher spreads.

**Use case**: Comparing today's volatility ($50k BTC daily swing) to baseline, independent of BTC absolute price.

See [Statistical Concepts](#statistical-concepts), [Normalization](#normalization), [Volatility](#volatility-σ).

### Density Analysis {#density-analysis}
Statistical technique measuring the concentration of observations (data points) around a central location or specific regions. In finance, analyzes where volume/liquidity clusters on price axis.

**AIMM context**: Uses TWAP-weighted density analysis to shape spline liquidity profiles. Where is 80% of trading volume clustered? Concentrate depth there. Where is volatility extreme? Widen spreads.

**Application**:
1. Compute historical volume distribution across price levels
2. Weight spline knots to match volume density
3. Result: Tighter spreads where most trading happens, wider at extremes

**Related**: [Liquidity Shaping](#liquidity-shaping), [Time-Weighted Average Price](#twap), [Spline (Cubic Interpolation)](#spline-cubic-interpolation).

### Statistical Methods {#statistical-methods}
**Master entry for quantitative techniques analyzing financial data and testing market assumptions.** Methodologies for estimation, validation, and forecasting used in portfolio management and protocol design.

| Method | Purpose | Inputs | Output | AIMM Application |
|--------|---------|--------|--------|------------------|
| **[Monte Carlo Simulation](#monte-carlo-simulation)** | Quantify tail risk, forecast outcomes | Distribution parameters, # simulations | Confidence bands, tail probabilities | Model extreme market scenarios for coverage ratio stress-testing |
| **[Regression Analysis](#regression-analysis)** | Model relationships, predict values | Dependent and independent variables | Coefficients, R², significance tests | Identify drivers of volatility or toxicity (oracle vs volume correlation) |
| **[Garman-Klass Estimator](#garman-klass-estimator)** | Estimate volatility from OHLC data | Open, High, Low, Close prices | Volatility estimate | Compute vol from limited data without returns distribution assumption |
| **[ADF Test (Stationarity)](#adf-test-augmented-dickey-fuller)** | Test if series is stationary | Time series data | Test statistic, p-value | Verify oracle prices don't have unit root (mean-reverting behavior) |
| **[Correlation / Cointegration Tests](#cointegration)** | Detect relationships between assets | Two time series | Correlation coefficient, cointegration statistic | Identify pairs for diversified pools (low correlation = lower IL) |

**Key Insights**:
- **Monte Carlo**: Scenario analysis (what if vol doubles?), stress-testing
- **Regression**: Causal inference (does higher volume increase spread alpha?)
- **Garman-Klass**: Better vol estimator than simple returns (uses price range)
- **ADF test**: Confirms market assumptions (prices are mean-reverting, not random walks)
- **Cointegration**: Finds pairs trading opportunities or diversification benefits

**Related**: [Risk & Volatility Metrics](#risk-adjusted-performance-metrics), [Statistical Concepts](#statistical-concepts), [Return & Performance Metrics](#risk-adjusted-performance-metrics).

### Monte Carlo Simulation {#monte-carlo-simulation}
**Probabilistic modeling technique using random sampling to simulate many possible outcomes and understand their distribution.** Generates confidence intervals and tail risk estimates.

**Process**:
1. Define parameters (volatility, mean return, correlations)
2. Generate N random scenarios (e.g., 10,000 paths)
3. Simulate price paths using stochastic model (e.g., geometric Brownian motion)
4. Analyze distribution of outcomes (mean, percentiles, tail risk)

**Example**:
```
Model: BTC prices with μ=0.05% daily return, σ=3% daily volatility
Monte Carlo: Simulate 10,000 possible price paths over 30 days
Results: 5th percentile = -40% (5% chance of >40% loss)
         95th percentile = +35% (5% chance of >35% gain)
```

**AIMM context**: Stress-test coverage ratio under extreme market scenarios. Monte Carlo reveals: "Under 1-in-100-day volatility scenario, coverage ratio drops to 45%-need safety buffer."

**Limitations**: Assumes parameters are stationary (fails during regime changes).

See [Statistical Methods](#statistical-methods), [Risk & Volatility Metrics](#risk-adjusted-performance-metrics).

### Regression Analysis {#regression-analysis}
**Statistical method modeling the relationship between a dependent variable (y) and one or more independent variables (x).** Quantifies how much x influences y and allows predictions.

**Linear Regression Formula**:
$$y = \alpha + \beta_1 \cdot x_1 + \beta_2 \cdot x_2 + \ldots + \varepsilon$$

where:
- $\alpha$ = intercept
- $\beta_i$ = coefficient (slope) for variable $x_i$
- $\varepsilon$ = error term (unexplained variation)

**Key Metrics**:
- **R²**: Percentage of y's variance explained by model (0-1 scale)
- **β coefficients**: Magnitude and direction of variable influence
- **p-value**: Statistical significance (p < 0.05 = significant)

**Example**:
```
Model: Daily_IL = α + β₁×Volatility + β₂×Volume + ε
Result: β₁ = 0.45 (1% vol increase → 0.45% more IL)
        β₂ = -0.02 (1% volume increase → -0.02% less IL via fee offset)
        R² = 0.72 (72% of IL variation explained)
```

**AIMM context**: Regression identifies factors driving protocol performance. Examples:
- Does coverage ratio volatility predict toxic flow?
- How much does oracle staleness impact spread adequacy?

See [Statistical Methods](#statistical-methods), [Return & Performance Metrics](#risk-adjusted-performance-metrics).

### Garman-Klass Estimator {#garman-klass-estimator}
**Volatility estimator using high-low price range (OHLC data) instead of full return distribution.** More efficient than standard deviation when you have limited data.

**Formula** (simplified):
```
GK_vol = √(0.5 × log(H/L)² - (2log(2) - 1) × log(C/O)²)
where H=high, L=low, O=open, C=close
```

**Advantages over standard deviation**:
- ✅ Uses price range (high-low), not just open-close returns
- ✅ More efficient with small samples
- ✅ Less sensitive to gaps and overnight moves
- ✅ Lower variance estimator (more stable)

**When to use**:
- **Garman-Klass**: 5-minute or hourly bars (limited data points)
- **Standard deviation**: 1-minute bars or longer-term (abundant data)

**AIMM context**: Compute intraday volatility from 5-minute OHLC bars without waiting for end-of-day returns.

See [Volatility](#volatility-σ), [Statistical Methods](#statistical-methods).

### ADF Test (Augmented Dickey-Fuller) {#adf-test-augmented-dickey-fuller}
**Statistical test determining whether a time series is stationary (stable properties) or has a unit root (non-stationary / random walk).** Critical for validating model assumptions.

**Null hypothesis**: Series has unit root (non-stationary, drifts without mean reversion)

**Test statistic**: Compares observed autocorrelation to critical value.

**Interpretation**:
- **p-value < 0.05**: Reject null → Series is stationary (mean-reverting)
- **p-value > 0.05**: Fail to reject → Series is non-stationary (random walk)

**Example**:
- **Stablecoin price**: ADF p-value = 0.01 → Stationary (reverts to $1.00)
- **BTC price**: ADF p-value = 0.45 → Non-stationary (random walk, no mean to revert to)
- **BTC returns** (daily % changes): ADF p-value = 0.02 → Stationary (mean return exists)

**AIMM context**: Test that oracle prices are stationary within regimes. Non-stationarity signals market regime change (circuit breaker trigger).

**Related**: [Stationarity](#stationarity), [Statistical Methods](#statistical-methods).

### Spread {#spread}
The difference between the highest bid and the lowest ask, representing a measure of market liquidity and the transaction cost for a trade. A narrow spread indicates high liquidity and low transaction costs; a wide spread indicates low liquidity and high transaction costs. Closely related to [Slippage](#slippage), [Price Impact](#price-impact), and [Volatility](#volatility-σ). See [Bid](#bid-price) and [Ask](#ask-price) for component definitions.

**AIMM Implementation**: AIMM's spread consists of base [Volatility](#volatility-σ) adjustment ([Vega](#vega-ν)) plus a deviation surcharge for [adverse selection](#adverse-selection) when trades worsen [coverage ratio](#coverage-ratio):
```
spread = volatilityBand + deviationSurcharge (if worsening coverage)
```

### Bid Price {#bid-price}
See [Prices](#prices).

### Ask Price {#ask-price}
See [Prices](#prices).

### Spline (Cubic Interpolation) {#spline-cubic-interpolation}
**Master entry for piecewise polynomial methods.** Splines construct smooth curves through control points ([knots](#knot-spline)) without global polynomial oscillation. AIMM uses splines to define [liquidity profiles](#liquidity-profile) with arbitrary depth shaping.

**Why Splines Matter**:
- **Smoothness**: C¹ continuous (smooth first derivative) ensures natural price curves
- **Local control**: Knots affect only nearby regions (not global shape)
- **Monotonicity**: Enforced to prevent negative depth or price reversals
- **Flexibility**: Arbitrary depth at any price level (vs fixed curves like x·y=k)

**Spline Types**:

| Method | Continuity | Oscillation | Monotone | AIMM Use |
|--------|-----------|-------------|----------|----------|
| **[Fritsch-Carlson Monotone Cubic Hermite](#fritsch-carlson-monotone-cubic-hermite-interpolation)** | C¹ | None | Enforced (sign-preservation + α²+β²≤9 clamp) | **PRIMARY** |
| **[Catmull-Rom](#catmull-rom-spline)** | C¹ | Possible | Not enforced | Rejected (non-monotone) |
| **[Makima](#makima-spline)** | C¹ | Minimal | With constraint | Alternative |

**How AIMM Uses Splines**:
```
Liquidity Profile = Spline(knots at price levels, weights for depth)
Price(tradeSize) = Traverse spline from current price
```

Allows tighter depth near market price (retail-friendly), wider depth at extremes (LP protection), and dynamic adjustment based on [coverage ratio](#coverage-ratio).

See [Knot](#knot-spline), [Liquidity Shaping](#liquidity-shaping), [Price Impact](#price-impact).

#### Fritsch-Carlson Monotone Cubic Hermite Interpolation
**Canonical AIMM spline.** Cubic Hermite interpolation with monotonicity strictly enforced via Fritsch & Carlson's 1980 algorithm ("Monotonic Piecewise Cubic Interpolation", *SIAM J. Numer. Anal.* 17(2)). Two-step construction: (1) compute tangents as averaged secants with **sign-preservation** (if adjacent secants disagree in sign, tangent = 0); (2) apply the `α² + β² ≤ 9` magnitude clamp on normalized tangents, which **guarantees** monotonicity of the cubic between knots.

**Why monotonicity matters**: Liquidity depth must be monotone in price offset from peg. A non-monotone segment would imply negative marginal liquidity over that range → broken pricing and internal arbitrage. Fritsch-Carlson is the standard solution.

**AIMM implementation**: `Spline.sol::_tangents` applies sign-preservation on adjacent secants plus the `α² + β² ≤ 9` clamp. Per-segment cubic Hermite evaluation has closed-form analytical integral, used for VWAP calculation.

**Distinction from Catmull-Rom**: Catmull-Rom uses centered secant tangents without sign-preservation or magnitude clamp → **not monotone** in general, can overshoot. Catmull-Rom was considered and rejected for AIMM.

See [Catmull-Rom Spline](#catmull-rom-spline), [Liquidity Shaping](#liquidity-shaping), [Knot](#knot-spline).

#### Makima Spline
Cubic spline interpolation designed to minimize oscillation while preserving monotonicity. Uses modified Akima-style weighted averages of neighboring slopes and naturally handles monotone data.

**Advantages over Fritsch-Carlson**: Less oscillation around knots (stable for noisy data), smoother transitions in steep regions.

**Disadvantages**: More complex computation, slightly less rigid monotonicity guarantee.

**When to use**: Fritsch-Carlson monotone cubic Hermite for exact monotone control (AIMM choice); Makima as alternative for noisy data sets with less strict monotonicity needs.

See [Fritsch-Carlson Monotone Cubic Hermite Interpolation](#fritsch-carlson-monotone-cubic-hermite-interpolation).

### Staking {#staking}
Locking tokens (BTR or LP) to earn rewards and governance rights. Subject to lock duration and cooldowns.

### Stop Orders {#stop-orders}
**Master entry for conditional orders triggered by price thresholds.** Stop orders automatically execute when a price level is reached, protecting against adverse moves or locking in profits.

**Stop Order Types**:

| Type | Trigger | Converts To | Protection | Use Case |
|------|---------|-------------|-----------|----------|
| **[Stop-Loss](#stop-loss-order)** | Price ↓ below threshold | Market order | Downside | Risk management |
| **[Stop-Limit](#stop-limit-order)** | Price ↓ below threshold | Limit order | Price control | Avoid slippage |
| **[Take Profit](#take-profit-order)** | Price ↑ above threshold | Market order | Upside lock-in | Lock gains |
| **[Trailing Stop](#trailing-stop-order)** | Price ↓ % below peak | Market order | Dynamic downside | Trend following |

**Key Mechanics**:
- **Resting order**: Stays inactive until trigger price is reached
- **Activation**: When trigger hits, converts to market/limit order
- **Execution**: Depends on order type (stop-loss = market = guaranteed fill)
- **Slippage risk**: Market orders can execute at worse prices in volatile conditions

**Trade-Offs**:

| Strategy | Certainty | Price Control | Use |
|----------|-----------|---------------|-----|
| **Stop-Loss** | High (market) | None | Emergency exit |
| **Stop-Limit** | Low (may not fill) | High | Controlled exit |
| **Take Profit** | High (market) | None | Lock gains quickly |
| **Trailing Stop** | High (dynamic) | None | Trend following |

**Risks**:
- ⚠️ **Gap risk**: If price jumps over trigger, order may not execute
- ⚠️ **Slippage**: Market orders execute at available price (can be far from trigger)
- ⚠️ **Flash crashes**: Stop orders can cascade during sharp moves

**Related**: [Limit Orders](#limit-orders), [Time-In-Force](#time-in-force-tif), [Price Impact](#price-impact).

### Stop-Loss Order {#stop-loss-order}
Order that becomes a market order when an asset's price falls to a specified stop price, automatically executing a sale to limit losses. Provides downside protection but can trigger at unfavorable prices in volatile markets.

### Stop-Limit Order {#stop-limit-order}
Conditional order combining a stop price and limit price. When the stop price is reached, the order becomes a limit order (not a market order), offering price control but no guaranteed execution if price moves past the limit.

### Take-Profit Order {#take-profit-order}
Order that becomes a market order when an asset's price rises to a specified threshold, automatically executing a sale to lock in gains. Inverse of [stop-loss](#stop-loss-order)-protects profits rather than limits losses.

**Use case**: Trader buys at $100, sets take-profit at $110. When price reaches $110, position automatically sells at current market price (may be $110 or higher). Ensures gains are captured but can miss further upside if market continues rising.

**Comparison**:
- **Stop-Loss**: Sell ↓ when price drops (downside protection)
- **Take-Profit**: Sell ↑ when price rises (upside lock-in)

See [Stop Orders](#stop-orders), [Stop-Loss Order](#stop-loss-order), [Stop-Limit Order](#stop-limit-order).

### Trailing Stop Order {#trailing-stop-order}
Dynamic stop-loss order that automatically adjusts the trigger price as the asset price rises, maintaining a fixed percentage or absolute distance below the peak price. Provides downside protection while allowing upside capture.

**How it works**:
1. Set trailing amount (e.g., 5% or $10)
2. As price rises, trigger price moves up by the trailing amount
3. If price falls back by the trailing amount, position automatically sells

**Example**:
- Buy at $100, set 5% trailing stop
- Price rises to $110, trigger moves to $104.50 (110 - 5%)
- Price rises to $120, trigger moves to $114 (120 - 5%)
- Price falls to $113, position sells at market

**Advantages**:
- ✅ Automatic upside participation (trigger chases rising prices)
- ✅ Consistent downside protection (maintain % or $ buffer)
- ✅ No manual adjustment needed

**Disadvantages**:
- ⚠️ Can trigger prematurely on small pullbacks
- ⚠️ Slippage risk (market orders execute at available price)

**Use case**: Trend-following strategies where you want to capture gains while protecting against reversals.

See [Stop Orders](#stop-orders), [Stop-Loss Order](#stop-loss-order), [Price Impact](#price-impact).

---

## T

### Tick {#tick}
Discrete price point in concentrated liquidity AMMs (Uniswap v3/v4). AIMM doesn't use ticks, prices are continuous via splines.

### Threshold Encryption {#threshold-encryption}
Cryptographic scheme where a secret (encryption key, signature) is split among multiple parties using threshold cryptography (k-of-n scheme). Any k parties can collaborate to decrypt/sign, but fewer than k cannot. Used in encrypted mempools to hide transaction content until after ordering, with block proposers holding key shares released only when ordering is final. Prevents MEV attacks but introduces complexity and potential MEV spoofing risks.

### Time-In-Force (TIF) {#time-in-force-tif}
**Master entry for order lifetime specifications.** Instructions defining when an order executes, expires, or gets cancelled. Different TIF types serve different trading strategies and risk profiles.

**TIF Types by Execution Urgency**:

| Type | Duration | Execution | Guarantee | Use Case |
|------|----------|-----------|-----------|----------|
| **[FOK](#fill-or-kill-fok)** | Instant | All-or-nothing | Complete or cancel | Block trades, high-conviction |
| **[IOC](#immediate-or-cancel-ioc)** | Instant | Partial OK | Whatever fills now | Market impact management |
| **[AON](#all-or-none-aon-order)** | Variable | Full only | Complete execution | Avoid fragmentation |
| **[GTC](#good-till-cancelled-gtc)** | Indefinite | Incremental | Patient execution | Limit orders, passive strategies |
| **[Day Order](#day-order)** | Session | Incremental | Until market close | Intraday strategies |

**Key Trade-Offs**:
- **Execution certainty**: FOK guarantees size or nothing; IOC guarantees some execution
- **Patience**: Day orders and GTC allow waiting; FOK/IOC are immediate
- **Slippage exposure**: Longer TIF = more market movement risk before full fill
- **Urgency signal**: FOK signals strong intent; GTC signals flexibility

**Practical Considerations**:
- **GTC time limits**: Most exchanges auto-cancel after 30-90 days (prevent accumulation)
- **FOK rejection rate**: High in low-liquidity assets (may not find full size)
- **IOC execution quality**: Depends on [depth](#liquidity-depth) at [bid/ask](#bid-price)

**Related**: [Limit Orders](#limit-orders), [Stop Orders](#stop-orders), [Price Impact](#price-impact), [Slippage](#slippage).


#### Fill or Kill (FOK)
[Time-In-Force](#time-in-force-tif) requiring immediate and complete execution or automatic cancellation. No partial fills, no waiting. All-or-nothing immediate execution.

**Trade-offs**: High position certainty (full or nothing) but low execution certainty (often rejected).

See [Time-In-Force](#time-in-force-tif), [IOC](#immediate-or-cancel-ioc), [AON](#all-or-none-aon-order).

#### Immediate-or-Cancel (IOC)
[Time-In-Force](#time-in-force-tif) that executes immediately for available liquidity, then cancels any unfilled portion. Partial fills allowed; no resting order.

**Trade-offs**: Fast execution with partial fills accepted, but may not achieve full size.

**Example**: IOC for 1000 tokens, market has 600 → fills 600, cancels 400.

See [Time-In-Force](#time-in-force-tif), [FOK](#fill-or-kill-fok), [AON](#all-or-none-aon-order).

#### All-or-None (AON) Order
[Time-In-Force](#time-in-force-tif) requiring complete execution of entire order or no execution. No partial fills, but can wait in order book (unlike FOK).

**Trade-offs**: Guaranteed full size if executes, but may wait indefinitely or never fill.

See [Time-In-Force](#time-in-force-tif), [FOK](#fill-or-kill-fok), [IOC](#immediate-or-cancel-ioc).

#### Good Till Cancelled (GTC)
[Time-In-Force](#time-in-force-tif) that keeps order active indefinitely until fully executed or manually canceled. Most exchanges auto-cancel after 30-90 days to prevent stale order accumulation.

**Use case**: Limit orders, passive strategies where trader is willing to wait for favorable price.

See [Time-In-Force](#time-in-force-tif), [Limit Order](#limit-order).

#### Day Order
[Time-In-Force](#time-in-force-tif) that keeps order active only during current trading session, auto-canceling at end of day if unfilled. Default on most exchanges.

**Use case**: Intraday strategies, avoiding overnight exposure.

See [Time-In-Force](#time-in-force-tif), [GTC](#good-till-cancelled-gtc).

---

### Timeboost {#timeboost}
Arbitrum's MEV auction system that allows users to bid for priority transaction ordering within blocks. Launched in late 2024 as an alternative to pure [FCFS/FIFO](#fcfs--fifo-first-come-first-served--first-in-first-out) ordering.

**How it works**:
1. **Express Lane**: A special ordering lane where winning bidders get their transactions included first
2. **Auction**: Users (typically [MEV searchers](#mev-searcher)) bid for express lane access in recurring auctions
3. **Revenue Distribution**: Auction proceeds go to the Arbitrum DAO treasury (potentially to ARB stakers)
4. **Non-Express Transactions**: Still processed via FCFS but after express lane transactions

**Contrast with**:
- [FCFS/FIFO](#fcfs--fifo-first-come-first-served--first-in-first-out): Pure arrival-time ordering (original Arbitrum)
- [Gas Auction](#gas-auction) (Ethereum L1): Priority via gas price, chaotic bidding wars
- [Jito](#jito) (Solana): Bundle auctions at validator level

**Rationale**: Rather than letting MEV leak to latency games (fast connections to sequencer) or external parties, Timeboost captures MEV value explicitly and directs it to the protocol. This makes MEV extraction transparent and economically efficient while still providing FCFS ordering for users who don't need priority.

See [Sequencer](#sequencer), [MEV](#mev-maximal-extractable-value), [FCFS/FIFO](#fcfs--fifo-first-come-first-served--first-in-first-out).

### Tinydancer {#tinydancer}
Solana [light client](#light-node) implementation verifying the [Proof of History](#proof-of-history) chain and state validity without running a full validator.

**How it works**:
1. **PoH verification**: Validates the sequential hash chain from slot leaders
2. **State proof verification**: Verifies merkle proofs for account state against signed state roots
3. **Minimal bandwidth**: Only downloads hashes and proofs, not full blocks

**Use cases**:
- Wallets verifying balances on mobile
- Off-chain applications needing trustless Solana state
- Embedded systems with limited bandwidth

**Trust model**: Same as full node, trusts that majority of validators are honest (>2/3 stake).

See [Light Node](#light-node), [Proof of History](#proof-of-history), [Gulf Stream](#gulf-stream).

### Timelock {#timelock}
Delay before sensitive operations execute:
- CRITICAL (7 days): Base token migration
- HIGH (3 days): Ownership transfer
- BASE (2 days): Oracle updates
- LOW (1 day): Add asset, fee changes

### Transaction Relay Proxy (TRP) {#transaction-relay-proxy-trp}
Specialized infrastructure routing transactions from users to appropriate endpoints for execution. TRPs provide MEV protection by sending transactions to private mempools or MEV-resistant RPC providers instead of public mempools where searchers observe them. Examples include MEV-resistant relays that encrypt transaction details until block inclusion.

### Toxic Flow {#toxic-flow}
Trades exploiting information asymmetry, the counterparty knows something the pool doesn't. Toxic flow is unidirectional, driven by CEX price leads, and extracts value even if prices revert (the pool loses on both the up-move and the down-move). Contrast with **rebalancing arbitrage**, which brings pool prices back to market and is net-positive for liquidity. AIMM charges deviation surcharge on coverage-worsening trades during high oracle disagreement, making toxic flow expensive while keeping rebalancing cheap.

### Transient Storage {#transient-storage}
See [EIP-1153](#eip-1153-transient-storage).

### Treasury {#treasury}
Protocol-controlled address that can collect accumulated protocol fees. Subject to timelock for updates.

---

## U

### Undercollateralized {#undercollateralized}
Coverage < 100%. Pool owes more than it holds. Triggers:
- Withdrawal haircuts
- Positive inventory skew (premium)
- Potential liability decay

### UUPS (Universal Upgradeable Proxy Standard) {#uups-universal-upgradeable-proxy-standard}
Upgrade pattern where logic contract contains upgrade function. Used by Treasury and Bridge contracts.

---

## V

### Vega (ν) {#vega}
Per-asset sensitivity parameter controlling how volatility affects both the base spread and [liquidity dispersion](#dispersion). **Tuning guidance**:
- **5000 (0.5x)**: Gentle volatility response (stablecoins, low-vol assets)
- **10000 (1.0x)**: Linear response (default, balanced)
- **20000 (2.0x)**: Aggressive spread widening during volatility (volatile assets, high beta)

**Formulas**:
- `volatilityBand = 100 + σ × vega / (100 × 10000)` (spread adjustment)
- `dispersion = clamp(baseDispersion + σ × vega / MULT_BASE, minDisp, maxDisp)` (depth curve adjustment)

**Economic role**: Vega implements **volatility sensitivity** from [Avellaneda-Stoikov](#avellaneda-stoikov-framework). Higher vega means the pool widens spreads and concentrates liquidity when volatility spikes. **Joint effect with Gamma & Lambda**: Vega affects spread directly via volatility, while gamma affects spread via inventory skew, and lambda via price uncertainty. Together they form the **tri-factor fee model** (volatility + inventory + deviation). See [Volatility (σ)](#volatility-σ), [Spread](#spread), [Dispersion](#dispersion), and `docs/1. AIMM/1.2. Pricing/1.2.5. Parametrization.md` for implementation.

### Volatility (σ) {#volatility}
Price variability measurement. AIMM tracks via dual EMAs:
- Fast EMA: α = 0.02% (200 in 1e6)
- Slow EMA: α = 0.18% (1800 in 1e6)

### Validium {#validium}
Layer 2 scaling solution using cryptographic proofs (like zk-rollups) but storing data off-chain instead of on L1. Reduces L1 data overhead at the cost of relying on a data availability committee or external storage. Faster and cheaper than zk-rollups but lower security guarantees for data availability.

### Validator {#validator}
Network participant who stakes cryptocurrency as collateral to participate in [consensus](#consensus) and block production. Validators earn rewards for honest participation but face penalties ([slashing](#slashing)) for misbehavior.

**Validator duties** (chain-dependent):

| Duty | Purpose | Reward | Penalty |
|------|---------|--------|---------|
| **Block proposal** | Create new blocks | Base reward + MEV | Missed opportunity |
| **Block attestation** | Sign validity of blocks | Full reward | Slashing if wrong |
| **Finality** | Vote on checkpoints | Included in reward | Slashing if equivocate |

**Ethereum validators**:
- **Requirement**: 32 ETH stake
- **Software**: [Consensus client](#consensus-client) ([Lighthouse](#lighthouse), [Prysm](#prysm), etc.) + [Execution client](#execution-client) ([Geth](#geth), [Nethermind](#nethermind), etc.)
- **Rewards**: ~3-4% APY + MEV tips
- **Penalties**: Up to 100% stake loss for egregious misbehavior

**Solana validators**:
- **Software**: [Agave](#agave) or [Jito-Solana](#jito) for MEV, or [Firedancer](#firedancer) for next-gen
- **Participation**: Vote on [Tower BFT](#tower-bft) fork choice
- **Rewards**: Transaction fees + MEV (via [Jito](#jito) bundles)

**Sui validators**:
- **Software**: [Sui Core](#sui-core)
- **Duties**: [Narwhal](#narwhal) committee member + [Mysticeti](#mysticeti) voter
- **Rewards**: Transaction fees + commission on delegated stake

**MEV considerations**:
- **Ethereum**: Validators delegate block construction to [block builders](#block-builder) via MEV-Boost, retaining proposal rights
- **Solana**: Validators directly control block composition with [Jito](#jito) MEV auctions
- **Arbitrum/Base**: [Sequencer](#sequencer) controls ordering, validators verify/finalize on L1

See [Consensus](#consensus), [Block Builder](#block-builder), [Sequencer](#sequencer), [Slashing](#slashing), [Finality](#finality).

### Validator Set {#validator-set}
The collection of all active validators in a Proof of Stake network at a given time, responsible for proposing blocks, attesting to canonical chain, and maintaining finality. Validator set composition can change based on stake and protocol rules. The distributed validator set provides security through consensus.

### Voting Power {#voting-power}
Governance influence calculated from staked positions:
```
V = (L × lpBase/10000) + (min(boosted, boostCap × G) × lpBoosted/10000) + (G × govMultiplier/10000)
```
Where L = LP stake, G = governance stake.

---

## W

### WAD {#wad}
Fixed-point unit = 1e18. Used for coverage ratios, prices, and percentages requiring high precision.

### Wrapped Native {#wrapped-native}
ERC-20 version of native token (WETH for ETH). AIMM auto-wraps/unwraps for native token support.

### Weight (Spline) {#weight-spline}
Liquidity concentration in profile segment. Higher weight = more depth in that price range. Weights must sum to 200.

---

## Z

### Zero-Knowledge Proof (ZKP) {#zero-knowledge-proof-zkp}
Cryptographic method where a prover can prove a statement is true to a verifier without revealing any information beyond the statement's validity. **Implementations**: [ZK-SNARKs](#zk-snarks-zero-knowledge-succinct-non-interactive-arguments-of-knowledge) (succinct, requires setup), [ZK-STARKs](#zk-starks-zero-knowledge-scalable-transparent-arguments-of-knowledge) (transparent, no setup). **Applications**: [Confidential Transfer](#confidential-transfer) (hide amounts), [Mixer](#mixer) (break transaction links via [Commitment](#commitment) + [Merkle Proof](#merkle-proof)), [Encrypted Mempool](#encrypted-mempool) (MEV resistance), [ZK Compression](#zk-compression) (state optimization). In blockchains, ZKPs hide sender, recipient, or transaction amount while proving validity. Foundation for privacy protocols and scaling solutions. Related: [Bulletproof](#bulletproof), [Hashing](#hashing), [Nullifier](#nullifier).

### ZK-SNARKs (Zero-Knowledge Succinct Non-Interactive Arguments of Knowledge) {#zk-snarks-zero-knowledge-succinct-non-interactive-arguments-of-knowledge}
Specific zero-knowledge proof system providing succinct, non-interactive proofs. Requires trusted setup but produces small proofs. Used in privacy protocols and zk-rollups for Layer 2 scaling. Examples: Zcash, Aztec, some rollup implementations.

### ZK-STARKs (Zero-Knowledge Scalable Transparent Arguments of Knowledge) {#zk-starks-zero-knowledge-scalable-transparent-arguments-of-knowledge}
Zero-knowledge proof system requiring no trusted setup (transparent), producing slightly larger proofs than SNARKs but with stronger security assumptions. Used in Layer 2 solutions and privacy protocols seeking to avoid trusted setup complexity.

### ZK Compression {#zk-compression}
Technique using zero-knowledge proofs to compress blockchain state, significantly reducing storage costs. Used in Solana and other chains for state optimization. While primarily a scalability feature, enables more cost-effective privacy-preserving applications by reducing on-chain footprint.

---

## ALM / Vault Terms

> Definitions specific to BTR Vaults (`alm/evm/`). Cross-references to the dedicated docs in `/7. Vaults/`.

### ALM (Adaptive Liquidity Management) {#alm-adaptive-liquidity-management}
BTR product surface that allocates single-asset deposits to external concentrated-liquidity pools (UniV3, UniV4, PancakeV3, PancakeInfinity, Algebra, Ramses; Aerodrome Slipstream via the UniV3 adapter), and, forward-looking, BTR's own AIMM pools. Implemented in `alm/evm/src/`. Distinct from the existing TradFi-borrowed [ALM (Asset-Liability Management)](#alm-asset-liability-management) entry which describes the Platypus-style accounting framework. See [Vaults Overview](/docs/7-Vaults). **BTR Supply** = BTR's ALM product, Phase 1 launch product, vault-based, allocates across UniV3, UniV4, PancakeV3, PancakeInfinity, Algebra, Ramses + (Phase 2) AIMM.

### HWM (High-Water Mark) {#hwm-high-water-mark}
Vault-level pair `(hwmAssets, hwmSupply)` against which performance is measured. Updated **only** in `harvestPerformance()` (`Vault.sol:harvestPerformance`, ≈ L441); never in `_accrue`. Equivalent to comparing share prices `ta/sup > hwmAssets/hwmSupply` while preserving integer-math precision via the cross-multiplied form `ta × hS > hA × sup`. Performance fee is charged on the strictly positive `gain = ta − hwmAssets × sup / hS`.

### LVR (Loss Versus Rebalancing) {#lvr-loss-versus-rebalancing}
Structural shortfall of a passive AMM LP relative to a perfectly-rebalanced book that quotes the same prices. Approximation: `LVR_rate ≈ (σ² / 8) × L_active` per unit time. Drives most of the unmanaged-LP underperformance; BTR Vaults attack it via tighter ranges, adaptive rebalances, and worst-of marks. See [Risk Model §2](/docs/7.4-Risk-Model).

### IL (Impermanent Loss) {#il-impermanent-loss}
Divergence between holding LP shares and holding the deposit asset directly when price moves through (or out of) the position range. CLMM-specific: out-of-range crystallizes IL into a single-sided holding at the boundary tick. BTR Vaults do not eliminate IL; they bound and surface it through pessimistic share price.

### Pessimistic Worst-Of Mark {#pessimistic-worst-of-mark}
NAV / `assetValue` rule used by every Dex adapter. Given LP-implied amounts `(amt0, amt1)`, the side opposite the requested denomination is converted via **`min(oracle, pool)`**. Closes both JIT-pool-pump and stale-oracle attack surfaces simultaneously. See `Dex.assetValue` (`Dex.sol:assetValue`, ≈ L454) and [NAV & Fees §1](/docs/7.3-NAV-Fees).

### TDWAP (Time-Decayed Weighted Average Price) {#tdwap-time-decayed-weighted-average-price}
Internal mid-price construction used by AIMM's [Internal Oracle](#internal-oracle). Distinct from the Vault's pessimistic worst-of mark, the Vault uses external Chainlink USD feeds (`PriceProvider`) plus the live pool `slot0()`, not TDWAP.

### principalOf-as-Shares {#principalof-as-shares}
Semantics of the `Dex.principalOf[holder]` mapping: the value is the holder's **internal LP-share balance** in the adapter, not raw cost basis. `pull` mints shares pro-rata to current marked NAV; `withdraw` burns shares rounded up. This makes late-joiners non-dilutive to incumbent PnL. See `Dex.sol:principalOf` ≈ L121, `:pull` ≈ L355, `:withdraw` ≈ L380.

### Intent (ERC-7683) {#intent-erc-7683}
A swap order escrowed at a settler contract that fills asynchronously off-chain and settles back via `settleIntent`. Used by `Dex.rebalance` for routes whose immediate execution would be too costly. While an intent is open on `tokenIn` or `tokenOut`, the adapter is treated as transient-frozen, share-mint paths gate (`Dex._checkLive`), the vault's sync deposit/redeem disable, and the async ticket path remains open.

### Atomic Route {#atomic-route}
A swap route executed synchronously through the Swapper's pinned LiFi router within a single transaction. Contrasts with **Intent** routes. The post-mint `maxSlipBps` envelope only applies when **all** routes in a `rebalance` call are atomic, mixed sets defer to per-route `minOut` + per-intent enforcement at settle. See `Dex.sol:rebalance` (atomic-vs-intent branch), ≈ L402.

### ERC-4626 {#erc-4626}
Vault standard for tokenized yield-bearing shares. BTR Vault is strict-ERC-4626 on the deposit / mint / sync withdraw / sync redeem surface. `withdraw` reverts `WithdrawAsync` rather than queue a ticket, for strict integrators (Pendle, Morpho, Euler).

### ERC-7540 {#erc-7540}
Async-redeem extension to ERC-4626. BTR Vault implements it as the **fallback** for `redeem` when the vault is unhealthy or idle is insufficient: `requestRedeem` enqueues a controller-keyed ticket; `processWithdrawals(controllers[])` settles the cohort with a uniform haircut; `claimRedeem(controller, receiver)` pays out. ERC-7540 operator approvals (`isOperator`) are first-class.

### Exit-Fee Floor {#exit-fee-floor}
Invariant `exitPip ≥ 1.2 × max_i(deviationBps_i + heartbeatDriftBps_i)` (in pip; `1 bps = 10 pip`). Enforced at `Vault.setFees` write time via `_exitFloor()` (`Vault.sol:_exitFloor`, ≈ L650; floor body @ `VaultAux.sol:exitFloor`). Prevents an MEV searcher from extracting up to `(deviation + heartbeat-drift)` bps of free value through a deviating exit window. The 1.2× factor leaves a 20% margin.

### Pip (Vault fees) {#pip-vault-fees}
Fee unit used by the Vault. `PIP_BASE = 100_000`, so `100 pip = 0.1%`, `1 bps = 10 pip`. Applies to `exitPip`, `mgmtPip`, `perfPip`. Allows 0.001% precision; max representable is 65.535%.

### Cohort (Async Redeem) {#cohort-async-redeem}
Batch of pending ERC-7540 redeem tickets settled together by `processWithdrawals(controllers[])`. The cohort is summed, idle is checked, and each ticket is haircut by a uniform `hcBps`. Controllers must be passed sorted ascending (uniqueness enforced, `VaultAux.sol:processWithdrawals`, ≈ L45) so duplicate-controller `needed` cannot inflate.

### Claim-by-Balance Haircut {#claim-by-balance-haircut}
Final per-claim cap at `claimRedeem` time: when the vault contract balance is below `claimableAssets` (e.g., due to a competing cohort), the per-claim payout is further capped at `bal × cl.assets / claimableAssets`. Combined with the cohort `hcBps`, produces fair regime-stable settlement (`VaultAux.sol:claimRedeemBody`, ≈ L91; `Vault.sol:claimRedeem` ≈ L363).

### Defensive Ratchet {#defensive-ratchet}
Pattern for keeper-permitted parameter direction: keeper can only move a knob in the safer direction, `Vault.depositCap` ↓ only (≥ totalAssets), `Dex.maxSlipBps` ↓ only, `Dex.minIntervalSecs` ↑ only (capped at `+1 day` per call). Owner is the only authority that can loosen. Owner-loosen of `maxSlipBps` is timelocked (1 day) via `queueSlip` since Pass-13 P13-7; owner cannot bypass via `setConfig` since Pass-15 P15-1 (instant `setConfig` widen now reverts; only narrow path remains instant). Pass-19 P19-1 tightened the adapter-side `queueSlip` cap from 900 bps to 700 bps (`ADAPTER_SLIP_CAP_BPS = JOINT_SLIP_CAP_BPS - ADAPTER_SLIP_HEADROOM_BPS`), sizing the 300 bps headroom to cover `Vault.MAX_EXIT_PIP = 3_000 pip = 300 bps = 3%` (Pass-21 P21-1 corrected the constant value from `30_000` → `3_000`; the prior value was off by 10× and would have soft-DoS'd `Vault._assertJointSlip` for any owner-permitted `exitPip > 700`) so no owner-permitted `exitPip` can soft-DoS `Vault._assertJointSlip`.

### Kill Switch (per adapter) {#kill-switch-per-adapter}
Per-adapter `killed` flag (`Dex.sol:killed` ≈ L81). Owner kill is instant + revertible only via `queueUnkill` + `executeUnkill` (`FREEZE_TIMELOCK = 1 day`, hoisted to `Constants.sol:FREEZE_TIMELOCK` in Pass-29 P29-1). Keeper kill is rate-limited to 1/h per adapter and charged against `AccessControl.KEEPER_KILL_DAILY_CAP = 2` (cluster-wide UTC-day cap). Once killed, `assetValue` reverts and vault `_healthy()` is false. Symmetric across `BtrPoolAdapter.sol` (queueUnkill/executeUnkill + Pass-9 P9-3 unkillEta-zeroing; Pass-11 P11-1 gates the wipe to owner-only on Dex.sol to prevent compromised-keeper grief). Pass-13 P13-7 adds timelocked owner-loosening of `maxSlipBps` via `queueSlip`/`executeSlip` on both adapters; keeper ratchet-down via `setConfig` unchanged. Pass-15 P15-1 removed the owner-widen path from `setConfig` entirely (any widen reverts NotAuth and must be staged through `queueSlip`); Pass-15 P15-2 mirrors `JOINT_SLIP_CAP_BPS` adapter-side so a queued value can never exceed the runtime joint-slip cap. Pass-19 P19-1 tightened the adapter-side `queueSlip` cap from 900 to 700 bps to cover `Vault.MAX_EXIT_PIP = 3_000 pip = 300 bps` headroom (Pass-21 P21-1 corrected the constant `30_000` → `3_000`); also fixed `BtrPoolAdapter.executeSlip` which still compared against `JOINT_SLIP_CAP_BPS` (now mirrors `Dex.sol` and uses `ADAPTER_SLIP_CAP_BPS`).

### Sequencer Guard {#sequencer-guard}
`PriceProvider._checkSequencer()` consults the Chainlink L2 sequencer-up feed. Sequencer down OR within `SEQ_GRACE = 1 hour` of recovery ⇒ all `getPrice` calls revert ⇒ adapters report frozen ⇒ vault gates sync deposit / redeem. See [Risk Model §4](/docs/7.4-Risk-Model).

### Adapter Deregistration {#adapter-deregistration}
Owner-only `AccessControl.deregisterAdapter(a)` (`AccessControl.sol:deregisterAdapter` ≈ L105) flips `isAdapter[a]` off. Vaults using that adapter become unhealthy on the next `_healthy()` check. The owner subsequently calls `Vault.removeAdapter(a)` to drain accounting state, permitted because the deregistered status overrides the `lastReported != 0` busy check. The escape hatch for a compromised or buggy adapter clone (`Vault.sol:removeAdapter`, ≈ L503).

### Two-Step Ownership {#two-step-ownership}
`AccessControl` overrides Solady's 1-step `transferOwnership` and `renounceOwnership` to revert. Handover requires `requestOwnershipHandover` then `completeOwnershipHandover` with a non-zero pending owner. Closes the back-door renounce vector (`AccessControl.sol:transferOwnership` ≈ L68, `:renounceOwnership` ≈ L69, `:completeOwnershipHandover` ≈ L72).

### Wedge (setFees) {#wedge-setfees}
Forward-compat hazard where `Vault.setFees` has an **empty admissible range** for `exitPip`. Triggered when adapter risk knobs satisfy `(deviationBps + heartbeatDriftBps) × 12 > MAX_EXIT_PIP = 3_000`. Cause: `_exitFloor()` returns a value above `MAX_EXIT_PIP`, so the simultaneous bounds `exitPip ≥ floor` AND `exitPip ≤ MAX_EXIT_PIP` are unsatisfiable; every `setFees` call reverts `ThresholdViolation` until knobs are tightened. Aggregate cap therefore: `(dev + hb) ≤ 250 bps`. Recovery: `queueRisk` lower `deviationBps`/`heartbeatDriftBps` on the offending adapter(s) (timelocked 1 day), or coordinated protocol upgrade raising `Constants.MAX_EXIT_PIP` (with matching `Constants.ADAPTER_SLIP_HEADROOM_BPS` bump per the headroom invariant). Cite Pass-23 P23-5.

### Exact-fit (P21-1 headroom) {#exact-fit-p21-1-headroom}
Tight equality `Constants.ADAPTER_SLIP_HEADROOM_BPS = MAX_EXIT_PIP / 10 = 300 bps`. Equivalent to `ADAPTER_SLIP_CAP_BPS + MAX_EXIT_PIP / 10 = JOINT_SLIP_CAP_BPS` (`700 + 300 = 1000`). The 300 bps adapter-side headroom is **exactly** the worst-case `exitBps` term so the joint envelope `adapter.maxSlipBps + exitPip/10 ≤ JOINT_SLIP_CAP_BPS` is saturable but not loose. Before Pass-21 P21-1, `MAX_EXIT_PIP` was the typo'd `30_000` (10× over) which left headroom 10× under-sized and would have soft-DoS'd `Vault._assertJointSlip` for any reasonable `exitPip`. Pass-19 P19-1 sized the headroom; Pass-21 P21-1 corrected the constant; Pass-29 P29-4 added compile-time invariant guards (`_GUARD_HEADROOM`, `_GUARD_CAP`) in `Constants.sol` that fail-compile if either invariant is ever broken.

### Worst-Of (alias) {#worst-of-alias}
Synonym for [Pessimistic Worst-Of Mark](#pessimistic-worst-of-mark) when used as an adjective ("worst-of price", "worst-of NAV").

---

## Related Documentation

- [AMM Landscape](/docs/concepts/amm-landscape) -BTR DEX vs peer AMM comparison (Uniswap V2/V3/V4, Curve V1/V2, Balancer, DODO, Maverick, LFJ LB, Platypus, Wombat, OrbSwap)
- [Capital Efficiency](/docs/concepts/capital-efficiency) -Shared-inventory multiplier (VCR, `N-1`× hub-pair depth, contention discount γ)
- [Foundations §10 OrbSwap](/docs/concepts/foundations) -Sphere invariant, polar-tick mathematics, torus extension
- [Architecture Overview](/docs/overview-aimm)
- [Inventory Management](/docs/1.1.1-Inventory-Management)
- [Spread & Fees](/docs/1.1.4-Spread-&-Fees)
- [Parameters Reference](/docs/1.1.7-Parametrization)
- [Vaults Overview](/docs/7-Vaults)
- [Vault Architecture](/docs/7.1-Architecture)
- [Vault NAV & Fees](/docs/7.3-NAV-Fees)
- [Vault Risk Model](/docs/7.4-Risk-Model)

---

## Documentation Conventions

### Placeholder / "coming soon" pattern

When a page is pre-launch:
> ⚠ **Pre-launch — content pending.** [Specific reason: screenshots / data / audit finalization]. In the meantime, see [alternative resource].

When a page is gated to Phase 2:
> 🚧 **Phase 2 — coming with BTR DEX launch.** [Brief teaser]. In the meantime, see [Phase-1 alternative].
