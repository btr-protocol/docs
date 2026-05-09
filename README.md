# BTR Protocol — Documentation Index

Canonical docs root for the BTR protocol. Three product surfaces under one tree, served by `back/services/docs/`.

## Three Surfaces

| Surface | What it is | Entry point |
|---------|-----------|-------------|
| **BTR Swap** | Off-chain liquidity meta-aggregator. Composes LiFi, Socket, Squid, Rango, Unizen, 1inch, 0x, ParaSwap, Odos, KyberSwap, OpenOcean, Firebird. Single API, best-of-kind routing, calldata validation for keepers. | [`swap/`](./swap/01.%20Overview.md) |
| **BTR Dex (AIMM)** | BTR's own AMM. Multi-asset pools, anchor-path routing via LCA, dynamic inventory-aware fees, single-sided deposits, coverage-based IL protection. | [`dex/`](./dex/1.%20AIMM/Overview.md) |
| **BTR Supply (ALM Vaults)** | Single-asset ERC-4626/7540 vaults that allocate to external CL pools (UniV3/V4, Pancake V3/Infinity, Algebra, Ramses, Aerodrome Slipstream) and forward-looking AIMM. Worst-of(pool, oracle) marks, HWM perf-fee, sync + async redeem. | [`supply/`](./supply/01.%20Overview.md) |

## Protocol-Wide

| File | Topic |
|------|-------|
| [`Overview.md`](./Overview.md) | Top-level product surfaces + cross-cutting concepts |
| [`Manifesto.md`](./Manifesto.md) | Technical manifesto — problems, solutions, architecture, roadmap |
| [`Foundations.md`](./Foundations.md) | Design precedents: Avellaneda-Stoikov, Platypus / Wombat ALM, Curve v2 |
| [`Glossary.md`](./Glossary.md) | Glossary of protocol terms (AIMM + ALM unified) |
| [`HYBRID_SEARCH.md`](./HYBRID_SEARCH.md) | Notes on the docs-search implementation |
| [`legal/`](./legal/) | Terms of Service, Privacy Policy, Risk Disclaimer, Cookie Policy, Acknowledgement, Geographic Restriction Strategy |

## Layout

```
docs/
├── README.md            ← this file
├── Manifesto.md
├── Glossary.md
├── Foundations.md
├── Overview.md
├── HYBRID_SEARCH.md
├── legal/               ← legal markdown (served by /docs-api)
├── swap/                ← BTR Swap (off-chain aggregator)
├── dex/                 ← BTR Dex (AIMM): AIMM, Governance, Security, User & Dev guides, Contributing
├── supply/              ← BTR Supply (ALM Vaults)
└── prime/               ← BTR Prime (quant trading) — internal, not served on public docs
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

- Top-level files (`Manifesto.md`, `Glossary.md`, `Foundations.md`, `Overview.md`) are protocol-wide.
- Per-surface docs live under `swap/`, `dex/`, `supply/` — keep numbering consistent within each surface.
- After edits, re-run the build above to regenerate the search index. The watcher in dev mode also picks up changes without a restart.

## Out of Scope

- `prime/` — BTR Prime (quant trading consumer of NX Rates) docs. Internal; not currently served via the public docs endpoint.
- API source: `swap/` SDK and aggregator adapters live at `/Users/derpa/Work/btr/swap/packages/core/src/` — code only, not docs.
