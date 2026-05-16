---
title: "Concepts Overview"
description: "BTR Protocol overview - ALM vaults (Phase 1 live, allocates across UniV3/V4/Pancake/Algebra/Ramses), Swap aggregator, AIMM DEX (Phase 2)."
audience: both
type: explanation
status: live
phase: n/a
order: 2
lang: en
publish: true
---
# Concepts Overview

> **BTR** ships three complementary surfaces: **Supply (ALM Vaults)** -Automated Liquidity Management vaults that allocate single-asset deposits across 3rd-party concentrated-liquidity DEX pools (forward-looking: AIMM pools); **Swap** -off-chain liquidity meta-aggregator (LiFi, Socket, 1inch, …); and **DEX (AIMM)** -BTR's own AMM (anchor-path-priced, inventory-aware).

> **Live now**: BTR Supply (ALM) - single-asset vaults allocating across UniV3, UniV4, PancakeV3, PancakeInfinity, Algebra, Ramses. **Roadmap**: BTR DEX (AIMM) joins as additional adapter target.

## Three Surfaces

| Product | Role | Users | Launch Phase | Section |
|---------|------|-------|--------------|---------|
| **BTR Supply (ALM Vaults)** | Single-asset ERC-4626/7540 vaults that deploy LP across UniV3, UniV4, PancakeV3, PancakeInfinity, Algebra, Ramses (Aerodrome Slipstream via UniV3 adapter) -and (forward-looking) AIMM. Worst-of(pool, oracle) marks, HWM perf-fee, sync + async redeem. | Passive depositors, treasuries, institutional LPs. | **Phase 1 (Live)** | [BTR Supply](../vaults/01.%20Overview.md) |
| **BTR Swap** | Off-chain meta-aggregator. Composes LiFi, Socket, Squid, Rango, Unizen, 1inch, 0x, ParaSwap, Odos, KyberSwap, OpenOcean, Firebird. Best-of-kind routing, keeper calldata validation. | Front-end users, vault keepers, integrators. | **Phase 1 (Live)** | [BTR Swap](../swap/01.%20Overview.md) |
| **BTR Dex (AIMM)** | BTR's own AMM. Multi-asset pools, anchor-path routing, dynamic inventory-aware fees. On launch, becomes an additional Supply adapter target. | Traders, routers, integrators. | **Phase 2 (Future)** | [BTR Dex](../dex/1.%20AIMM/Overview.md) |

Dex and Supply share `AccessControl` (governance SSoT), price feeds, and the protocol treasury once the DAO is live. Swap is the off-chain execution layer consumed by both front-end users and Supply's vault keepers.

---

## Phase 1 - BTR Supply (ALM Vaults) - Live

<!-- no-header -->
| | |
|---|---|
| [**Overview**](../vaults/01.%20Overview.md) | ALM product surface -ERC-4626/7540 single-asset vaults |
| [**Architecture**](../vaults/02.%20Architecture.md) | Vault, Dex adapter, Swapper, AccessControl, Factory, clones |
| [**Adapters & Pools**](../vaults/03.%20Adapters%20&%20Pools.md) | UniV3, UniV4, PancakeV3, PancakeInfinity, Algebra, Ramses (Aerodrome Slipstream via UniV3 adapter) |
| [**NAV & Fees**](../vaults/04.%20NAV%20&%20Fees.md) | Pessimistic worst-of(pool, oracle); HWM perf-fee; mgmt-fee; exit-fee floor |
| [**Risk Model**](../vaults/05.%20Risk%20Model.md) | IL, LVR, deviation, sequencer outage, intent gating, kill switch |
| [**Lifecycle**](../vaults/06.%20Lifecycle.md) | Deposit, redeem (sync + async ticket), allocate, rebalance, harvest |
| [**Operator Doctrine**](../vaults/07.%20Operator%20Doctrine.md) | Owner vs keeper, ratchets, kill cap, timelocks |
| [**Cross-chain (forward-looking)**](../vaults/08.%20Cross-chain%20%28forward-looking%29.md) | Extension scenarios + open questions |
| [**Token & Governance (forward-looking)**](../vaults/09.%20BTR%20Token%20&%20Governance%20%28forward-looking%29.md) | Token utility scope, subject to change |

---

## Phase 2 - BTR DEX (AIMM) - Future Venue

> On launch, BTR DEX becomes an additional adapter target alongside UniV3, UniV4, PancakeV3, PancakeInfinity, Algebra, Ramses in Supply vaults.

### Core Technical Features

BTR's AIMM protocol introduces several mechanisms not found in existing AMMs:

| Feature | Technical Innovation |
|---------|---------------------|
| **AIMM** | Adaptive Inventory Market Maker -dynamically manages inventory risk through coverage ratios and skew-aware pricing |
| **Anchor Path Pricing** | Multi-asset routing via LCA (Lowest Common Ancestor) algorithm on an anchor tree topology |
| **Asset-Specific Liquidity Profiling** | Liquidity can be extremely concentrated around historical price density using Fritsch-Carlson monotone cubic Hermite spline profiles |
| **N-Asset Pooling** | No limit to the number of assets in a single pool -any-to-any swaps via anchor tree routing |
| **Coverage-Based IL Protection** | Reserves and liabilities tracked separately; LPs withdraw the same token count deposited |
| **Cooperative Arbitrage** 🚧 | *Roadmap (not yet shipped)* - whitelisted arbitrageurs compete for rebates, proceeds donated to LPs |

> 🚧 **FUTURE WORK - NOT YET IMPLEMENTED.** Cooperative Arbitrage is a designed feature on the BTR DEX roadmap, not yet shipped on-chain. No Solidity implementation exists in the current release. Feature target: post-mainnet, phase TBD. All Cooperative Arbitrage references in this document describe the proposed design.

Unlike traditional AMMs that use pair-based pools (Uniswap) or stablecoin-specific curves (Curve), BTR's anchor tree allows unlimited assets with capital efficiency that scales with liquidity depth.

---

<!-- no-header -->
| | |
|---|---|
| [**Manifesto**](manifesto.md) | Technical manifesto -problems, solutions, architecture, roadmap |
| [**Foundations**](foundations.md) | Design precedents: Avellaneda-Stoikov, Platypus/Wombat ALM, Curve v2 |
| [**Architecture**](../dex/1.%20AIMM/Overview.md) | System overview, modules, and data structures |

---

## Core Concepts

<!-- no-header -->
| | |
|---|---|
| [**Inventory Management**](../dex/1.%20AIMM/1.1.%20Pricing/1.1.1.%20Inventory%20Management.md) | Coverage ratios, skew calculation, ALM mechanics |
| [**Liquidity Shaping**](../dex/1.%20AIMM/1.1.%20Pricing/1.1.2.%20Liquidity%20Shaping.md) | Fritsch-Carlson monotone cubic Hermite spline profiles, depth curves |
| [**Spread & Fees**](../dex/1.%20AIMM/1.1.%20Pricing/1.1.4.%20Spread%20&%20Fees.md) | Bi-factor dynamic fees (volatility band + deviation surcharge) |
| [**Toxic Flow Mitigation**](../dex/1.%20AIMM/1.1.%20Pricing/1.1.6.%20Toxic%20Flow%20Mitigation.md) | Adverse selection protection, Cooperative Arbitrage |
| [**Internal Oracle**](../dex/1.%20AIMM/1.2.%20Modules/1.2.2.%20Internal%20Oracle.md) | Dual-window TWAP, volatility EMAs |

---

## Key Differentiators

AIMM introduces several innovations not found in existing AMMs:

| Feature | Description |
|---------|-------------|
| **Single-sided deposits** | Deposit one token, receive fungible LP tokens (no forced pairs) |
| **N-asset pooling** | Anchor tree topology routes any-to-any swaps through common ancestors |
| **Algo-optimized profiles** | Liquidity concentration based on historical price density |
| **Coverage-based IL protection** | Reserves and liabilities tracked separately; LPs withdraw same token count |
| **Cooperative Arbitrage** 🚧 | *Roadmap (not yet shipped)* - whitelisted arbitrageurs compete for rebates, donate proceeds to LPs |

---

## Protocol Features

<!-- no-header -->
| | |
|---|---|
| [**Flash Loans**](../dex/1.%20AIMM/1.2.%20Modules/1.2.6.%20Flash.md) | ERC-3156 compliant flash lending |
| [**Rescue Module**](../dex/1.%20AIMM/1.2.%20Modules/1.2.3.%20Admin.md) | Emergency asset recovery |
| [**Anchor Path Pricing**](../dex/1.%20AIMM/1.1.%20Pricing/1.1.3.%20Anchor%20Path%20Pricing.md) | Multi-asset routing via LCA algorithm |

---

## Protocol (shared across ALM + DEX)

<!-- no-header -->
| | |
|---|---|
| [**Protocol Overview**](../protocol/01.%20Overview.md) | Shared surfaces: AccessControl, Treasury, Distributor, Staking, Bridge, Governance |
| [**AccessControl**](../protocol/02.%20AccessControl.md) | SSoT roles + registries, 2-step Solady, timelocks |
| [**Treasury**](../protocol/03.%20Treasury.md) | Fee collection (Vault + DEX), POL, spending, supply cap |
| [**Staking**](../protocol/04.%20Staking.md) | sBTR + sLP + sVault (forward-looking), cooldowns |
| [**Distributor**](../protocol/05.%20Distributor.md) | Campaign-based emissions to all stake pools |
| [**Bridge**](../protocol/06.%20Bridge.md) | LayerZero v2 OApp, BTR + receipts cross-chain |
| [**Governance**](../protocol/07.%20Governance.md) | Voting + scope: Vault, DEX, Treasury, Emissions, Bridge |
| [**BTR Token**](../protocol/08.%20BTR%20Token.md) | ERC-20, 100M cap, utility |
| [**Emissions**](../protocol/09.%20Emissions.md) | Halving schedule, routing to stake pools |

---

## Security

<!-- no-header -->
| | |
|---|---|
| [**Security Overview**](../dex/3.%20Security/Overview.md) | Defense-in-depth architecture |
| [**Access Control**](../dex/3.%20Security/3.3.%20Access%20Control.md) | Timelocks, roles, emergency controls |
| [**Flow Guards**](../dex/3.%20Security/3.4.%20Flow%20Guards.md) | JIT protection, reentrancy, cooldowns |
| [**Oracles**](../dex/3.%20Security/3.5.%20Oracles.md) | Oracle security and fallback mechanisms |

---

## Developer Integration

<!-- no-header -->
| | |
|---|---|
| [**AMM Integration**](../dex/1.%20AIMM/1.3.%20Integration/Overview.md) | Swaps, liquidity, pool deployment, hooks (Phase 2 - AIMM) |
| [**Developer Guides**](../dex/5.%20Developer%20Guides/Overview.md) | Treasury, bridging, white-labelling |
| [**Contributing**](../dex/6.%20Contributing/ARCHITECTURE.md) | Internal development practices |

## Developer Reference

<!-- no-header -->
| | |
|---|---|
| [**Parametrization**](../dex/1.%20AIMM/1.1.%20Pricing/1.1.7.%20Parametrization.md) | Complete parameter reference (AIMM) |
| [**Invariants**](../dex/1.%20AIMM/1.1.%20Pricing/1.1.8.%20Invariants.md) | System-wide constraints and fuzzing targets (AIMM) |
| **ALM Contracts** | `alm/evm/src/` -Vault + Dex adapter + Swapper + AC + Factory + PriceProvider |
| **AIMM Contracts** (Phase 2) | `dex/contracts/src/` -Solidity implementation |
| **TypeScript SDK** | `~/Work/btr/sdk` -Client library + ABIs |

---

## Related surfaces

- **Want to swap?** → [Swap](../trade/overview.md)
- **Want to earn yield?** → [Vaults](../vaults/01.%20Overview.md)
- **Want to provide liquidity to BTR DEX (Phase 2)?** → [Pools - coming soon](../liquidity/coming-soon.md)
- **Want to govern / stake?** → [Protocol → Governance](../protocol/governance-summary.md)
- **Worried about risk?** → [Security Overview](../security/overview.md), [Risk Disclaimer](../legal/risk-disclaimer.md)
