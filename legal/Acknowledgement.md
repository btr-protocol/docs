# ACKNOWLEDGEMENT

*Last Updated: May 2026*

## RISK ACKNOWLEDGMENT

You are using BTR -comprising the AIMM decentralized exchange protocol and the BTR ALM (Automated Liquidity Management) vault system -which is experimental software subject to ongoing development. Use is entirely at your own risk.

**By accessing BTR, you acknowledge and accept:**

1. **Full Responsibility**: You take sole responsibility for your use of the protocol and all associated risks. You have read and understand:
   - [Risk Disclaimer](/risk-disclaimer) -Comprehensive risk disclosures, including ALM-specific sections §7.5–§7.12
   - [Terms of Service](/terms-of-service) -Legal agreement and conditions, including ALM-specific Section 20

2. **Potential for Total Loss**: Using the protocol may result in partial or complete loss of deposited funds. This includes:
   - Loss from smart-contract bugs in BTR contracts (DEX or ALM)
   - Loss from third-party DEX pool bugs, exploits, or shutdown (Uniswap v3 / v4, PancakeSwap, Aerodrome, Algebra, Ramses, others)
   - Loss from impermanent loss (IL), loss-versus-rebalancing (LVR), tick-range exit drag, or fee-tier mismatch on Dex adapters
   - Loss from oracle deviation, sequencer outage, kill-switch activation, or governance authority compromise
   - Loss from intent / atomic-swap counterparty default
   No compensation will be provided for any losses.

3. **No Guarantees**: The protocol is provided "as is" without warranties. Smart contracts may contain bugs. Prices may be volatile. Transactions are irreversible. ALM vault NAV may fluctuate non-monotonically due to pessimistic worst-of(pool, oracle) accounting. Asynchronous redemptions may be subject to cohort haircuts.

4. **Geographic Compliance**: BTR services are **NOT available** to residents, citizens, or entities in restricted jurisdictions. The full list of restricted countries is specified in [Terms of Service, Section 2](/terms-of-service#2-restrictions-of-use). If you are located in a restricted jurisdiction, you may not access or use the Services. Geographic restrictions apply regardless of VPN or proxy use.

5. **No Advice**: BTR does not provide financial, legal, or tax advice. You should consult qualified advisors before using the protocol. Vault performance metrics displayed on the Interface (yield, NAV, fee rates) are informational and do not predict future results.

6. **Third-Party DEX and Routing** (ALM): You acknowledge that ALM vaults allocate to third-party concentrated-liquidity pools and route swaps through third-party aggregators / settlers. BTR does not control these third parties and does not warrant their security or continued availability.

7. **Operator and Keeper Authority** (ALM): You acknowledge that BTR or its delegates hold multi-sig governance authority over Dex adapter configuration, swapper rotation, factory rotation, and treasury rotation, subject to on-chain timelocks (6 hours to 14 days depending on action) and rate limits (per-adapter kill 1/h; cluster cap 2/day). You accept the centralization risk inherent in this design.

8. **Forward-Looking Items**: You acknowledge that cross-chain ALM allocation, additional DEX integrations, BTR token issuance, and future governance frameworks are forward-looking and not commitments. They may change, be delayed, or be cancelled.

## CONFIRMATION

By clicking "I Accept" or otherwise accessing the Services, you confirm that:
- You have read and understood the [Risk Disclaimer](/risk-disclaimer), including ALM-specific sections
- You have read and agreed to the [Terms of Service](/terms-of-service), including ALM-specific Section 20
- You are not located in a restricted jurisdiction
- You accept all risks and assume full responsibility for your use of the Services
- For ALM specifically: you understand vault accounting, fee structure, redemption mechanics, defensive parameters, and third-party DEX dependencies

## TECHNICAL SUPPORT

For technical issues, contact [contact@btr.supply](mailto:contact@btr.supply).
