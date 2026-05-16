---
title: "Bug Bounty Program"
description: "Responsible disclosure program for BTR smart contracts: scope, reward tiers, safe harbor, and SLA"
audience: both
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: true
---
# BUG BOUNTY PROGRAM

*Last Updated: May 2026*

## 1. PURPOSE

BTR welcomes responsible security disclosures from independent researchers. This program defines the scope, reward tiers, safe-harbor protections, and reporting process.

## 2. SCOPE

### 2.1 In scope

- BTR ALM production smart contracts deployed at the canonical addresses published in the [BTR documentation](../reference/contract-addresses.md):
  - Vault
  - Dex Adapters
  - Swapper (chain singleton)
  - PriceProvider
  - AccessControl
- BTR ALM keeper bots and operator infrastructure to the extent that a vulnerability leads to direct user loss or protocol compromise.
- BTR front-end ([https://btr.supply](https://btr.supply)) to the extent that a vulnerability leads to user-fund loss, transaction tampering, or credential theft.

### 2.2 Out of scope

- Testnet deployments.
- Third-party smart contracts that BTR integrates with (Uniswap v3/v4, PancakeSwap V3/Infinity, Aerodrome, Algebra, Ramses, LiFi, Chainlink, ERC-7683 settlers).
- Forks of BTR contracts deployed by third parties.
- Theoretical attacks without a demonstrable path on production code.
- Issues already publicly disclosed or known to BTR (see existing [Audit Reports](./audit-reports.md)).
- UI bugs that do not lead to fund loss or transaction tampering.
- Best-practice or styling concerns absent a security impact.
- Vulnerabilities requiring physical access to a user's device or privileged compromise of a user's wallet.

## 3. REWARD TIERS

Reward sizing is determined by severity, exploitability, and impact on users. Final classification rests with BTR security; we will be transparent about the rationale.

| Severity | Description                                                                 | Reward range (indicative)             |
|----------|-----------------------------------------------------------------------------|---------------------------------------|
| Critical | Direct, unauthorised loss of user or protocol funds; permanent freeze       | Up to USD 250,000 (or per Immunefi schedule once listed) |
| High     | Significant fund loss requiring non-trivial preconditions; long freeze      | USD 25,000 - 100,000                  |
| Medium   | Limited fund loss; governance bypass; oracle / accounting deviation         | USD 5,000 - 25,000                    |
| Low      | Griefing; minor accounting drift; DoS without fund loss                     | USD 500 - 5,000                       |

Rewards are paid in stablecoin or BTR-treasury-approved assets at BTR's discretion. Rewards are conditional on the report meeting the requirements in §5.

If BTR lists this program on a third-party platform (e.g., Immunefi), the platform-published schedule controls and supersedes the indicative ranges above.

## 4. SAFE HARBOR

### Safe Harbor

To the maximum extent permitted by applicable law, BTR Foundation grants safe-harbor to good-faith security researchers conducting research within the scope of this Bug Bounty. Specifically:

- **United States - Computer Fraud and Abuse Act (CFAA, 18 U.S.C. § 1030):** No civil action will be brought against in-scope, good-faith research.
- **United States - DMCA § 1201:** Reverse-engineering of BTR smart contracts and front-end JavaScript for security-research purposes is authorized.
- **United Kingdom - Computer Misuse Act 1990:** Same as CFAA above.
- **European Union - NIS2 Directive (2022/2555):** Security research conducted under this policy qualifies as "coordinated vulnerability disclosure" for NIS2 purposes.
- **European Union - Cyber Resilience Act:** Reporting conducted under this policy meets responsible-disclosure standards.

### Out-of-scope and Safe-Harbor Revocation

Safe Harbor does NOT extend to:
- Researchers resident in sanctioned jurisdictions (see [Geofencing](./geofencing-and-sanctions-screening.md))
- Researchers extorting BTR or third parties under threat of disclosure
- Public disclosure prior to remediation (90-day default, may extend with mutual agreement; researcher and BTR co-publish post-remediation)
- Activity exceeding the minimum necessary for proof-of-concept

We adopt the Immunefi v2.3 disclosure template for non-listed researchers; Immunefi-listed researchers follow Immunefi's standard procedures.

Safe harbor is conditional on the researcher:

- making a good-faith effort to avoid privacy violations, service degradation, or destruction of data;
- not exploiting the vulnerability beyond what is necessary to demonstrate impact;
- not extracting more value than the minimum required to prove the bug;
- withholding public disclosure until BTR has remediated the issue or 90 days have elapsed, whichever is earlier;
- complying with all applicable laws.

BTR cannot authorise testing of third-party systems. Safe harbor does not extend to vulnerabilities discovered against out-of-scope assets.

## 5. RESPONSIBLE DISCLOSURE PROCESS

Submit reports to **security@btr.supply** encrypted with the BTR security PGP key (fingerprint published at [https://btr.supply/.well-known/security.txt](https://btr.supply/.well-known/security.txt) when available; otherwise request the key at the same address).

Reports should include:

- a clear technical description of the vulnerability;
- step-by-step reproduction (preferably with a forking-test or Foundry PoC);
- a proposed severity assessment;
- the researcher's preferred payout destination and contact details.

## 6. SLA

| Stage                              | Target                              |
|------------------------------------|-------------------------------------|
| Initial acknowledgement            | 2 business days                     |
| Triage and severity classification | 5 business days                     |
| Remediation plan                   | 10 business days for Critical/High  |
| Bounty decision                    | 30 business days post-remediation   |

## 7. DUPLICATES AND ATTRIBUTION

The first researcher to report a previously-unknown vulnerability with sufficient detail to reproduce is eligible for the bounty. Duplicate reports may receive partial credit at BTR's discretion. With consent, BTR will publicly credit researchers in the relevant [Audit Reports](./audit-reports.md) entry.
