---
title: "Security Overview"
description: "How BTR Protocol approaches security — audits, bounties, pause controls, and roles."
audience: both
type: reference
status: planned
phase: 1
order: 1
lang: en
publish: true
last_reviewed: 2026-05-15
---

# Security Overview

> ⚠ **Pre-launch security profile.** All operational artifacts below (multisig signers, status page, incident channel, audit firm, deployment addresses) will be populated before mainnet General Availability. Until that population, treat BTR contracts as experimental.

## At a glance

| Aspect | Value | Source |
|--------|-------|--------|
| Independent audit firm(s) | *Pending — to be announced before GA* | [Audit Reports](../legal/audit-reports.md) |
| Multisig configuration | *m-of-n: configuration TBD* | [Roles & Timelock](./roles-and-timelock.md) |
| Pause authority | Foundation Owner (timelocked rotation) | [Pause & Emergency](./pause-and-emergency.md) |
| Status page | *To be announced* | — |
| Incident channel | *To be announced* | — |
| Bug bounty | Self-hosted (Immunefi onboarding TBD) | [Bug Bounty](../legal/bug-bounty.md) |
| Timelock rotation | 7 days (Treasury, Swapper, Staking), 14 days (Factory) | [Roles & Timelock](./roles-and-timelock.md) |
| Contract addresses | *Pending mainnet deployment* | [Contract Addresses](../reference/contract-addresses.md) |

## Pillars

1. **Independent third-party audits** before mainnet GA. Internal audit cohorts (Phase 42C/L/M/N/O) close findings prior to handover.
2. **Timelock governance** on all parameter rotations. No instant-flip authority on Treasury / Swapper / Staking / Factory.
3. **Pause + Emergency** procedures, with on-chain salvage as the user-protection backstop. See [Pause & Emergency](./pause-and-emergency.md).
4. **Bug Bounty** with safe-harbor for good-faith research. See [Bug Bounty](../legal/bug-bounty.md).

## Sub-pages
- [Audits](./audits.md) — pointer to [Audit Reports](../legal/audit-reports.md)
- [Bug Bounty](./bug-bounty.md) — pointer to [Bug Bounty](../legal/bug-bounty.md)
- [Pause & Emergency](./pause-and-emergency.md)
- [Roles & Timelock](./roles-and-timelock.md)
- [Contract Addresses](./contracts.md) — pointer to [Contract Addresses](../reference/contract-addresses.md)
