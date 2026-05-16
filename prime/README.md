---
title: "BTR Prime - Documentation Index"
description: "ML-driven trading platform for crypto spot + concentrated liquidity. Consumes Bar streams from NX Rates."
audience: tech
type: reference
status: live
phase: n/a
order: 99
lang: en
publish: false
---
# BTR Prime - Documentation Index

ML-driven trading platform for crypto spot + concentrated liquidity. Consumes Bar streams from NX Rates.

## Where to start

| If you... | Read |
|-----------|------|
| Need the service architecture (optimizer, executor, providers) | [architecture.md](architecture.md) |
| Need the ML methodology (features, GBM, fitness) | [methodology.md](methodology.md) |
| Need NXR's data model / wire format | `../../nx/nx-rates/mitch/model/` |
| Need the composite TDWAP maths | `../../nx/nx-rates/docs/aggregation-methodology.md` |

## Two views

`architecture.md` and `methodology.md` cover orthogonal concerns:

- **architecture.md** -services, processes, K8s resources, data flow, CLI surface, file layout. Read this when wiring up infra or onboarding a new strategy binary.
- **methodology.md** -ML algorithm: features, walk-forward GBM, Parkinson labeling, fitness function, trading rules. Read this when changing the model or interpreting results.

## Cross-repo boundary

BTR is a **consumer** of NX Rates. Canonical specs for upstream concepts live in `~/Work/nx/nx-rates`:

| Concept | Canonical source |
|---------|------------------|
| `mitch::Bar` (96 B) wire layout | nx-rates/mitch/model/bar.md |
| Composite Index (TDWAP) frame | nx-rates/mitch/model/index.md |
| TDWAP / staleness / triangulation maths | nx-rates/docs/aggregation-methodology.md |
| UDP mcast / WS transport | nx-rates/docs/api.md |
| Forwarder + sink architecture | nx-rates/docs/architecture.md |

BTR docs **link** to those rather than re-specify.
