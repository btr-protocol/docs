---
title: "Per-Vault Page Template"
description: "Template for individual vault detail pages."
audience: user
type: reference
status: planned
phase: 1
order: 99
lang: en
publish: false
---

# `<Vault Display Name>`

> **Placeholder** — content + screenshots forthcoming. See [GitHub issues](https://github.com/btr-protocol) for status.

## Underlying venue
`<e.g. Uniswap v3 — ETH/USDC 0.05%>`

## Strategy
`<concise strategy description: range width, rebalance cadence, hedging>`

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
`[ADDRESS]` on `[CHAIN]`

## Audit references
- `[Audit firm / date / link]`

## Risk tier
**Risk tier:** `[Low | Medium | High]`

### Risk Tier Definitions (legend)

| Tier | Color | Icon | Quantitative anchor |
|------|-------|------|---------------------|
| Low | Green | 🟢 | 30d maxDD < 5%, oracle: stable-pair Chainlink, withdraw delay ≤ 1h, no synthetic exposure |
| Medium | Amber | 🟡 | 30d maxDD 5-15%, oracle: stable-pair OR major-pair Chainlink, withdraw delay ≤ 24h |
| High | Red | 🔴 | 30d maxDD > 15% OR exotic-pair oracle OR withdraw delay > 24h OR synthetic/long-tail exposure |

Color + icon ensure a11y; tier is also conveyed in text.

## Withdraw delay
`[N hours / instant]`

## Deposit Configuration
- **Deposit cap:** `[CAP]` (max per-wallet) / `[GLOBAL_CAP]` (vault-wide)
- **Asset model:** `[Single-asset USDC / dual-asset USDC+WETH]`
- **Share token:** `[sBtr-USDC-V3-ETH]` (ERC-20, claimable on `[CHAIN_ID]`)
- **Chain:** `[Ethereum / Arbitrum / Base / BNB]` (chain ID `[CHAIN_ID]`)
- **Oracle:** `[Chainlink ETH/USD]`, staleness threshold `[HEARTBEAT]`
- **Last rebalance:** `[ISO 8601 timestamp]`

## Historical Performance (illustrative)
- **30-day maxDD:** `[%]`
- **180-day maxDD:** `[%]`
- **Cumulative net return since inception:** `[%]`

## Example Deposit Walkthrough
1. *Pending screenshots — see [Tutorials → Deposit into Vault](../../tutorials/deposit-into-vault.md) once content lands.*
