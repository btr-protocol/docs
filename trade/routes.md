---
title: "Routes"
description: "BTR Swap's route-aggregation stack: Li.Fi, BtrDex, and Btr-Swap."
audience: user
type: explanation
status: planned
phase: 1
order: 3
lang: en
publish: true
---

# Routes

BTR Swap composes three layers to find best execution:

1. **Li.Fi** - cross-chain bridge + DEX aggregator.
2. **BtrDex** - BTR's own concentrated-liquidity venue (Phase 2).
3. **Btr-Swap** - BTR's in-house aggregator SDK (`@btr-supply/swap`) that brokers quotes across both.

This page details how routes are scored, when each layer is preferred, and how MEV protection is applied.
