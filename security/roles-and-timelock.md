---
title: "Roles and Timelock"
description: "BTR multisig signers, role assignments, and timelock parameters."
audience: both
type: reference
status: planned
phase: 1
order: 5
lang: en
publish: true
last_reviewed: 2026-05-15
---

# Roles and Timelock

## Multisig signers

Active signers and threshold will be listed here once the deployment Safe is finalized. *(placeholder)*

## Timelock parameters

| Parameter | Value | Notes |
|---|---|---|
| `ROTATION_TIMELOCK` | 7 days | Role-rotation grant acceptance window |
| `FACTORY_TIMELOCK` | 14 days | New-vault factory deployment delay |
| `ADMIN_TIMELOCK` | 7 days | Admin role re-assignment delay |
| `UPGRADE_TIMELOCK` | 14 days | UUPS upgrade activation delay |

## Upgrade procedure

1. Proposal queued via timelock controller.
2. Public delay window (see UPGRADE_TIMELOCK).
3. Execution by `UPGRADER` role through multisig.
4. Post-upgrade verification by `KEEPER` and external monitors.

Detailed runbook to be added.
