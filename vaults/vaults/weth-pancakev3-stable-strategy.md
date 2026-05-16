---
title: "WETH - PancakeSwap v3 Stable Strategy"
description: "WETH vault deploying into PancakeSwap v3 stable-pair concentrated liquidity."
audience: user
type: reference
status: planned
phase: 1
order: 2
lang: en
publish: false
---

# WETH - PancakeSwap v3 Stable Strategy

> **Placeholder** - content + screenshots forthcoming. See [GitHub issues](https://github.com/btr-protocol) for status.

## Underlying venue
PancakeSwap v3 - WETH/stable pair (BNB Chain)

## Strategy
Tight concentrated-liquidity range around the WETH/stable price; rebalance on range exit; modest leverage cap.

## APY breakdown

| Component | Value |
|---|---|
| Trading fees | `[%]` |
| Incentives | `[%]` |
| Net (post-fees) | `[%]` |

## TVL
`[$XXX,XXX]`

## Fees

| Fee | Rate |
|---|---|
| Management | `[%]` per year |
| Performance | `[%]` of profit |
| Exit | `[%]` |

## Contract address
`[ADDRESS]` on BNB Chain

## Audit references
- `[Audit firm / date / link]`

## Risk tier
**Risk tier:** `Medium` 🟡

See [risk tier legend](./_template.md#risk-tier-definitions-legend) for quantitative anchors.

## Withdraw delay
`[N hours / instant]`

## Deposit Configuration
- **Deposit cap:** `[CAP]` (max per-wallet) / `[GLOBAL_CAP]` (vault-wide)
- **Asset model:** Dual-asset WETH + stable
- **Share token:** `[sBtr-WETH-PCSv3-STABLE]` (ERC-20, claimable on BNB Chain)
- **Chain:** BNB Chain (chain ID `56`)
- **Oracle:** Chainlink ETH/USD, staleness threshold `[HEARTBEAT]`
- **Last rebalance:** `[ISO 8601 timestamp]`

## Historical Performance (illustrative)
- **30-day maxDD:** `[%]`
- **180-day maxDD:** `[%]`
- **Cumulative net return since inception:** `[%]`

## Example Deposit Walkthrough
1. *Pending screenshots - see [Tutorials → Deposit into Vault](../../tutorials/deposit-into-vault.md) once content lands.*
