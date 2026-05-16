---
title: "Geofencing and Sanctions Screening"
description: "Technical controls and self-attestation regime for jurisdictional restrictions and sanctions compliance"
audience: both
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: true
---
# GEOFENCING AND SANCTIONS SCREENING

*Last Updated: May 2026*

## 1. PURPOSE

This notice describes the technical and self-attestation controls BTR uses to enforce the jurisdictional restrictions and sanctions-screening requirements in the [Terms of Service](./terms-of-service.md).

## 2. JURISDICTIONAL CONTROLS (FRONT-END)

The BTR front-end ([https://btr.supply](https://btr.supply)) is configured to:

- **IP-based geo-blocking**: requests from IP addresses geolocated to a Restricted Country listed in [Terms of Service §2](./terms-of-service.md) are blocked at the edge via the Cloudflare Web Application Firewall ("CF WAF"). Blocked users receive an HTTP 451 ("Unavailable For Legal Reasons") response where supported.
- **VPN / proxy disclaimer**: users are informed that circumventing geographic restrictions via VPN, proxy, or other location-obfuscation technology is a breach of the Terms.
- **Country self-attestation**: on first interaction, users are asked to self-attest that they are not resident in, citizen of, or accessing the Services from a Restricted Country.

These controls are imperfect and may be circumvented. Circumvention by the user is a breach of the Terms and does not create any obligation on BTR to permit access.

## 3. SANCTIONS SCREENING

BTR's restricted-list scope includes:

- jurisdictions subject to comprehensive sanctions by the U.S. Office of Foreign Assets Control ("OFAC");
- jurisdictions subject to comprehensive sanctions by the EU, UK HMT, UN Security Council, or Switzerland SECO;
- the Restricted Countries listed in [Terms of Service §2](./terms-of-service.md).

### 3.1 Wallet screening

For interactions that route through BTR-operated infrastructure (front-end, keeper services, intent routing), wallet addresses may be screened against industry-standard sanctions and risk datasets.

- **Current posture**: BTR self-attests adherence to sanctions screening in its operational policies. Where a third-party screening provider (e.g., TRM Labs, Chainalysis) is integrated, the integration will be disclosed here once live.
- **Counterparty action**: addresses identified as sanctioned, OFAC-listed, or otherwise high-risk per BTR's risk policy may be denied front-end access. The on-chain smart contracts remain permissionless; this control is a front-end / infrastructure control only.

### 3.2 Direct on-chain interaction

Users interacting directly with the smart contracts (bypassing the front-end) are nonetheless subject to all applicable laws, including sanctions regimes. BTR does not authorise sanctioned use of the Services.

## 4. APPEAL PROCESS

A user who believes their IP block or wallet block is in error may appeal by emailing **compliance@btr.supply** (this mailbox is monitored on a reasonable-best-efforts basis; response SLA: 5 business days) with:

- a statement of the user's country of residence and citizenship;
- the wallet address (if applicable);
- a brief description of the issue.

BTR will respond within a reasonable period. BTR is not obligated to remove any block and may decline appeals at its discretion based on the available information.

## 5. NO WAIVER OF USER RESPONSIBILITY

Nothing in this notice waives any user's obligation under the [Terms of Service](./terms-of-service.md) to comply with the law and the Restricted Country regime. Successful access via circumvention does not create any right of use.

## 6. UPDATES

BTR may add, remove, or modify the technical controls described here at any time without notice. This notice reflects the current posture as of the date above.
