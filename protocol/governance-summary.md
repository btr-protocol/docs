---
title: "Governance — Summary"
description: "Friendly summary of BTR governance: roles, timelock, on-chain voting (forward-looking)."
audience: user
type: explanation
status: planned
phase: 2
order: 12
lang: en
publish: true
---

# Governance — Summary

> **Placeholder** — content + screenshots forthcoming. See [GitHub issues](https://github.com/btr-protocol) for status.

## What it is

BTR is governed through a layered model:

1. **Admin & operational roles** today: a multisig + timelock controller administers upgrades, parameter changes, and emergency pauses. Role assignments are bounded by `ROTATION_TIMELOCK`, `FACTORY_TIMELOCK`, and friends (see [Roles and Timelock](../security/roles-and-timelock.md)).
2. **On-chain voting** later (Phase 2): once veBTR launches, BTR holders cast votes on proposals affecting fees, emissions, treasury spend, and protocol-level parameters.

> **Status**: Phase-1 governance is **advisory + multisig-admin**. Full on-chain voting is Phase-2.

## Why it matters to users

- **Predictable upgrades**: every protocol change passes through timelock with a public delay window.
- **Future voice**: lock BTR → veBTR → vote (Phase 2).
- **Audit trail**: all role rotations and parameter changes recorded on-chain.

## Deep dive

For full mechanics — proposal lifecycle, quorum, council scope, supermajority thresholds — see [`07. Governance.md`](./07.%20Governance.md).
