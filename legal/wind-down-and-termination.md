---
title: "Wind-Down and Termination"
description: "Cease-of-operations playbook: cohort ticket fate, on-chain rescue path, user self-custody emphasis, and communications"
audience: both
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: true
---
# WIND-DOWN AND TERMINATION

*Last Updated: May 2026*

## 1. PURPOSE

This notice describes what happens to user positions, redemption cohorts, and protocol assets if BTR (the operator, the front-end, or the keeper services) ceases operations - whether voluntarily, due to regulatory action, or following a security incident.

## 2. CORE PRINCIPLE: SMART CONTRACTS PERSIST

The BTR ALM smart contracts are deployed on Ethereum and compatible EVM chains. They continue to operate independently of:

- the BTR front-end ([https://btr.supply](https://btr.supply));
- BTR-operated keeper services;
- the BTR legal entity (the Foundation, the DAO, or any contributor);
- BTR-operated communications channels (X, Discord, GitHub).

Users retain control of their share tokens (ERC-20 / ERC-7540) and may interact with the smart contracts directly using any compatible interface or self-built tooling.

## 3. COHORT TICKET FATE

ERC-7540 redemption tickets ("cohort tickets") have a defined on-chain lifecycle:

- **Request phase**: deposits are recorded; the cohort accumulates until cutover.
- **Cutover**: the cohort closes and enters processing.
- **Processing**: assets are withdrawn from adapters, swapped if needed, and queued for distribution.
- **Settled**: assets are distributable to ticket holders pro-rata.

If BTR ceases operations:

- **Tickets already past cutover** continue processing as long as the cohort-processing keeper logic can be invoked. Any party may invoke the relevant permissionless functions to advance settlement if the BTR keeper is offline.
- **Pending tickets at request phase** may be cancellable per the on-chain rules, returning the user to the synchronous-redeem path or to direct withdrawal of share tokens.
- **Cohort haircuts** (if any) are applied per the on-chain pessimistic NAV accounting, independently of BTR's operational status.

Users with active cohort tickets should consult the on-chain contract directly to determine ticket state and available actions.

## 4. ON-CHAIN RESCUE PATH

The Treasury contract exposes a `salvage` function that, subject to AccessControl authority and timelock, can:

- redeem residual adapter positions to underlying assets;
- route via the Swapper singleton to consolidate funds in a recoverable form;
- transfer funds to the Treasury for pro-rata distribution to share holders.

In a wind-down scenario, BTR (or its successors per the AccessControl ownership-handover process) may invoke the on-chain salvage path; BTR Foundation is not obligated to do so and acts at its discretion subject to governance approval. If the multisig is compromised or non-responsive, the timelock and rate-limit constraints in AccessControl provide a window for user response (revoke approvals, withdraw share tokens, etc.).

## 5. USER SELF-CUSTODY EMPHASIS

Users retain custody of their share tokens at all times. Best practices for wind-down resilience:

- Keep share tokens in a wallet whose seed phrase you control. Do **not** leave share tokens on a custodial interface.
- Record the canonical contract addresses (Vault, Adapter set, Swapper, PriceProvider, AccessControl) from the [BTR documentation](../reference/contract-addresses.md).
- Familiarise yourself with on-chain redemption paths (synchronous redeem; ERC-7540 request) before you need to use them in adversarial conditions.
- Subscribe to communication channels (§6) so that you receive wind-down notifications.

## 6. COMMUNICATIONS DURING WIND-DOWN

In the event of a planned wind-down, BTR will provide notice via:

- **Email** to [contact@btr.supply](mailto:contact@btr.supply) subscribers (where users have opted in);
- **X / Twitter**: [@BtrSupply](https://x.com/BtrSupply);
- **Governance forum** announcement;
- **On-chain event** via the Treasury or AccessControl contract for material status changes.

In the event of an unplanned termination (e.g., regulatory shutdown or core-team incapacity), communications may be limited or absent. Users should monitor on-chain state directly.

### 6.5. Regulatory Notifications

In the event of a planned cessation of operations, BTR Foundation will:
- File a notice with the Cayman Islands Monetary Authority (CIMA) if applicable, and with the Cayman Islands Registry of Companies (Companies Act s.156 strike-off, where applicable).
- Comply with FATCA/CRS reporting obligations for any reportable accounts maintained by the Foundation during wind-down.
- Cooperate with applicable home-jurisdiction regulators of users where regulatory information requests are received.

## 7. POST-TERMINATION OBLIGATIONS

After cessation of operations:

- BTR's obligations under the [Terms of Service](./terms-of-service.md) survive only to the extent expressly stated therein (typically: limitation of liability, indemnification, governing law, dispute resolution).
- BTR is not obligated to continue operating the front-end, keepers, or oracle-update bots.
- BTR is not obligated to maintain documentation, websites, or communication channels.

## 8. NO COMPENSATION

No compensation, insurance, or backstop exists for losses arising from BTR's cessation of operations, including losses from forced exit at unfavourable prices, cohort haircuts, adapter shutdowns, or oracle staleness. See [Risk Disclaimer](./risk-disclaimer.md) and [Acknowledgement](./acknowledgement.md).

## 9. TAX CHARACTERIZATION (NO ADVICE)

Wind-down events may have tax consequences for users. BTR does not provide tax advice. Users should consult qualified counsel re:
- **§1001 (US) realization** on cohort-ticket conversion to salvaged tokens
- **CGT** (UK/EU) treatment of forced exit
- **Cost-basis carryover** vs. realization at salvage
- **FATCA/CRS account closure** reporting if applicable

BTR cooperates with user-side tax reporting on a reasonable-best-efforts basis but makes no representation as to characterization in any jurisdiction.
