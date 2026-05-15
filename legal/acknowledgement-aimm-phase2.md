---
title: "Acknowledgement (AIMM, Phase 2)"
description: "Forward-looking AIMM DEX-specific user acknowledgement, conditional on AIMM mainnet launch"
audience: both
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# ACKNOWLEDGEMENT - AIMM DEX (PHASE 2, FORWARD-LOOKING)

*Last Updated: May 2026*

**Status:** Planned. The AIMM DEX product is not currently deployed. This acknowledgement is forward-looking and only becomes binding upon mainnet launch of the AIMM DEX and an explicit user opt-in at that time. Nothing in this document constitutes a representation, commitment, or guarantee that the AIMM DEX will be launched, launched on any particular timeline, or launched with the parameters described below.

## SCOPE

This acknowledgement covers the **AIMM DEX** (Automated Inventory Market Maker) - a forward-looking multi-asset DEX with hub-and-spoke routing, dynamic fees, and coverage-ratio-based active liquidity management built on the ERC-2535 Diamond proxy pattern. It is separate from, and additional to, the Phase-1 [Acknowledgement](./acknowledgement.md) covering BTR Supply (the ALM product).

## AIMM-SPECIFIC ACKNOWLEDGEMENTS

By accessing the AIMM DEX (when deployed), you acknowledge and accept:

1. **Coverage Ratio Haircuts**: AIMM tracks per-asset coverage ratio (reserves / liabilities). When coverage falls below 1.0, LP withdrawals receive proportionally reduced amounts via a power-law haircut. At 0% coverage, LPs in that asset can suffer **total loss**. No insurance or backstop exists.

2. **Diamond Proxy Upgradeability**: AIMM uses the ERC-2535 Diamond proxy pattern. Protocol owners can add, replace, or remove facets, subject to timelocks (3-7 days). Even with timelocks, centralized upgrade authority could potentially bypass community consensus. Storage-collision risk applies to all Diamond facet additions.

3. **Hub-and-Spoke Routing**: Multi-asset swaps may route through intermediate hub assets. Each hop incurs fees, slippage, and exposure to the hub asset's coverage state. You accept routing risk and the possibility of partial-fill or reverted swaps.

4. **Dynamic Fees**: Trading fees on AIMM pools are dynamic and may change based on coverage ratio, volatility, and governance parameters. Quoted fees at swap-submit time may differ from executed fees within slippage tolerances.

5. **Flash Loan Exposure**: AIMM exposes ERC-3156 flash loans. Borrowers may use flash loans to orchestrate complex attacks across protocols, manipulate oracles, or exploit arithmetic edge cases. You accept the inherent flash-loan attack surface.

6. **MEV Exposure**: AIMM has no native MEV protection. Validators or searchers may front-run, sandwich, or extract value from your transactions.

7. **Governance Attack Risk**: Malicious actors may attempt to gain control of governance tokens to pass proposals detrimental to users (excessive minting, fee manipulation, unauthorized upgrades, treasury drain). You accept governance-attack risk.

8. **Audit Status**: You acknowledge that multiple internal audit cohorts have reviewed the BTR contract stack including AIMM (see [Audit Reports](./audit-reports.md)). Third-party audits remain in scope before AIMM mainnet General Availability. You will treat AIMM as experimental until third-party reports are published.

9. **Forward-Looking Disclaimer**: All statements about AIMM functionality, parameters, fee structure, governance, and launch timeline are forward-looking and subject to change, delay, or cancellation. Nothing constitutes a commitment or offer.

## CONDITIONAL CONFIRMATION

Upon AIMM mainnet launch and your explicit opt-in, you will confirm that:
- You have read and understood the [Risk Disclaimer](/risk-disclaimer), including DEX-specific sections
- You have read and agreed to the [Terms of Service](/terms-of-service), including AIMM-specific Section 20
- You are not located in a restricted jurisdiction
- You accept all AIMM-specific risks including coverage-ratio haircuts, Diamond upgradeability, dynamic fees, flash-loan exposure, MEV exposure, and governance-attack risk
- You assume full responsibility for your use of the AIMM DEX

## TECHNICAL SUPPORT

For technical issues, contact [contact@btr.supply](mailto:contact@btr.supply).
