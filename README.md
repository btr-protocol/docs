---
title: "BTR Protocol -Documentation Index"
description: "Canonical docs root for the BTR protocol. Three product surfaces under one tree, served by back/services/docs/."
audience: both
type: explanation
status: live
phase: n/a
order: 99
lang: en
publish: true
---
# BTR Protocol -Documentation Index

Canonical docs root for the BTR protocol. Three product surfaces under one tree, served by `back/services/docs/`.

## Three Surfaces

| Surface | What it is | Entry point |
|---------|-----------|-------------|
| **BTR Swap** | Off-chain liquidity meta-aggregator. Composes LiFi, Socket, Squid, Rango, Unizen, 1inch, 0x, ParaSwap, Odos, KyberSwap, OpenOcean, Firebird. Single API, best-of-kind routing, calldata validation for keepers. | [`swap/`](./swap/01.%20Overview.md) |
| **BTR Dex (AIMM)** | BTR's own AMM. Multi-asset pools, anchor-path routing via LCA, dynamic inventory-aware fees, single-sided deposits, coverage-based IL protection. | [`dex/`](./dex/1.%20AIMM/Overview.md) |
| **BTR Supply (ALM Vaults)** | Single-asset ERC-4626/7540 vaults that allocate to external CL pools (UniV3/V4, Pancake V3/Infinity, Algebra, Ramses) and forward-looking AIMM. Worst-of(pool, oracle) marks, HWM perf-fee, sync + async redeem. | [`vaults/`](./vaults/01.%20Overview.md) |

## Protocol-Wide

| File | Topic |
|------|-------|
| [`concepts/overview.md`](./concepts/overview.md) | Top-level product surfaces + cross-cutting concepts |
| [`concepts/glossary.md`](./concepts/glossary.md) | Glossary of protocol terms (ALM dual meaning + AIMM unified) |
| [`legal/acknowledgement.md`](./legal/acknowledgement.md) | Terms of Service, Privacy Policy, Risk Disclaimer, Cookie Policy, Acknowledgement |

Note: `concepts/manifesto.md` and `concepts/foundations.md` describe BTR DEX architecture and design precedents.

## Layout

```
docs/
├── README.md            ← this file
├── concepts/            ← protocol-wide concepts: manifesto, overview, glossary, foundations, capital-efficiency, amm-landscape
├── getting-started/     ← user onboarding
├── tutorials/           ← step-by-step user tutorials
├── guides/              ← user how-to guides (FAQ, troubleshooting, withdraw, claim)
├── reference/           ← reference: contract addresses, SDK, API, glossary pointer
├── security/            ← security model: audits, bug bounty, pause, roles & timelock
├── trade/               ← BTR Swap user docs (overview, slippage, routes)
├── liquidity/           ← Phase-2 LP docs (coming soon)
├── legal/               ← legal markdown (served by /docs-api)
├── swap/                ← BTR Swap (off-chain aggregator)
├── dex/                 ← BTR DEX (AIMM): AIMM, Governance, Security, User & Dev guides, Contributing
├── vaults/              ← BTR Supply (ALM Vaults) — renamed from supply/
├── protocol/            ← protocol mechanics: AccessControl, Treasury, Staking, Distributor, Bridge, Governance, BTR Token, Emissions
└── prime/               ← BTR Prime (quant trading) - internal, not served on public docs
```

## Build & Serve

The docs back-service builds a search index + pre-compiled HTML corpus from this tree:

```bash
cd /Users/derpa/Work/btr/back
bun install
bun run --filter @btr-protocol/docs-service build
bun run --filter @btr-protocol/docs-service dev
```

The front-end fetches `/docs-api/{structure,all,search-index}` (proxied to `localhost:3003/docs/...`) and renders them in `front/src/components/features/docs/`.

## Editing

- Protocol-wide files (`concepts/overview.md`, `concepts/glossary.md`, `concepts/manifesto.md`, etc.) live under `concepts/`.
- Per-surface docs live under `swap/`, `dex/`, `vaults/` - keep numbering consistent within each surface.
- After edits, re-run the build above to regenerate the search index. The watcher in dev mode also picks up changes without a restart.

## Out of Scope

- `prime/` -BTR Prime (quant trading consumer of NX Rates) docs. Internal; not currently served via the public docs endpoint.
- API source: `swap/` SDK and aggregator adapters live at `/Users/derpa/Work/btr/swap/packages/core/src/` -code only, not docs.
