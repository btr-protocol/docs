---
title: "USDC - Uniswap v3 ETH Strategy"
description: "USDC vault deploying into Uniswap v3 ETH/USDC concentrated liquidity."
audience: user
type: reference
status: planned
phase: 1
order: 1
lang: en
publish: false
---

# USDC - Uniswap v3 ETH Strategy

> **Placeholder** - content + screenshots forthcoming. See [GitHub issues](https://github.com/btr-protocol) for status.

## Underlying venue
Uniswap v3 - ETH/USDC 0.05% pool (Ethereum mainnet)

## Strategy
Concentrated liquidity around spot; periodic rebalance to maintain target range width. Optional delta hedge via perp short.

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
`[ADDRESS]` on Ethereum

## Audit references
- `[Audit firm / date / link]`

## Risk tier
**Risk tier:** `Medium` 🟡

See [risk tier legend](./_template.md#risk-tier-definitions-legend) for quantitative anchors.

## Withdraw delay
`[N hours / instant]`

## Deposit Configuration
- **Deposit cap:** `[CAP]` (max per-wallet) / `[GLOBAL_CAP]` (vault-wide)
- **Asset model:** Single-asset USDC
- **Share token:** `[sBtr-USDC-V3-ETH]` (ERC-20, claimable on Ethereum)
- **Chain:** Ethereum (chain ID `1`)
- **Oracle:** Chainlink ETH/USD, staleness threshold `[HEARTBEAT]`
- **Last rebalance:** `[ISO 8601 timestamp]`

## Historical Performance (illustrative)
- **30-day maxDD:** `[%]`
- **180-day maxDD:** `[%]`
- **Cumulative net return since inception:** `[%]`

## Example Deposit Walkthrough
1. *Pending screenshots - see [Tutorials → Deposit into Vault](../../tutorials/deposit-into-vault.md) once content lands.*
