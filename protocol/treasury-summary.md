---
title: "Treasury - Summary"
description: "Friendly summary of the BTR treasury: fees in, POL out, supply enforcement."
audience: user
type: explanation
status: live
phase: 1
order: 13
lang: en
publish: true
---

# Treasury - Summary

## What it is

The BTR Treasury is the protocol's single fund-management hub. It collects all protocol fees (swap, vault, bridge), holds protocol-owned liquidity (POL), pays grants and operational expenses under bounded spending caps, and enforces the 100M BTR hard supply cap.

## Why it matters to users

- **Sustainability**: predictable fee accrual funds long-term protocol development without diluting holders.
- **Transparency**: monthly (2% of supply) and annual (8% of supply) spending caps are on-chain enforced; nothing can be spent above the cap without supermajority governance.
- **Aligned operators**: vault performance fees route through the same treasury SSoT as DEX fees - no off-the-books revenue paths.

## Key parameters

| Field | Value |
|---|---|
| Monthly spend cap | 2% of total supply |
| Annual spend cap | 8% of total supply |
| Supply cap enforcement | 100M BTR hard ceiling (supermajority required to alter) |
| Authority | Council + governance, bounded by timelock |

## Deep dive

For full mechanics - fee routing diagram, vesting schedules, council scope, supply-cap proof - see [`03. Treasury.md`](./03.%20Treasury.md).
