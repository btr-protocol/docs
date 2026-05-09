# BTR Protocol Documentation

> **BTR** ships three complementary surfaces: **Swap** — off-chain liquidity meta-aggregator (LiFi, Socket, 1inch, …); **Dex (AIMM)** — BTR's own AMM (anchor-path-priced, inventory-aware); and **Supply (ALM Vaults)** — adaptive liquidity managers that allocate single-asset deposits to external concentrated-liquidity DEX pools and (forward-looking) AIMM pools.

## Three Surfaces

| Product | Role | Users | Section |
|---------|------|-------|---------|
| **BTR Swap** | Off-chain meta-aggregator. Composes LiFi, Socket, Squid, Rango, Unizen, 1inch, 0x, ParaSwap, Odos, KyberSwap, OpenOcean, Firebird. Best-of-kind routing, keeper calldata validation. | Front-end users, vault keepers, integrators. | [BTR Swap](/docs/swap/01-Overview) |
| **BTR Dex (AIMM)** | BTR's own AMM. Multi-asset pools, anchor-path routing, dynamic inventory-aware fees. | Traders, routers, integrators. | [BTR Dex](/docs/dex/1-AIMM) |
| **BTR Supply (ALM Vaults)** | Single-asset ERC-4626/7540 vaults that deploy LP across UniV3/V4, PancakeV3/Infinity, Algebra, Ramses, Aerodrome Slipstream — and (forward-looking) AIMM. Worst-of(pool, oracle) marks, HWM perf-fee, sync + async redeem. | Passive depositors, treasuries, institutional LPs. | [BTR Supply](/docs/supply/01-Overview) |

Dex and Supply share `AccessControl` (governance SSoT), price feeds, and the protocol treasury once the DAO is live. Swap is the off-chain execution layer consumed by both front-end users and Supply's vault keepers.

---

## Core Technical Features

BTR's AIMM protocol introduces several breakthrough innovations:

| Feature | Technical Innovation |
|---------|---------------------|
| **AIMM** | Adaptive Inventory Market Maker — dynamically manages inventory risk through coverage ratios and skew-aware pricing |
| **Anchor Path Pricing** | Multi-asset routing via LCA (Lowest Common Ancestor) algorithm on an anchor tree topology |
| **Asset-Specific Liquidity Profiling** | Liquidity can be extremely concentrated around historical price density using Catmull-Rom spline profiles |
| **N-Asset Pooling** | No limit to the number of assets in a single pool — any-to-any swaps via anchor tree routing |
| **Coverage-Based IL Protection** | Reserves and liabilities tracked separately; LPs withdraw the same token count deposited |
| **Cooperative Arbitrage** | Whitelisted arbitrageurs compete for rebates, proceeds donated to LPs |

Unlike traditional AMMs that use pair-based pools (Uniswap) or stablecoin-specific curves (Curve), BTR's anchor tree allows unlimited assets with capital efficiency that scales with liquidity depth.

---

<!-- no-header -->
| | |
|---|---|
| [**Manifesto**](/docs/manifesto) | Technical manifesto — problems, solutions, architecture, roadmap |
| [**Foundations**](/docs/foundations) | Design precedents: Avellaneda-Stoikov, Platypus/Wombat ALM, Curve v2 |
| [**Architecture**](/docs/overview-aimm) | System overview, modules, and data structures |

---

## Core Concepts

<!-- no-header -->
| | |
|---|---|
| [**Inventory Management**](/docs/1.1.1-Inventory-Management) | Coverage ratios, skew calculation, ALM mechanics |
| [**Liquidity Shaping**](/docs/1.1.2-Liquidity-Shaping) | Catmull-Rom spline profiles, depth curves |
| [**Spread & Fees**](/docs/1.1.4-Spread-&-Fees) | Bi-factor dynamic fees (volatility band + deviation surcharge) |
| [**Toxic Flow Mitigation**](/docs/1.1.6-Toxic-Flow-Mitigation) | Adverse selection protection, Cooperative Arbitrage |
| [**Internal Oracle**](/docs/1.2.2-Internal-Oracle) | Dual-window TWAP, volatility EMAs |

---

## Key Differentiators

AIMM introduces several innovations not found in existing AMMs:

| Feature | Description |
|---------|-------------|
| **Single-sided deposits** | Deposit one token, receive fungible LP tokens (no forced pairs) |
| **N-asset pooling** | Anchor tree topology routes any-to-any swaps through common ancestors |
| **Algo-optimized profiles** | Liquidity concentration based on historical price density |
| **Coverage-based IL protection** | Reserves and liabilities tracked separately; LPs withdraw same token count |
| **Cooperative Arbitrage** | Whitelisted arbitrageurs compete for rebates, donate proceeds to LPs |

---

## Protocol Features

<!-- no-header -->
| | |
|---|---|
| [**Flash Loans**](/docs/1.2.6-Flash) | ERC-3156 compliant flash lending |
| [**Rescue Module**](/docs/1.2.7-Rescue) | Emergency asset recovery |
| [**Anchor Path Pricing**](/docs/1.1.3-Anchor-Path-Pricing) | Multi-asset routing via LCA algorithm |

---

## Governance

<!-- no-header -->
| | |
|---|---|
| [**Governance Overview**](/docs/overview-governance) | DAO structure, voting, token mechanics |
| [**BTR Token**](/docs/2.1-BTR-Token) | Governance token, staking, claim power |
| [**Emission Control**](/docs/2.5-Emission-Control) | Halving schedule, parameter bounds |
| [**DAO Treasury**](/docs/2.3-DAO-Treasury) | Fund management and spending |

---

## Security

<!-- no-header -->
| | |
|---|---|
| [**Security Overview**](/docs/overview-security) | Defense-in-depth architecture |
| [**Access Control**](/docs/3.3-Access-Control) | Timelocks, roles, emergency controls |
| [**Flow Guards**](/docs/3.4-Flow-Guards) | JIT protection, reentrancy, cooldowns |
| [**Oracles**](/docs/3.5-Oracles) | Oracle security and fallback mechanisms |

---

## Developer Integration

<!-- no-header -->
| | |
|---|---|
| [**AMM Integration**](/docs/1.3-Integration) | Swaps, liquidity, pool deployment, hooks |
| [**Developer Guides**](/docs/5-Developer-Guides) | Treasury, bridging, white-labelling |
| [**Contributing**](/docs/6-Contributing) | Internal development practices |

## Vaults (ALM)

<!-- no-header -->
| | |
|---|---|
| [**Vaults Overview**](/docs/7-Vaults) | ALM product surface — ERC-4626/7540 single-asset vaults |
| [**Architecture**](/docs/7.1-Architecture) | Vault, Dex adapter, Swapper, AccessControl, Factory, clones |
| [**Adapters & Pools**](/docs/7.2-Adapters-Pools) | UniV3/V4/PancakeV3/Infinity/Algebra/Ramses/Aerodrome Slipstream |
| [**NAV & Fees**](/docs/7.3-NAV-Fees) | Pessimistic worst-of(pool, oracle); HWM perf-fee; mgmt-fee; exit-fee floor |
| [**Risk Model**](/docs/7.4-Risk-Model) | IL, LVR, deviation, sequencer outage, intent gating, kill switch |
| [**Lifecycle**](/docs/7.5-Lifecycle) | Deposit, redeem (sync + async ticket), allocate, rebalance, harvest |
| [**Operator Doctrine**](/docs/7.6-Operator-Doctrine) | Owner vs keeper, ratchets, kill cap, timelocks |
| [**Cross-chain (forward-looking)**](/docs/7.7-Cross-chain) | Extension scenarios + open questions |
| [**Token & Governance (forward-looking)**](/docs/7.8-Token-Governance) | Token utility scope, subject to change |

---

## Developer Reference

<!-- no-header -->
| | |
|---|---|
| [**Parametrization**](/docs/1.1.7-Parametrization) | Complete parameter reference (AIMM) |
| [**Invariants**](/docs/1.1.8-Invariants) | System-wide constraints and fuzzing targets (AIMM) |
| **AIMM Contracts** | `dex/contracts/src/` — Solidity implementation |
| **ALM Contracts** | `alm/evm/src/` — Vault + Dex adapter + Swapper + AC + Factory + PriceProvider |
| **TypeScript SDK** | `~/Work/btr/sdk` — Client library + ABIs |
