---
title: "MiCA EU User Notice"
description: "Markets in Crypto-Assets Regulation positioning for EU users: exemption rationale, non-classification, and CASP routing"
audience: both
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: true
---
# MiCA EU USER NOTICE

*Last Updated: May 2026*

## 1. PURPOSE

This notice explains BTR's position under Regulation (EU) 2023/1114 on Markets in Crypto-Assets ("MiCA") for users resident in, or accessing the Services from, the European Economic Area ("EEA").

This notice is **informational only** and does not constitute legal advice. MiCA interpretation, technical standards, and enforcement practice continue to evolve. Users should consult qualified legal counsel in their jurisdiction.

## 2. BTR TOKEN: MICA NON-CLASSIFICATION ANALYSIS

The BTR Token (when issued; see [BTR Token Notice](./btr-token-notice.md)) is intended to fall **outside** the three principal MiCA crypto-asset categories:

- **Not an Asset-Referenced Token (ART)** under MiCA Art. 3(1)(6): the BTR Token does not purport to maintain a stable value by referencing another value, right, or combination thereof (including fiat currencies).
- **Not an E-Money Token (EMT)** under MiCA Art. 3(1)(7): the BTR Token does not purport to maintain a stable value by referencing a single official currency.
- **Not a stablecoin in any form**.

The BTR Token is intended to be a **utility / governance** crypto-asset under MiCA Art. 3(1)(5) (other crypto-asset), subject to the residual offering rules in Title II.

## 3. MICA OFFERING-EXEMPTION RATIONALE

### 2. MiCA Threshold and Non-Public-Offer Analysis

BTR's position on MiCA applicability rests on the following analysis (not legal advice; subject to outside-counsel sign-off pre-EEA distribution):

**(a) Article 4(2) thresholds.** MiCA Title II (Art. 4–14) regulates *offers to the public of crypto-assets other than ART/EMT*. Article 4(2)(c) exempts offers below **EUR 1,000,000 over a 12-month period** to EEA persons. BTR's distribution plan caps EEA participation below this threshold during the transition window. If aggregate EEA participation approaches the threshold, BTR will publish a MiCA-compliant white paper before exceeding it.

**(b) Non-public-offer carve-out.** Offers limited to fewer than 150 persons per Member State (Art. 4(2)(b)) or restricted to qualified investors (Art. 4(2)(a)) are also out-of-scope. BTR's geofencing controls (see [Geofencing & Sanctions Screening](./geofencing-and-sanctions-screening.md)) implement these limits.

**(c) Not an ART/EMT.** BTR token is not an asset-referenced token (Art. 3(1)(6)) - no peg, no basket, no reference to fiat/commodity stability. It is not an e-money token (Art. 3(1)(7)) - no single-fiat-currency redemption. It is a "crypto-asset other than ART/EMT" subject only to Title II if offered to the EEA public.

**(d) Recital 22 / decentralization considerations.** ESMA's Final Report on MiCA (October 2024) clarifies that fully-decentralized services without an identifiable issuer or service provider fall outside MiCA Titles III, IV, V (CASP authorization). BTR Supply's vault contracts have no centralized issuer; governance is conducted via on-chain proposal and timelock. This is a fact-pattern analysis rather than a regulatory exemption.

**Article 109 (transitional provisions)** applies only to crypto-assets in circulation before 30 December 2024. BTR token has not yet been issued. Art. 109 does NOT apply.

**No CASP authorization claim.** BTR does not operate as a CASP within the EEA. Users routing transactions through BTR contracts do so directly; no order-handling, custody, or advice service is offered by the BTR Foundation to EEA users.

**Watchlist.** ESMA Q&A updates on "fully decentralized" criteria, and any Member State guidance on MiCA interaction with TFR (Travel Rule), are tracked by BTR Foundation and may trigger doc updates.

## 4. CASP ROUTING (FRONT-END BEHAVIOUR)

MiCA Titles III-V regulate Crypto-Asset Service Providers ("CASPs"). BTR does not operate a CASP. The BTR front-end ([https://btr.supply](https://btr.supply)) provides:

- non-custodial, self-executed interactions with on-chain smart contracts;
- read-only display of protocol state;
- routing to third-party intent settlers and atomic-swap routers for cross-chain execution.

For EEA users, the front-end may:

- restrict access to specific surfaces based on MiCA classification of the underlying activity;
- route swap and bridging requests to MiCA-authorised CASPs where available;
- decline to facilitate activities that, under MiCA, would require BTR to operate as a CASP without authorisation.

## 5. NO CASP STATUS REPRESENTATION

BTR does **not** hold a CASP authorisation under MiCA and does **not** represent itself as a CASP. Users acknowledge that the Services described in the [Terms of Service](./terms-of-service.md) are provided by autonomous, non-custodial smart contracts.

## 6. FORWARD-LOOKING

Regulatory technical standards under MiCA, ESMA guidelines, and national competent authority practice continue to evolve. This notice will be updated as needed.
