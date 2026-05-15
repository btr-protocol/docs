---
title: "Staking — Summary"
description: "Friendly summary of BTR staking: lock BTR, earn fee share, vote with veBTR."
audience: user
type: explanation
status: planned
phase: 2
order: 11
lang: en
publish: true
---

# Staking — Summary

> **Placeholder** — content + screenshots forthcoming. See [GitHub issues](https://github.com/btr-protocol) for status.

## What it is

Staking lets you lock BTR for a chosen duration to receive **veBTR** (vote-escrowed BTR). veBTR earns a share of protocol fees and confers governance weight proportional to lock length.

> **Status**: Phase-2 (forward-looking). Staking is **not yet live**.

## Why it matters to users

- **Earn fees**: a slice of swap, vault, and bridge fees routes to stakers.
- **Govern**: veBTR is the vote weight in on-chain governance (also Phase 2).
- **Align**: longer locks → larger veBTR multiplier → larger fee share + vote weight.

## Key parameters (planned)

| Field | Value |
|---|---|
| Min lock | TBD |
| Max lock | TBD |
| Lock curve | Linear decay (planned) |
| Fee-share source | Protocol-wide via [Distributor](./05.%20Distributor.md) |

## Deep dive

For full mechanics — lock math, distributor flow, claim cadence — see [`04. Staking.md`](./04.%20Staking.md).
