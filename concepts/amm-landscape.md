---
title: "AMM Landscape: Where BTR DEX Fits"
description: "Canonical comparison of BTR DEX (AIMM) against major peer AMMs. Honest, technically precise, no marketing inflation. For theoretical lineage see Foundations"
audience: both
type: explanation
status: live
phase: n/a
order: 5
lang: en
publish: true
---
# AMM Landscape: Where BTR DEX Fits

> Canonical comparison of BTR DEX (AIMM) against major peer AMMs. Honest, technically precise, no marketing inflation. For theoretical lineage see [Foundations](/docs/concepts/foundations); for positioning narrative see [Manifesto](/docs/concepts/manifesto).

---

## 1. Comparison Matrix

Legend: **CE** = capital efficiency at peg / in-distribution. **LVR** = loss-versus-rebalancing exposure. **Async** = whether trade splitting changes total fill (path-dependence). **+Asset** = marginal cost to add an Nth asset. **Anchor** = oracle / reference price dependency.

| Protocol | Max assets / pool (practical) | Asset class fit | CE (in-dist.) | Path-indep. | Composability (in-contract) | LVR exposure | Oracle dep. | MEV surface | LP onboarding | Regime adaptation | +Asset cost | Depeg robustness | Permissioning | TVL / age | Gas / swap (typical) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **Uniswap V2** [^1] | 2 | any | Low (full-range) | Yes | Cross-contract hops | High (constant fee, no skew) | None | Sandwich, JIT-light | Trivial (deposit ratio) | None (immutable curve) | Deploy new pair | Isolated (peg-agnostic) | Permissionless | $1B+ / 5y / heavily audited | ~100k |
| **Uniswap V3** [^2] | 2 | any (best volatile) | High in-range, 0 out-of-range | Yes | Cross-contract hops, tick crossings within pool | Per-tick LVR; JIT-exposed without hook mitigation. | None | Sandwich, JIT-pro, tick-grief | Range selection (hard for retail) | Per-LP manual rebalance | Deploy new pool + fee tier | Isolated | Permissionless | $4B+ / 4y / battle-tested | ~120-200k |
| **Uniswap V4** [^3] | 2 (per pool; singleton vault) | any | Same as V3 + hook-extensible | Yes (V3 math) | Singleton PoolManager → flash accounting, multi-hop in one tx | Same as V3 unless hook mitigates | Optional (hook-defined) | Same as V3 + hook-specific | Hook complexity adds cognitive load | Hook-driven (custom curves, dynamic fees) | Deploy new pool (cheap clone, no factory deploy) | Isolated | Permissionless + hook curation | $1B+ TVL across hooks / 1y / audited | ~80-150k (singleton) |
| **Curve V1 Stableswap** [^4] | 2-4 (8 max, rare) | stable peg only | Very high near peg | Yes | Cross-pool via metapool / router | Low at peg, high off-peg | None | Mild sandwich, depeg cascades | Multi-asset deposit (proportional) | A coef governance-rotated (slow) | Deploy new pool + gauge | Joint invariant amplifies depeg loss across all assets | Permissionless + curated gauges | $2B+ / 5y / audited, depeg-tested (UST, USDC) | ~150-250k |
| **Curve V2 Cryptoswap** [^5] | 3 typical (tri-crypto) | volatile mixed | High near EMA peg | Yes | Cross-pool router | High during repeg events | Internal EMA oracle | Repeg MEV, donation attacks | Tri-asset proportional | Internal EMA repegging + admin γ/A | Deploy new pool | Permanent loss possible on repeg | Permissionless | $500M+ / 3y / audited | ~250-400k (Newton iteration) |
| **Balancer V2/V3** [^6] | 8 (weighted), 5 (composable stable) | mixed weighted | Moderate (weighted geomean) | Yes | Singleton Vault → batch swaps, flash loans | High (constant weights, LVR per pair) | None (V2) / optional hooks (V3) | Sandwich, weight-arb | Weight selection (deployer) | None (V2) / hook-driven (V3) | Deploy new pool, no `addAsset` | Weighted absorbs depeg proportionally | Permissionless | $800M+ / 4y / audited | ~150-300k |
| **DODO V1/V2 (PMM)** [^7] | 2 | any (stable + volatile) | High (concentrated around mid) | Yes | Per-pair contracts | Moderate (oracle-anchored mid) | Hard (Chainlink) | Oracle stale-update MEV | i (mid) + k (slip) per pool | Static; admin re-parametrize | Deploy new pool | Hard halt if oracle deviates | Curated PMM, permissionless classical | $50M+ / 4y / audited | ~120-180k |
| **Maverick V1/V2** [^8] | 2 | volatile (directional bias) | Very high (mode-based) | Yes (binned) | Per-pool | Lower than V3 (auto-shift bins) | None | Sandwich, mode-flip arb | Mode choice (Static/Right/Left/Both) | Mode-driven auto-rebalance per swap | Deploy new pool | Isolated | Permissionless | $100M+ / 2y / audited | ~150-250k |
| **LFJ Liquidity Book (formerly Trader Joe v2)** [^9] | 2 | any (volatile-tuned) | Very high (bin = local CSMM) | Yes (bin traversal) | Per-pair, bin-routed | Lower (variable fee surge) | None | Bin-jump MEV, surge-fee gaming | Bin selection + fungible per-bin tokens | Variable fee accumulator (volatility-reactive) | Deploy new pool | Isolated | Permissionless | $50M+ / 2y / audited | ~150-300k (bin traversal) |
| **Platypus** [^10] | 5-15 (multi-stable singleton) | stable only | Very high (coverage-anchored) | Yes | Singleton → all-to-all stable swaps | Low at coverage=1, high under-covered | Hard (Chainlink) | Oracle MEV, coverage-arb (Feb 2023 ~$8.5M `emergencyWithdraw` exploit) | `addAsset` call (cheap) | Static slip fn, governance params | `addAsset` call (~cheap) | Coverage-aware haircut, but joint solvency risk if one asset insolvent (Platypus exploit demonstrated) | Curated add-asset | $30M (post-exploit) / 3y / audited+exploited | ~100-150k |
| **Wombat** [^11] | 5-15 (multi-stable singleton) | stable + LSTs | Very high | Yes | Singleton, all-to-all | Same as Platypus | Hard | Same family | `addAsset` call | Static, governance | `addAsset` call | Asset-isolated within shared pool (better than Platypus joint) | Curated | $80M+ / 3y / audited | ~100-150k |
| **OrbSwap (CCMM)** [^12] | N (theoretically thousands, ~10-50 practical) | stable / pegged | Very high in polar tick (concentrated around peg) | Yes | Singleton orbital pool | Bounded by polar tick width | None (math-isolated) | Sandwich within tick | Polar tick selection (3D analog of V3 range) | Static curve, immutable | `addAsset` (extends sphere dimension) | Math-isolated: one depeg leaves orbital surface, others unaffected | Permissionless | Early / 2025 / pre-audit | High (n-dim math) |
| **BTR DEX (AIMM)** | 5-15 (blue-chip curated singleton; anchor-tree) | stable + LST + blue-chip volatile, mixed | High (spline-shaped depth, volatility-scaled dispersion) | Yes | **Anchor-tree LCA routing**, all swaps in one pool, internal hops free | **Priced explicitly** via two-factor fees (vol + momentum) + inventory skew on mid | HYBRID: internal multi-TF TWAP primary for quoting + Chainlink on base-token only as depeg circuit-breaker (not a primary price source) | Shrunk by dynamic fees + (planned) Cooperative Arbitrage rebates | Single-sided deposit, no range/weight/knot choice | **Algorithmic** (volatility EMA → dispersion; coverage → skew) + admin-mutable spline w/ timelock | `addAsset` call (anchor-tree node insert) | Per-asset depeg halt + coverage haircut + spline edge slip cap; **does not amplify across pool** | Curated blue-chip whitelist, permissionless white-label clones | New / 2026 / pre-launch audit | ~150-250k target (V4-competitive) |

---

## 1.5. BTR's Dual Role: Peer (AIMM) + Meta-LP (Supply)

BTR appears in this landscape twice, in two distinct roles:

| BTR Surface | Role | Launch Phase | Relation to peer AMMs in §1 |
|-------------|------|--------------|------------------------------|
| **BTR DEX (AIMM)** | Peer AMM. Curated multi-asset singleton with anchor-tree LCA routing, inventory-aware mid, spline depth. | **Phase 2 (Future)** | Direct peer in the §1 matrix — compared head-to-head with Uniswap, Curve, Balancer, etc. |
| **BTR Supply (ALM)** | **Not a peer AMM**. Meta-LP layer. Single-asset vaults that allocate capital into UniV3, UniV4, PancakeV3, PancakeInfinity, Algebra, Ramses + (Phase 2) AIMM. | **Phase 1 (Live)** | Sits **on top of** every CL DEX in the §1 matrix; consumes their pools as liquidity venues via adapters. |

The §1 matrix row labeled "**BTR DEX (AIMM)**" describes the peer-AMM surface only. **BTR Supply (ALM)** is not a column entry there because it does not have its own pricing curve — it inherits whichever curve the underlying adapter exposes.

---

## 2. Per-Peer Summaries

**Uniswap V2** [^1] — `x·y=k` two-asset CPMM, full-range LP. Trivially path-independent, gas-cheap, zero parameter risk. Capital-inefficient (most TVL idle far from spot), high unpriced LVR. Still dominant for long-tail pairs because deployment friction is zero.

**Uniswap V3** [^2] — Tick-based concentrated liquidity. LP nominates a price range; in-range behaves as local CPMM. Massive in-range CE, but out-of-range = 0 active liquidity; manual rebalance; JIT bots professionalized fee extraction.

**Uniswap V4** [^3] — Singleton `PoolManager` + hooks + flash accounting + native ETH. Pool math is V3; the delta is (a) one contract holds all pool state → cheaper multi-hop / flash loans, (b) hooks can rewrite fee logic and even curves. **Per-pool logic is still pair-isolated**: no shared inventory across pairs in the singleton. Hooks fragment liquidity by configuration.

**Curve V1 (Stableswap)** [^4] — Hybrid invariant flattens to constant-sum near peg, bends to constant-product far from it; `A` controls flat-region width. Up to 8 assets, 2-4 typical. Joint invariant: one asset depeg drains the pool across all assets to varying degrees (UST May 2022 — pool fully cratered; USDC March 2023 SVB — 3pool drained but recovered within days).

**Curve V2 (Cryptoswap)** [^5] — Stableswap + EMA price oracle + dynamic repeg governed by `γ` damping + PnL accounting. Tri-volatile around a slowly-drifting internal anchor. Capital-efficient near EMA peg; off-peg execution poor; repeg events incur permanent loss.

**Balancer V2/V3** [^6] — Weighted geomean invariant up to 8 assets, plus stable / composable-stable variants. V2's Vault is a singleton storage layer; per-pool math is still pairwise (no shared liquidity across pools in the Vault). V3 adds hooks.

**DODO V1/V2 (PMM)** [^7] — Proactive Market Maker uses Chainlink anchor `i` + slip coef `k`. Accuracy depends on oracle freshness. `k=0` → CSMM, `k=1` → CPMM. Parameters static post-deploy; no regime adaptation.

**Maverick V1/V2** [^8] — Two-asset CL with **mode-based auto-rebalance**: Static (V3-like), Right/Left (bins trail directional moves), Both (symmetric trail). Pool itself shifts LP bins on each swap, removing manual rebalance. Pair-isolated.

**LFJ Liquidity Book** [^9] — Discrete price bins, each bin a local CSMM. LP positions fungible per-bin. Variable fee accumulator surges spreads under realized vol. Halfway between V3 ticks and an order book.

**Platypus** [^10] — Singleton stable AMM with **coverage ratio** (`assets/liabilities`) per token driving slippage. Single-sided deposits, `addAsset` cheap. Original ALM AMM. Feb 16 2023 ~$8.5M flash-loan exploit — root cause: `emergencyWithdraw` in `MasterPlatypus` did not check outstanding USP debt, allowing the attacker to withdraw collateral while leaving USP minted against it. Coverage-mechanism arbitrage demonstrated joint-solvency risk.

**Wombat** [^11] — Platypus fork with per-asset isolation: each asset has its own coverage curve, ringfencing per-asset solvency. Same coverage-slippage family; extended to LSTs.

**OrbSwap (CCMM, Orbital)** [^12] — Polar-coordinate / n-sphere AMM (Paradigm research). Pegged assets sit on an n-dim sphere; per-asset "polar tick" lets one asset exit the sphere on depeg without dragging the rest, giving **mathematical isolation**. Pre-production; verification cost scales with N.

**BTR DEX (AIMM)** — Curated **multi-asset singleton** (5-15 blue-chip target: stables, LSTs, ETH, BTC, majors). Pipeline: (1) internal multi-TF TWAP → reference; (2) coverage-based **inventory skew** adjusts mid (inspired by Platypus + Avellaneda-Stoikov); (3) **spline-shaped depth** (monotone cubic Hermite, Fritsch-Carlson) with vol-scaled dispersion for market impact; (4) two-factor fees `ν·σ + λ·Δ` (vol + momentum) around adjusted mid; (5) **anchor-tree LCA routing** — all assets quote vs anchors, root in base numéraire (O(1) maintenance vs Curve V2 / Wombat O(N²)). **HYBRID oracle**: internal TWAP primary for quoting + Chainlink on base-token only as depeg circuit-breaker (not a primary price source). **Capital-efficiency multiplier**: one USDC unit counterparties N pairs simultaneously via shared inventory hub; depth-per-dollar scales with asset count instead of fragmenting across N(N-1)/2 pairs. Regime adaptation is **algorithmic** (dispersion ← σ; skew ← coverage) plus **admin-mutable spline** under timelock. `addAsset` inserts an anchor-tree node in one call.

---

## 3. Where BTR Wins, Where BTR Loses (Honest)

**BTR's niche: blue-chip multi-asset singleton with regime-adaptive policy.**

- **Wins on multi-hop / composability**: anchor-tree LCA routing executes USDC→stETH, DAI→WBTC etc. in one swap inside one contract — no router fanout, no per-hop slip stacking. Uniswap V4's singleton lowers gas but still traverses pair-pools sequentially. Anchor tree collapses N(N-1)/2 pair routing to ≤2 anchor hops.
- **Wins on add-asset cost**: `addAsset` call (Platypus/Wombat lineage) vs. full pool deployment (Uniswap / Curve / Balancer). Nth asset is O(1).
- **Wins on regime adaptation**: spline + vol-scaled dispersion + coverage-driven skew updates on every swap. Curve V1 needs governance to rotate `A`; Curve V2 repegs parametrically but lossily; Uniswap is static unless hook-mediated; Platypus/Wombat curves are static.
- **Strictly better than DODO / Platypus on policy flexibility**: explicit two-factor fees + admin-mutable spline vs. DODO's static `i/k` and Platypus's static slip function.
- **Loses to Curve V1 on stable-stable in-peg**: amplified invariant gives marginally better near-peg execution for pure stable pairs at scale, with 5+ years of audit hardening and mature gauge incentives. BTR competes on multi-asset breadth and adaptivity, not single-purpose peg concentration.
- **Loses to Uniswap V3/V4 on long-tail TAM**: permissionless pair deployment + tick model fits unknown / experimental pairs better than a curated blue-chip singleton. Long-tail is correctly served by V3/V4 and by BTR Supply's CL adapters layered on top.
- **Loses to OrbSwap (in theory) on mathematical isolation**: orbital geometry gives provable per-asset depeg isolation; BTR uses an operational depeg-halt circuit breaker. OrbSwap is pre-production; BTR ships.

---

## 4. Architectural Cheat Sheet

```
Uniswap V2:    reserves → invariant → price                       (static, 2 assets)
Uniswap V3/V4: reserves + ticks → local CPMM → price              (static, 2 assets, manual range)
Curve V1:      reserves + A → amplified invariant → price         (static-A, N stables, joint risk)
Curve V2:      reserves + A,γ + EMA → repegged invariant → price  (param-EMA, 3 volatile, repeg loss)
Balancer:      reserves + weights → geomean invariant → price     (static-weights, ≤8 assets)
DODO:          oracle + i,k → PMM price                           (static, 2 assets, oracle-anchored)
Maverick:      bins + mode → auto-shifted CL → price              (mode-static, 2 assets)
LFJ LB:        bins + var-fee → discrete CSMM → price             (vol-reactive, 2 assets)
Platypus:      reserves + coverage → slip fn → price              (static, N stables, joint risk)
Wombat:        reserves + per-asset coverage → slip fn → price    (static, N stables, isolated)
OrbSwap:       reserves on n-sphere + polar tick → price          (static, N pegged, math-isolated)
BTR DEX:       TWAP + coverage skew + spline σ-disp + ν·σ+λ·Δ fee (adaptive, N blue-chip, anchor-tree)
```

---

## 5. References

[^1]: Adams, H. *Uniswap V2 Core* (2020). https://uniswap.org/whitepaper.pdf
[^2]: Adams, H. et al. *Uniswap V3 Core* (2021). https://uniswap.org/whitepaper-v3.pdf
[^3]: Uniswap Labs. *Uniswap V4 Core* (2024). https://github.com/Uniswap/v4-core
[^4]: Egorov, M. *StableSwap — efficient mechanism for Stablecoin liquidity* (2019). https://docs.curve.finance/assets/pdf/stableswap-paper.pdf
[^5]: Egorov, M. *Automatic market-making with dynamic peg* (2021). https://docs.curve.finance/assets/pdf/whitepaper.pdf
[^6]: Balancer Labs. *Balancer V2 Vault* / *V3 Hooks* (2021, 2024). https://docs.balancer.fi/
[^7]: DODO Team. *DODO: A Next-generation On-chain Liquidity Provider Powered by PMM* (2020). https://docs.dodoex.io/
[^8]: Maverick Protocol. *Maverick AMM Whitepaper* (2022). https://docs.mav.xyz/
[^9]: Trader Joe / LFJ. *Liquidity Book Whitepaper* (2022). https://docs.lfj.gg/concepts/concentrated-liquidity
[^10]: Platypus Finance. *Platypus: An Open Liquidity Pool Protocol for Stableswap* (2021). https://cdn.platypus.finance/Platypus_Finance_Whitepaper.pdf
[^11]: Wombat Exchange. *Wombat: 4th-Gen Multichain-Native Stableswap* (2022). https://docs.wombat.exchange/
[^12]: Paradigm / OrbSwap. *Orbital AMM Lite Paper* (2025). https://orbswap.org/lite-paper
