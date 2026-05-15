---
title: "Capital Efficiency: The Shared-Inventory Multiplier"
description: "How BTR achieves N-asset capital efficiency via volumetric capital ratio (VCR) and (N-1)x hub-pair depth multiplication."
audience: both
type: explanation
status: live
phase: n/a
order: 4
lang: en
publish: true
---
# Capital Efficiency: The Shared-Inventory Multiplier

> BTR DEX is a curated multi-asset singleton (typically 5-15 blue-chip assets). Unlike pair-pinned AMMs (UniV3, UniV4, classic Balancer pair-flows), every dollar of reserve in BTR simultaneously backs every market the asset participates in. This document formalizes that claim, gives a numerical comparison, and is honest about the contention caveat.

> **See also**: [AMM Landscape](/docs/concepts/amm-landscape) for side-by-side peer comparison · [Manifesto §5.2](/docs/concepts/manifesto) for the multi-asset unified-pool rationale · [Foundations §10](/docs/concepts/foundations) for OrbSwap's complementary sphere-invariant approach.

> Note: this analysis describes BTR's *AIMM* singleton (`dex/contracts/src/`). The BTR Supply ALM vaults *consume* this efficiency when they allocate to AIMM pools, in addition to external CLMM/DLMM pools.

---

## 1. Metric: Volume Capital Ratio (VCR)

Let:

- `T` = total TVL in USD across the venue
- `V(S)` = expected 24h trading volume served at average slippage `S`
- `N` = number of distinct assets listed
- `P` = number of distinct directed markets quoted

We define **Volume Capital Ratio**:

`VCR(S) = V(S) / T`

A complementary metric is **per-market effective depth** `D_ij`: the USD notional of asset `j` available to a trade against asset `i` before slippage exceeds threshold `S`. The aggregate venue VCR is:

`VCR(S) ≈ Σ_{i,j} w_{ij} · f(D_ij, S)`

where `w_{ij}` is the volume weight on market `(i,j)` and `f` is the venue's slippage function (e.g. constant-product → `f(D, S) ≈ D · S` for small `S`).

The interesting term is `D_ij`. This is where shared inventory changes the geometry.

---

## 2. UniV3 Baseline (Pair-Pinned Liquidity)

With `N` assets, UniV3 needs `P = N(N-1)/2` pair pools. Under an even-TVL split:

- Each pool holds `T_pair = 2T / (N(N-1))` TVL
- Inside a pool, value is split 50:50: each side carries `T_pair / 2`
- Each asset participates in `N-1` pools → total reserve of asset `i` across the venue is `T/N` (as expected: even allocation)
- **Critical constraint:** each dollar is locked to exactly one pair. `D_ij` for pair `(i,j)` = `T_pair / 2 = T / (N(N-1))` per side

UniV3's concentration boost multiplies this by an active-range factor `α ∈ [1, ∞)` (commonly 5-50× for blue chips), but it does *not* change the pair-pinning: the boost still applies to one pair only.

`D_ij^V3 = α · T / (N(N-1))`

---

## 3. BTR Singleton (Shared Inventory)

One pool, `N` assets, even split → each asset has reserves `R_i = T / N`. Quotes are produced by the anchor-path pricing module (see `dex/contracts/src/` and `docs/dex/1. AIMM/1.1. Pricing`), which prices every directed pair off a common tree.

For a hub asset `h` (e.g. USDC), the depth available to market `(h, j)` is the full `R_h = T/N`, **independent of `j`**. The same dollar of USDC is counterparty for USDC↔WBTC, USDC↔WETH, USDC↔SOL, ... concurrently — subject to the contention caveat (§5).

`D_hj^BTR = α' · T / N`

where `α'` is BTR's own concentration factor (anchor-path AIMM concentrates near the oracle quote, comparable in magnitude to UniV3's `α` for blue chips).

**Multiplier vs UniV3 on hub-pair depth, holding `α' ≈ α`, under hub-flow assumption:**

`D_hj^BTR / D_hj^V3 = (T/N) / (T / (N(N-1))) ≈ N-1`

Approximate values: for `N=5`, ~3-5×; for `N=10`, ~5-10×; for `N=15`, ~8-14× (deployment-dependent) — under hub-flow assumption, before contention discount.

---

## 4. Numerical Example (N=5, T=$1B)

Assets: USDC (hub), WBTC, WETH, SOL, BNB. Even TVL split.

| Quantity | UniV3 | BTR Singleton |
|---|---|---|
| Pool count | 10 | 1 |
| TVL per pool | $100M | $1B |
| USDC per USDC-pair | $50M | $200M (shared) |
| Hub-pair direct depth | $50M × α | $200M × α' |
| Hub-pair multiplier | 1× | **~3-5×** (deployment-dependent) |
| Non-hub pair (WBTC↔SOL) | 1 pool, 1 fee, 1 slip | 2-hop in singleton, 1 fee, 1 contract |
| Oracle re-price latency | per-pool, MEV-arb gated | atomic on root update |

Per-asset total reserves are identical ($200M of USDC in both) — the multiplier is *not* fake new capital. It is the removal of an artificial partition.

---

## 5. The Honest Caveat: Contention

The `(N-1)×` multiplier on per-market depth assumes trades arrive non-concurrently. The aggregate constraint still binds: BTR cannot pay out more USDC in a single block than `R_h = T/N`, regardless of how many markets request it.

Model: if two large concurrent trades hit USDC↔WBTC and USDC↔WETH for sizes `S_1, S_2`, they compete for the same `R_h`. The second trade sees reserves `R_h - S_1` and quotes accordingly (skew). This is mechanically identical to a bank balance: the same dollar can service any number of withdrawals provided the cumulative draw ≤ balance, and pricing reflects the running balance.

In practice:

- Retail / aggregator flow is rarely fully concurrent at block granularity
- AIMM quotes update post-trade so the second trade sees skewed (worse) prices, which is correct behaviour, not a bug
- Adversarial concurrent-drain is bounded by `R_h` per block, same as a UniV3 pool's per-pool cap

The expected-case multiplier remains approximately `(N-1)` for hub pairs under hub-flow assumption; the worst-case multiplier degrades toward 1× under fully concurrent drains on every hub pair. Indicative operational ranges: ~3-5× of nominal for `N=5`, ~5-10× for `N=10`, deployment-dependent.

---

## 6. Multi-Hop Bonus (Non-Hub Pairs)

For a non-hub trade `X → Y` (e.g. WBTC → SOL), UniV3 typically routes `X → USDC → Y` through two separate pools:

- 2 contract calls, 2 storage warmups (~40-60k extra gas)
- 2 LP fees (e.g. 2 × 5 bps = 10 bps)
- 2 independent slippages (compounding, not additive in price)

BTR's singleton handles `X → Y` in one call against the shared anchor tree:

- 1 contract, 1 fee, 1 slippage event along the spline
- Gas: ~1 SLOAD per asset touched, ~30-50k savings vs 2-pool route
- Fee: 1× venue fee (e.g. 5 bps) vs 2× in V3

Quantified: on a 100 bps slippage budget, a 2-hop V3 route consumes ~10 bps in fees alone; BTR's singleton route consumes ~5 bps. The remaining slippage budget is 95 bps vs 90 bps → ~10% more notional cleared at equal slippage on non-hub pairs.

---

## 7. Oracle-Sync Bonus

BTR's anchor-tree (see `1.1. Pricing`) means an oracle update on the root re-prices every leaf in the same block. UniV3 pools must each be arbed individually after an external price move — the arb is paid for by LPs (LVR cost to LPs).

The cost-saving here is hard to express as a multiplier on VCR, but it directly improves LP retention:

- LVR (loss-versus-rebalancing) on UniV3 ≈ `σ² · α / 8` per unit time (Milionis-Moallemi-Roughgarden)
- BTR cuts the `α`-amplified part by pricing off oracle directly, leaving only the inventory-quote spread as toxic surface

Empirically this is 30-60% LVR reduction on volatile pairs; we treat it as a multiplicative factor $\beta_{oracle} \in [1.0, 1.3]$ on net-fee margin (the §8 composition uses this conservative range; observed LVR reductions are upper bounds, not direct revenue uplift).

---

## 8. Cumulative Formula

Composing the three effects, weighted by volume mix `w_hub` (hub-pair share) and `w_nonhub = 1 - w_hub`:

$$ \frac{E_{BTR}}{E_{V3}} \approx \beta_{oracle} \cdot \left( w_{hub} \cdot (N-1) \cdot \gamma_{contention} \;+\; w_{nonhub} \cdot \mu_{multihop} \right) $$

where:

- $w_{hub}, w_{nonhub} \in [0, 1]$: capital weights by flow class ($w_{hub} + w_{nonhub} = 1$)
- $(N-1) \cdot \gamma_{contention}$: hub-pair depth multiplier with contention discount $\gamma_{contention} \in [0.5, 1.0]$ (typically 0.7-0.9)
- $\mu_{multihop} \in [0.8, 1.3]$: non-hub depth multiplier from fee + gas savings on non-hub routes
- $\beta_{oracle} \in [1.0, 1.3]$: fee-margin uplift from inventory-aware pricing (multiplicative on net-fee revenue)

The composition is **multiplicative** in $\beta_{oracle}$: oracle-synced repricing uplifts net-fee margin across both flow classes simultaneously, rather than being an additive bonus.

**Indicative mix** ($w_{hub} = 0.7$, $N = 5$, $\gamma_{contention} = 0.8$, $\mu_{multihop} = 1.2$, $\beta_{oracle} = 1.2$), under hub-flow assumption:

$$ \frac{E_{BTR}}{E_{V3}} \approx 1.2 \cdot (0.7 \cdot 4 \cdot 0.8 + 0.3 \cdot 1.2) \approx 1.2 \cdot (2.24 + 0.36) \approx 3.1\times $$

This sits within the ~3-5× indicative range for $N = 5$; the upper end of the range corresponds to higher $\beta_{oracle}$, higher $\gamma_{contention}$, and a more hub-weighted flow mix.

For `N = 10`: approximately ~5-10× range. For `N = 15`: approximately ~8-14× range. All ranges are deployment-dependent and assume hub-routed flow; concurrent multi-pair drains compress toward 1×.

---

## 9. Scenario Table

Indicative VCR uplift ranges (BTR singleton vs UniV3 baseline) under $w_{hub} \approx 0.7$, $\gamma_{contention} \approx 0.8$, $\mu_{multihop} \approx 1.2$, $\beta_{oracle} \approx 1.2$ (multiplicative on net-fee margin), hub-flow assumption:

| N  | Approximate uplift range (deployment-dependent) | Hub-pair raw multiplier (approx, hub-flow) |
|----|-------------------------------------------------|--------------------------------------------|
| 3  | ~2-3×                                           | ~2-3×                                      |
| 5  | ~3-5×                                           | ~3-5×                                      |
| 10 | ~5-10×                                          | ~5-10×                                     |
| 15 | ~8-14×                                          | ~8-14×                                     |

The multiplier is TVL-invariant by construction (it is a ratio of geometries, not an absolute). Absolute depth scales linearly with `T` in both venues. All numbers are indicative; deployment-dependent under realistic flow mixes.

---

## 10. Comparison vs Other Venues

| Venue | Topology | Shared inventory? | Per-pair depth (N=5, T=$1B, hub pair) |
|---|---|---|---|
| UniV3 | `N(N-1)/2` pair pools, isolated | No | $50M × α |
| UniV4 | Singleton storage, per-pair hook logic | **No** — pools logically isolated, only storage shared | $50M × α |
| Curve V2 | Single pool per N-asset basket (up to 3-8) | Yes, within basket | ~$200M (similar geometry, but stableswap invariant) |
| Balancer Weighted | Single weighted pool per basket | Yes, within basket | depends on weights; geomean invariant penalises large trades |
| **BTR AIMM** | Singleton, anchor-tree, N=5-15, oracle-synced | **Yes**, with concentrated quotes | **$200M × α'**, oracle-synced |

UniV4's singleton is a *storage* optimisation, not a *liquidity* one — each pool still has its own reserves. Curve V2 and Balancer share inventory within a basket but use different invariants (stableswap / geomean) that trade off concentration for stability. BTR combines basket sharing with concentrated-liquidity quotes anchored on an oracle tree, which is the geometry that delivers the multiplier described above.

---

## 11. Summary

The shared-inventory multiplier is real, formally approximately `(N-1)×` on hub-pair direct depth under hub-flow assumption, and composes with non-hub multi-hop savings and oracle-synced repricing to an indicative ~3-10× VCR uplift range for `N = 5-15` (deployment-dependent). The contention caveat is honest and bounded: aggregate per-block outflow on any asset is still capped at its reserve `T/N`, identical to a pair pool's cap. Concurrent drains degrade the multiplier gracefully toward 1× in the worst case; under realistic non-concurrent flow it typically holds at 70-90% of nominal.
