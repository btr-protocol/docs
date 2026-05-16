---
title: "ACKNOWLEDGEMENT"
description: "Phase-1 ALM user acknowledgement for BTR Supply"
audience: both
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: true
---
# ACKNOWLEDGEMENT

*Last Updated: May 2026*

**Phase 1 Scope:** this acknowledgement covers BTR Supply (the ALM product) only. AIMM DEX-specific acknowledgements are deferred to [Acknowledgement (AIMM, Phase 2)](./acknowledgement-aimm-phase2.md) and are conditional on AIMM mainnet launch.

## RISK ACKNOWLEDGMENT

You are using BTR Supply -the BTR ALM (Automated Liquidity Management) vault system -which is experimental software subject to ongoing development. Use is entirely at your own risk.

**By accessing BTR Supply, you acknowledge and accept:**

1. **Full Responsibility**: You take sole responsibility for your use of the protocol and all associated risks. You have read and understand:
   - [Risk Disclaimer](./risk-disclaimer.md) -Comprehensive risk disclosures, including ALM-specific sections §7.5–§7.12
   - [Terms of Service](./terms-of-service.md) -Legal agreement and conditions, including ALM-specific Section 20

2. **Potential for Total Loss**: Using the protocol may result in partial or complete loss of deposited funds. This includes:
   - Loss from smart-contract bugs in BTR ALM contracts (vault / adapter / Swapper / PriceProvider / AccessControl)
   - Loss from third-party DEX pool bugs, exploits, or shutdown (Uniswap v3 / v4, PancakeSwap, Aerodrome, Algebra, Ramses, others)
   - Loss from impermanent loss (IL), loss-versus-rebalancing (LVR), tick-range exit drag, or fee-tier mismatch on Dex adapters
   - Loss from oracle deviation, sequencer outage, kill-switch activation, or governance authority compromise
   - Loss from intent / atomic-swap counterparty default
   No compensation will be provided for any losses.

3. **No Guarantees**: The protocol is provided "as is" without warranties. Smart contracts may contain bugs. Prices may be volatile. Transactions are irreversible. ALM vault NAV may fluctuate non-monotonically due to pessimistic worst-of(pool, oracle) accounting. Asynchronous redemptions may be subject to cohort haircuts.

4. **Geographic Compliance**: BTR Supply services are **NOT available** to residents, citizens, or entities in restricted jurisdictions. The full list of restricted countries is specified in [Terms of Service, Section 2](./terms-of-service.md#2-restrictions-of-use). If you are located in a restricted jurisdiction, you may not access or use the Services. Geographic restrictions apply regardless of VPN or proxy use.

5. **No Advice**: BTR does not provide financial, legal, or tax advice. You should consult qualified advisors before using the protocol. Vault performance metrics displayed on the Interface (yield, NAV, fee rates) are informational and do not predict future results.

6. **Third-Party DEX and Routing** (ALM): You acknowledge that ALM vaults allocate to third-party concentrated-liquidity pools and route swaps through third-party aggregators / settlers. BTR does not control these third parties and does not warrant their security or continued availability.

7. **Operator and Keeper Authority** (ALM): You acknowledge that BTR or its delegates hold multi-sig governance authority over Dex adapter configuration, swapper rotation, factory rotation, and treasury rotation, subject to on-chain timelocks (6 hours to 14 days depending on action) and rate limits (per-adapter kill 1/h; cluster cap 2/day). You accept the centralization risk inherent in this design.

8. **Audit Status**: You acknowledge that multiple internal audit cohorts have reviewed the BTR ALM contract stack (see [Audit Reports](./audit-reports.md)). Third-party audits remain in scope before mainnet General Availability. You will treat all software as experimental until third-party reports are published.

9. **Forward-Looking Items**: You acknowledge that cross-chain ALM allocation, additional DEX integrations, the AIMM DEX product, BTR token issuance, and future governance frameworks are forward-looking and not commitments. They may change, be delayed, or be cancelled. The AIMM DEX product, when deployed, will be governed by a separate acknowledgement.

### 10. No Fiduciary

User acknowledges that the BTR Foundation, its directors, contributors, and affiliates do not owe User any fiduciary duty arising from User's interaction with the Services. The Foundation may, but is not obligated to, take actions (including without limitation invoking the on-chain salvage path, pausing protocol functionality, or initiating wind-down) that protect or benefit Users; no such action creates an ongoing fiduciary undertaking.

## CONFIRMATION

By clicking "I Accept" or otherwise accessing the Services, you confirm that:
- You have read and understood the [Risk Disclaimer](./risk-disclaimer.md), including ALM-specific sections
- You have read and agreed to the [Terms of Service](./terms-of-service.md), including ALM-specific Section 20
- You are not located in a restricted jurisdiction
- You accept all risks and assume full responsibility for your use of the Services
- For ALM specifically: you understand vault accounting, fee structure, redemption mechanics, defensive parameters, and third-party DEX dependencies

## TECHNICAL SUPPORT

For technical issues, contact [contact@btr.supply](mailto:contact@btr.supply).
