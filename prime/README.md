---
title: "BTR Prime - Documentation Index"
description: "Mathematical methodology behind BTR Prime: adaptive Renko bars, Parkinson volatility, microstructure features, perpetual gradient boosting, walk-forward validation, directional and CL range output rules."
audience: tech
type: reference
status: live
phase: n/a
order: -1
lang: en
publish: true
---
# BTR Prime - Documentation Index

> Quantitative methodology behind BTR Prime: how perpetual gradient boosting on adaptive Renko bars + microstructure features powers both directional trading and concentrated-liquidity range management.

This documentation set is the **theoretical companion** to the production Rust code at `~/Work/btr/prime`. It is **not** a product manual - it is a math reference for internal quantitative review and external reproducibility. Every formula links to the file:line that implements it.

---

## Start here

- **[§00 Overview](./00.%20Overview.md)** - what Prime is, the pipeline at a glance, design invariants, conventions.

## The methodology (read in order)

| # | Page | One-line summary |
|---|---|---|
| 01 | [Information Bars](./01.%20Information%20Bars.md) | Why information time beats calendar time. Adaptive Renko brick construction. Variance homogenisation and near-Gaussianity theorems. |
| 02 | [Parkinson Volatility](./02.%20Parkinson%20Volatility.md) | Range-based variance estimator. GBM derivation from first principles. ~5x efficiency vs close-to-close. The universal σ-normalisation. |
| 03 | [Features](./03.%20Features.md) | 29 MTF per-bar + 12 Renko-tracker + 8 cross-asset (per alt) + 12 microstructure = **61** feature manifest; **32 active in production (Phase 42Z)**. Multi-timeframe scheme. Complete formula manifest with data-source provenance. |
| 04 | [Labels](./04.%20Labels.md) | Directional (LO / SO) labels with threshold-conviction decoupling. CL net-α-vs-HODL labels with Milionis-LVR dampening. Meta-labeling status. |
| 05 | [Perpetual Boosting](./05.%20Perpetual%20Boosting.md) | Friedman GBM to perpetual variant with complexity budget. Two-stage architecture (Stage-1 active, Stage-2 reserved). Inter-fold decay-weighted ensemble. |
| 06 | [Walk-Forward Validation](./06.%20Walk-Forward%20Validation.md) | Information-agnostic walk-forward. Purge + embargo invariants. Fitness as gated weighted geometric mean of monthly-geometric sub-scores. |
| 07 | [Directional Trading](./07.%20Directional%20Trading.md) | Signal to entry / SL / TP / leverage / position-fraction math. Long-only vs short-only separation. PnL accounting. |
| 08 | [CL Range Management](./08.%20CL%20Range%20Management.md) | Uniswap V3 invariant. LVR. Net-α label. σ-normalised projection to (lower, upper) ticks for the ALM Vault. |
| 09 | [Hyperparameters](./09.%20Hyperparameters.md) | Single canonical parameter reference: every tunable, its role, default, sensitivity. |
| 10 | [Open Questions](./10.%20Open%20Questions.md) | Honest catalogue of code↔doc mismatches, missing math, and the research roadmap. |

## Status

**Phase 1 (live):** all the math documented in §01-§08 reflects production code at master. Adaptive Renko bars, Parkinson volatility, feature pipeline, perpetual GBM (Stage-1 primary, decay-weighted inter-fold ensemble), walk-forward harness, fitness function, directional LO/SO output, CL range output.

**Phase 2 (planned, plumbed but inactive):** Stage-2 meta-classifier; intra-fold seed-diverse ensembling; explicit σ-floor trade rejection. See [§10 Open Questions §1](./10.%20Open%20Questions.md).

**Phase 3 (research):** GARCH/EGARCH overlay on σ; regime-detection meta-feature; tail-risk-aware fitness extensions; predictive-distribution outputs.

## Internal-only

- [Architecture (engineering view)](./architecture.md) - service inventory, deployment topology. `publish: false`.

## Audience & conventions

- **Audience:** quants, ML researchers, protocol engineers reviewing methodology before mainnet. Not for end-users.
- **LaTeX:** all math rendered via Temml; inline `$x$` and display `$$x$$`.
- **Code references:** form `path:line-range` against `~/Work/btr/prime/` unless prefixed.
- **Theorems:** numbered within each page (e.g. "Theorem 2.1" lives in §02).
- **Bibliography:** per-page, at the bottom.

## Cross-repo boundary

BTR Prime is a **consumer** of NX Rates. Canonical specs for upstream concepts live in `~/Work/nx/nx-rates`:

| Concept | Canonical source |
|---|---|
| `nxr_sdk::Bar` wire layout | `nx-rates/mitch/model/bar.md` |
| Composite Index (TDWAP) | `nx-rates/mitch/model/index.md` |
| TDWAP / staleness / triangulation | `nx-rates/docs/aggregation-methodology.md` |
| Adaptive Renko brick generator (ONE engine, hist == live) | `nx-rates/sdk/rust/src/renko.rs` (`RenkoGenerator`) |
| k calibrator (scale-to-target, rolling 365d) | `nx-rates/series-factory/src/bar_construction/calibrate.rs`; spec `nx-rates/docs/renko-methodology.md` |
| MTF Rogers-Satchell σ blender | `nx-rates/sdk/rust/src/vol.rs` (`vol_estimator::rs_sigma_from_ohlc`) |

This documentation **links** to those rather than re-specifying.

---

**Begin reading:** [§00 Overview →](./00.%20Overview.md)
