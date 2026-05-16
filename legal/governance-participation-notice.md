---
title: "Governance Participation Notice"
description: "Voting framework, eligibility, quorum/timelock, and disclaimer that votes are signals, not contractual offers"
audience: both
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# GOVERNANCE PARTICIPATION NOTICE

*Last Updated: May 2026*

## 1. PURPOSE

This notice describes the BTR governance process and the legal nature of participation. It supplements, and does not replace, the [Terms of Service](./terms-of-service.md) and [Risk Disclaimer](./risk-disclaimer.md).

Note: parts of the governance framework below are forward-looking and conditional on the issuance of the BTR Token (see [BTR Token Notice](./btr-token-notice.md)). Pre-issuance, governance is conducted off-chain by core contributors with community input.

## 2. VOTE = SIGNAL, NOT CONTRACTUAL OFFER

Casting a vote in BTR governance is a **signal of preference**. It does not:

- create any contractual obligation between any voter and BTR;
- bind BTR (the project, its contributors, the DAO, or any legal entity) to implement the outcome;
- create a fiduciary, agency, partnership, or trust relationship between BTR and any voter or token holder;
- constitute an offer, sale, or distribution of any token, security, or financial instrument.

The protocol's smart contracts will execute on-chain proposals that pass according to the on-chain rules; off-chain Snapshot votes are indicative and may be implemented at the discretion of the multisig signers subject to existing operational constraints.

## 3. VOTER ELIGIBILITY

Voting eligibility is restricted to holders of:

- the BTR Token (post-issuance) with voting power proportional to BTR balance;
- staked BTR (sBTR) with conviction-weighted voting power per the staking schedule;
- where applicable, sLP (staked LP) tokens for surface-specific votes.

Voters must not be located in a Restricted Country listed in [Terms of Service §2](./terms-of-service.md). Voters resident in jurisdictions where participation in decentralised governance would require registration, licensing, or regulatory approval not held by the voter are responsible for ensuring their participation is lawful.

## 4. QUORUM AND TIMELOCK

The following parameters are indicative and subject to governance approval:

| Parameter             | Target value          |
|-----------------------|-----------------------|
| Proposal threshold    | 0.1% of voting supply |
| Voting period         | 5-7 days              |
| Quorum                | 4% of voting supply   |
| Approval threshold    | Simple majority       |
| Execution timelock    | 6 hours - 14 days, depending on action category |

Critical operations (treasury rotation, swapper rotation, kill-switch deactivation) sit at the upper end of the timelock range. Routine parameter changes sit at the lower end.

## 5. NO FIDUCIARY DUTY

BTR contributors, multisig signers, keepers, and counsel do **not** owe fiduciary duties to any voter or token holder. Their duties (where any are owed at all) are owed to the relevant Cayman foundation or DAO entity in accordance with its constitution.

## 6. OFF-CHAIN SNAPSHOT AND ON-CHAIN EXECUTION

The governance flow is:

1. **Forum discussion** - proposals are discussed in the BTR governance forum.
2. **Snapshot vote** - off-chain signature-based voting via Snapshot. Results are advisory.
3. **On-chain proposal** - if Snapshot is supportive, a corresponding on-chain proposal is filed.
4. **On-chain vote** - executed via the governance contract, subject to quorum and approval threshold.
5. **Timelock** - successful proposals enter the timelock queue.
6. **Execution** - after timelock expiry, any account may execute the proposal. On-chain auto-execution is a property of the immutable timelock and proposal contracts; it is not a BTR Foundation undertaking. Timelock guardians may cancel proposals during the timelock window in accordance with the Foundation's emergency authority (see [Pause & Emergency](../security/pause-and-emergency.md)).

## 7. FORWARD-LOOKING STATEMENT

All governance parameters, processes, and timelines in this notice are forward-looking and subject to change. BTR makes no commitment to any particular governance design, parameter, or timeline. See the forward-looking-statement framing in the [BTR Token Notice](./btr-token-notice.md).
