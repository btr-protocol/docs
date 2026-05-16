---
title: "Conflicts and Operator Disclosure"
description: "Disclosure of keepers, multisig signers, treasury controllers, related-party adapter selection, and conflict-resolution policy"
audience: both
type: reference
status: planned
phase: n/a
order: 99
lang: en
publish: true
---
> 🚧 **Pre-launch document - do NOT rely on for legal/operational decisions.** Multisig signers, keeper addresses, and Foundation board memberships are placeholders. This document will be populated and re-signed before mainnet General Availability. Reading party should confirm finalization with the Foundation before treating any disclosure as binding.

# CONFLICTS AND OPERATOR DISCLOSURE

*Last Updated: May 2026*

## 1. PURPOSE

This notice discloses the operators of BTR's off-chain infrastructure and on-chain governance keys, the processes by which adapters and counterparties are selected, and the conflict-of-interest policy that governs operator decisions.

## 2. ON-CHAIN GOVERNANCE KEYS

### 2.1 Multisig signers

The BTR governance multisig is configured with **N-of-M** signers (default 3-of-5). Signer identities and addresses (placeholder pending publication):

| Role               | Signer name / handle | Address     |
|--------------------|----------------------|-------------|
| Core - Engineering | [NAME]               | [ADDRESS]   |
| Core - Engineering | [NAME]               | [ADDRESS]   |
| Core - Operations  | [NAME]               | [ADDRESS]   |
| Independent        | [NAME]               | [ADDRESS]   |
| Independent        | [NAME]               | [ADDRESS]   |

Addresses are published on-chain via the AccessControl contract. Signer rotation is subject to a 14-day timelock per the AccessControl rotation rules.

### 2.2 Treasury controller

The Treasury contract receives protocol fees and is controlled by the same multisig described in §2.1 unless a separate Treasury controller is designated by governance.

### 2.3 Keeper addresses

Keeper services operate at the following addresses (placeholder pending publication):

| Function                     | Address     |
|------------------------------|-------------|
| Oracle / heartbeat keeper    | [ADDRESS]   |
| Adapter rebalance keeper     | [ADDRESS]   |
| Defensive-ratchet kill bot   | [ADDRESS]   |
| Intent settlement keeper     | [ADDRESS]   |
| Cohort processing keeper     | [ADDRESS]   |

Keeper authority is scoped per the AccessControl `keeper` role. Keeper compromise does not, by design, permit theft of user funds, but may degrade or freeze protocol operation until rotation.

## 3. RELATED-PARTY ADAPTER SELECTION

BTR may, from time to time, select third-party concentrated-liquidity pools (Uniswap v3 / v4, PancakeSwap V3 / Infinity, Aerodrome Slipstream, Algebra, Ramses, others) as adapter targets for ALM vaults.

### 3.1 Selection process

- **Initial proposal**: a core contributor or community member proposes an adapter pairing (vault, pool, fee tier, tick-range parameters).
- **Risk review**: BTR security reviews the third-party pool's audit history, governance, oracle dependencies, and historical behaviour.
- **Parameter calibration**: BTR ops calibrates defensive parameters (deviation, heartbeat, slippage, interval) within the bounds enforced by the smart contracts.
- **Governance ratification**: post-issuance of BTR governance, all material adapter additions and rotations are subject to governance ratification per the [Governance Participation Notice](./governance-participation-notice.md).

### 3.2 Related-party disclosure

If any adapter, swapper integration, or treasury counterparty is a related party of a multisig signer, core contributor, or BTR-affiliated entity, the relationship will be disclosed here. As of the date above, no such related-party relationships are known.

## 4. CONFLICT-OF-INTEREST POLICY

Multisig signers, keepers, and core contributors are expected to:

- recuse themselves from decisions in which they have a direct or indirect material interest;
- disclose such interests to the rest of the multisig and to the community via the BTR governance forum;
- abstain from trading BTR-related tokens on the basis of material non-public information.

Violations are addressed by the BTR core team and, where applicable, by the relevant Cayman foundation board.

## 5. BOARD COMPOSITION

The BTR Foundation board (Cayman Islands) (placeholder pending formal incorporation and appointment):

| Role                 | Name        |
|----------------------|-------------|
| Director             | [NAME]      |
| Director             | [NAME]      |
| Independent director | [NAME]      |

The board's duties are owed to the Foundation in accordance with its constitution.

## 6. UPDATES

This notice is updated as signers, keepers, addresses, and board composition change. Users should refresh before relying on the addresses listed herein and verify against the on-chain AccessControl contract.
