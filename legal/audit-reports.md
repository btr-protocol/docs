---
title: "Audit Reports"
description: "Index of internal and third-party security audits performed on the BTR contract stack"
audience: both
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: true
last_reviewed: 2026-05-15
---
> ⚠ **Internal review only.** The audit-cohort summaries in this document represent BTR's internal review process. Third-party audit reports (PDFs) are pending. **No warranty of completeness, accuracy, or future security is made.** Users and integrators should treat BTR contracts as experimental until independent third-party audits are completed and published. See [Risk Disclaimer §2.1](./risk-disclaimer.md) for the canonical audit-status statement.

# AUDIT REPORTS

*Last Updated: May 2026*

## 1. OVERVIEW

This page indexes all security-audit cohorts conducted on the BTR contract stack. Each entry summarises scope, completion date, findings, and remediation status. Internal cohorts are conducted by BTR-affiliated security reviewers. Third-party audits remain in scope before mainnet General Availability and will be added to this index upon completion.

All software covered by these audits should be treated as experimental until third-party audit reports are published.

## 2. AUDIT INDEX

### 2.1 Phase 42C R1-R18 - DEX audit infinity loop (Internal)

- **Scope:** AIMM Diamond facets, hub-and-spoke routing, coverage-ratio accounting, dynamic fees, ERC-3156 flash-loan path.
- **Completion date:** 2025-12 through 2026-01 (iterative).
- **Findings summary:** 18 review rounds with iterative HIGH/MED/LOW issue identification and remediation. Scope was AIMM Diamond + routing + flash-loan path; internal status: HIGH findings addressed in final round per internal sign-off; no third-party verification yet.
- **Remediation status:** Internal sign-off; pending third-party verification.
- **Report:** *(coming soon - report PDF being prepared)*

### 2.2 Phase 42K.10D - dex/alm Solidity realignment (Internal, 42 passes)

- **Scope:** Cross-cutting realignment between DEX and ALM Solidity surfaces; AccessControl, Treasury, salvage path, adapter authority model.
- **Completion date:** 2026-02 (42 passes).
- **Findings summary:** Scope was cross-cutting realignment between DEX/ALM surfaces; internal status: collector and QDB integration test cohorts reported as converged per BTR internal review; no third-party verification yet.
- **Remediation status:** Internal sign-off; deploy runbook drafted; pending third-party verification.
- **Report:** *(coming soon - report PDF being prepared)*

### 2.3 Phase 42L - Pass-44 through Pass-47 (Internal, mainnet-readiness)

- **Scope:** Mainnet-readiness review across 8 repos. Focus on UUPS upgradeability storage gaps, PoolBatch sizing, MEV exposure, oracle deviation, sequencer guard.
- **Completion date:** 2026-05-15.
- **Findings summary:** Scope was mainnet-readiness across 8 repos (UUPS storage gaps, PoolBatch sizing, MEV exposure, oracle deviation, sequencer guard); internal status: 14 review cohorts + 4 remediation cohorts; 2 HIGH items identified as mainnet blockers were addressed per BTR internal review (UUPS `__gap` added; PoolBatch reduced by 5,152 bytes); no third-party verification yet.
- **Remediation status:** Internal sign-off for in-scope items; third-party audit pending before General Availability.
- **Report:** *(coming soon - report PDF being prepared)*

### 2.4 Phase 42M - (Internal)

- **Scope:** *(to be confirmed in published report)*.
- **Completion date:** *(see report)*.
- **Findings summary:** *(see report)*.
- **Remediation status:** *(see report)*.
- **Report:** *(coming soon - report PDF being prepared)*

### 2.5 Phase 42N - (Internal)

- **Scope:** *(to be confirmed in published report)*.
- **Completion date:** *(see report)*.
- **Findings summary:** *(see report)*.
- **Remediation status:** *(see report)*.
- **Report:** *(coming soon - report PDF being prepared)*

### 2.6 Phase 42O - (Internal)

- **Scope:** *(to be confirmed in published report)*.
- **Completion date:** *(see report)*.
- **Findings summary:** *(see report)*.
- **Remediation status:** *(see report)*.
- **Report:** *(coming soon - report PDF being prepared)*

### 2.7 Third-Party Audits (Planned)

- **Status:** In scope. Engagements to be announced.
- **Reports:** *(coming soon - report PDF being prepared)*

## 3. FINDINGS DISCLOSURE POLICY

BTR commits to publishing each audit report (or a clean public summary) once remediation is complete and any responsible-disclosure window has closed. Critical findings active at the time of publication will be redacted until remediation is verified on-chain.

## 4. REPORTING NEW VULNERABILITIES

Researchers identifying new vulnerabilities should follow the [Bug Bounty Program](./bug-bounty.md) responsible-disclosure process. Do not file new vulnerabilities as audit-report comments.
