# Software Architecture Best Practices

**Focus**: DeFi Protocol Architecture + System Design + Modularity

---

## Quick Reference

| Area | Standard |
|------|----------|
| **Architecture Style** | Modular monolith → microservices when needed |
| **Smart Contracts** | Upgradeable proxies, diamond pattern |
| **Frontend** | Signal-first, atomic components |
| **Backend** | Event-driven, WebSocket-first |
| **Cross-layer** | SDK as single source of truth |

---

## 1. Architecture Principles

### Core Principles

| Principle | Description | Application |
|-----------|-------------|-------------|
| **Modularity** | Independent, reusable components | Separate concerns, clear boundaries |
| **Upgradability** | Never deploy set-in-stone logic | Proxy patterns, governance control |
| **Security by design** | Defense in depth | Multi-layer security, audits |
| **Gas efficiency** | Optimize hot paths | Storage packing, libraries |
| **Determinism** | Same inputs → same outputs | Pure functions, testable logic |

### Trade-off Framework

When making architectural decisions:

| Trade-off | Consider | Bias |
|-----------|----------|------|
| **Gas vs Complexity** | Add complexity only if gas savings > 10% | Simplicity |
| **Security vs Convenience** | Never sacrifice security | Security |
| **Centralization vs Speed** | Gradual decentralization | Progressive |
| **Generality vs Specificity** | Build for use case, generalize later | Specific |

---

## 2. System Architecture

### High-Level Components

```
┌─────────────────────────────────────────────────────────────┐
│                         Users                               │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                    Frontend (Preact)                        │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │ Swap UI ││ Liquidity││ Chart   ││Governance│         │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘          │
│       │          │           │           │                 │
│       └──────────┼───────────┴───────────┘                 │
│                  │         SDK (TypeScript)                 │
│                  └───────────────┬───────────────────┐      │
└────────────────────────────────┼───────────────────│──────┘
                                   │                   │
                  ┌────────────────┴─────────┐   ┌─────┴─────┐
                  │      Backend (Bun)       │   │  Wallet   │
                  │  ┌────────────────────┐ │   │ Provider  │
                  │  │ WebSocket Server   │ │   └───────────┘
                  │  │ Price Collector    │ │
                  │  │ State Sync         │ │
                  │  └────────┬───────────┘ │
                  └───────────┼─────────────┘
                              │ RPC
                  ┌───────────▼─────────────┐
                  │    Smart Contracts       │
                  │  ┌────────────────────┐  │
                  │  │ Pool (AMM)         │  │
                  │  │ Governance         │  │
                  │  │ Oracle             │  │
                  │  │ Vault              │  │
                  │  └────────────────────┘  │
                  └─────────────────────────┘
```

### Data Flow

```
User Action → Frontend → SDK → Wallet (signature)
     ↓
Backend (optional state check)
     ↓
Smart Contract (execute)
     ↓
Backend (index events)
     ↓
Frontend (signal update)
```

---

## 3. Smart Contract Architecture

### Layered Contract Design

```
┌─────────────────────────────────────────────────────┐
│                    Proxy Layer                      │
│  • CREATE3 deterministic deployment                  │
│  • Upgradeable via governance                        │
└─────────────────────┬───────────────────────────────┘
                      │ delegates to
┌─────────────────────▼───────────────────────────────┐
│                   Implementation                     │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │ Core Module  │  │ Fee Module   │  │ Oracle    │ │
│  │ - swap()     │  │ - calcFee()  │  │ - getPrice│ │
│  │ - deposit()  │  │ - setParams()│  │           │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ LP Module    │  │ Security     │                 │
│  │ - mint()     │  │ - pause()    │                 │
│  │ - burn()     │  │ - rescue()   │                 │
│  └──────────────┘  └──────────────┘                 │
└─────────────────────────────────────────────────────┘
```

### Module Responsibilities

| Module | Responsibility | External Interface |
|--------|---------------|---------------------|
| **Core** | Swap logic, AMM formula | `swap()`, `getQuote()` |
| **LP** | Liquidity provision, shares | `deposit()`, `withdraw()` |
| **Fees** | Fee calculation, distribution | `calcFee()`, `claimFees()` |
| **Oracle** | Price feeds, TWAP | `getPrice()`, `updatePrice()` |
| **Governance** | Parameter updates, voting | `propose()`, `vote()` |
| **Security** | Emergency controls | `pause()`, `rescue()` |

### Storage Patterns

```solidity
/// @dev ERC-7201 namespaced storage
/// @custom:storage-location erc7201:btr.pool.storage
struct PoolStorage {
    // Reserves (packed for gas)
    uint96 reserve0;           // Slot 0, 0-95 bits
    uint96 reserve1;           // Slot 0, 96-191 bits
    uint32 lastUpdate;         // Slot 0, 192-223 bits
    uint8 paused;              // Slot 0, 224-231 bits

    // Mappings (separate slots)
    mapping(address => uint256) balances;
    mapping(bytes32 => Asset) assets;
}

// Get storage with inline assembly
function _getPoolStorage() internal pure returns (PoolStorage storage $) {
    assembly {
        $.slot := POOL_STORAGE_SLOT
    }
}
```

### Upgrade Patterns

| Pattern | Pros | Cons | Use Case |
|---------|------|------|----------|
| **UUPS** | Gas efficient | Implementation handles upgrade | Core contracts |
| **Transparent** | Simple proxy | Higher gas | Simple contracts |
| **Diamond (EIP-2535)** | Modular, many facets | Complex | Large protocols |

---

## 4. Frontend Architecture

### Component Hierarchy

```
App
├── Providers (Wallet, Router, Signals)
│   └── Global signal stores
├── Layout
│   ├── Header (navigation, wallet connect)
│   └── PageWrapper (content container)
└── Pages
    ├── SwapPage
    │   ├── SwapCard (form)
    │   │   ├── TokenSelect (dropdown)
    │   │   ├── AmountInput (validated)
    │   │   └── QuoteDisplay (price, fees)
    │   └── PriceChart (TradingView)
    ├── LiquidityPage
    └── GovernancePage
```

### Signal Architecture

```typescript
// Global store (singleton pattern)
export class SwapStore {
  // State
  public readonly direction = signal<OrderDirection>('sell');
  public readonly amountIn = signal<string>('');
  public readonly quote = signal<Quote | null>(null);

  // Derived (computed)
  public readonly isValid = computed(() =>
    this.amountIn.value !== '' && this.quote.value !== null
  );

  // Actions
  public setDirection = (dir: OrderDirection): void => {
    this.direction.value = dir;
  };
}

// Export singleton
export const swapStore = new SwapStore();
```

### Data Fetching Strategy

| Data Type | Source | Update Strategy |
|-----------|--------|-----------------|
| **User balances** | Wallet/Contract | On wallet connect, signal updates |
| **Pool reserves** | Contract calls | Poll every 30s + WebSocket events |
| **Price quotes** | Backend/Contract | On user input, debounce 300ms |
| **Historical data** | Backend | Lazy load, cache in IndexedDB |
| **Governance proposals** | Subgraph | Poll every 60s |

---

## 5. Backend Architecture

### Service Architecture

```
┌─────────────────────────────────────────────────────┐
│                   API Gateway                        │
│  • Rate limiting, CORS, authentication               │
└───────────────────┬─────────────────────────────────┘
                    │
┌───────────────────┴─────────────────────────────────┐
│                   Services                           │
│  ┌────────────┐  ┌────────────┐  ┌─────────────┐   │
│  │ Collector  │  │ Price Feed │  │ State Sync  │   │
│  │ (WebSocket)│  │ (Oracle)   │  │ (Indexer)   │   │
│  └─────┬──────┘  └─────┬──────┘  └─────┬───────┘   │
└────────┼───────────────┼───────────────┼───────────┘
         │               │               │
┌────────▼───────────────▼───────────────▼───────────┐
│                   Data Layer                        │
│  ┌────────────┐  ┌────────────┐  ┌─────────────┐  │
│  │ SQLite     │  │ Redis      │  │ Postgres    │  │
│  │ (hot data) │  │ (cache)    │  │ (analytics) │  │
│  └────────────┘  └────────────┘  └─────────────┘  │
└────────────────────────────────────────────────────┘
```

### Service Responsibilities

| Service | Responsibility | Tech |
|---------|---------------|------|
| **Collector** | Aggregate data from chains | Bun + WebSocket |
| **Price Feed** | Oracle price aggregation | Zig + external APIs |
| **State Sync** | Index contract events | Bun + SQLite |
| **Notifier** | Push notifications to clients | WebSocket |

### Event Processing

```typescript
// Event-driven architecture
class EventProcessor {
  async processBlock(block: Block): Promise<void> {
    for (const log of block.logs) {
      switch (log.topics[0]) {
        case SWAP_EVENT_SIGNATURE:
          await this.handleSwap(log);
          break;
        case DEPOSIT_EVENT_SIGNATURE:
          await this.handleDeposit(log);
          break;
      }
    }
    await this.broadcastUpdates();
  }
}
```

---

## 6. SDK Architecture

### SDK as Source of Truth

All metadata must come from SDK:

| Data | SDK Module | Frontend Usage |
|------|-----------|----------------|
| **Token addresses** | `sdk/src/eth/tokens.ts` | Import, never duplicate |
| **Chain configs** | `sdk/src/eth/chains.ts` | Network selection |
| **Contract ABIs** | `sdk/src/eth/contracts.ts` | Contract interaction |
| **Price functions** | `sdk/src/pricing/` | Quote calculation |

### SDK Structure

```
sdk/
├── src/
│   ├── eth/              # Blockchain metadata
│   │   ├── tokens.ts      # Token addresses, decimals, symbols
│   │   ├── chains.ts      # Chain IDs, RPC endpoints
│   │   └── contracts.ts   # Contract addresses, ABIs
│   ├── pricing/           # Price calculations
│   │   ├── amm.ts         # AMM formulas
│   │   └── router.ts      # Path finding
│   ├── pool/              # Pool interface
│   │   ├── swap.ts        # Swap functions
│   │   └── liquidity.ts   # LP functions
│   ├── governance/        # Governance interface
│   └── types/             # TypeScript types
└── package.json
```

---

## 7. Cross-Chain Architecture

### Multi-Chain Strategy

```
┌─────────────────────────────────────────────────────┐
│                   Layer: User Interface             │
│              (Chain-agnostic frontend)               │
└───────────────────┬─────────────────────────────────┘
                    │
┌───────────────────┴─────────────────────────────────┐
│                   Layer: Abstraction                 │
│         SDK with chain selection logic               │
└───────────────────┬─────────────────────────────────┘
                    │
      ┌─────────────┼─────────────┬─────────────┐
      │             │             │             │
┌─────▼─────┐ ┌────▼──────┐ ┌───▼──────┐ ┌───▼──────┐
│ Ethereum  │ │ BSC      │ │ Polygon  │ │ Arbitrum │
│           │ │          │ │          │ │          │
└───────────┘ └──────────┘ └──────────┘ └──────────┘
```

### Cross-Chain Considerations

| Area | Strategy |
|------|----------|
| **Deployment** | CREATE3 for same address on all chains |
| **Bridging** | Integration with LayerZero/Wormhole |
| **Oracles** | Chainlink on all chains |
| **State** | Per-chain pools, aggregated frontend view |

---

## 8. Security Architecture

### Defense in Depth

```
┌─────────────────────────────────────────────────────┐
│                   Layer 1: Contract                  │
│  • Audited code                                       │
│  • Access control (timelock, multi-sig)             │
│  • Reentrancy guards                                 │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│                   Layer 2: Protocol                  │
│  • Flow guards (cooldowns)                           │
│  • Oracle checks (TWAP, deviation)                  │
│  • Price limits                                     │
└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐
│                   Layer 3: Operations                 │
│  • Monitoring/alerts                                │
│  • Bug bounty                                       │
│  • Emergency pause                                  │
└─────────────────────────────────────────────────────┘
```

### Access Control Hierarchy

| Role | Permissions | Approval |
|------|-------------|----------|
| **Admin** | All functions | Multi-sig |
| **Guardian** | Emergency pause | Single |
| **Arbitrageur** | Whitelisted functions | Governance |
| **User** | Standard functions | N/A |

---

## 9. Performance Optimization

### Frontend Optimization

| Area | Technique | Impact |
|------|-----------|--------|
| **Bundle size** | Code splitting, tree shaking | <500KB gzipped |
| **Rendering** | Signal-first, avoid re-renders | 60fps UI |
| **Data loading** | Lazy load, infinite scroll | Faster initial load |
| **Caching** | Service worker, IndexedDB | Offline capable |

### Contract Optimization

| Area | Technique | Savings |
|------|-----------|---------|
| **Storage** | Pack structs, use memory | ~20k gas/SSTORE |
| **Loops** | Cache length, batch operations | ~5k gas/iteration |
| **Events** | Emit only necessary | ~375 gas/event |
| **Libraries** | Use OpenZeppelin | Audited, optimized |

---

## 10. Architecture Review Process

### Review Checklist

- [ ] Module boundaries are clear
- [ ] Dependencies are minimal
- [ ] Upgrade path is defined
- [ ] Security implications documented
- [ ] Gas costs estimated
- [ ] Test coverage planned
- [ ] Documentation updated

### Approval Process

| Change Type | Reviewers | Approval |
|-------------|-----------|----------|
| **Core module** | 2 architects + security | 5/5 consensus |
| **New feature** | Lead architect + security | 3/5 consensus |
| **Bug fix** | Lead architect | Single approval |
| **Optimization** | Lead + gas auditor | 2/5 consensus |

---

## Internal References

- [`SMART_CONTRACTS.md`](./SMART_CONTRACTS.md) - Contract architecture
- [`FRONTEND.md`](./FRONTEND.md) - Frontend patterns
- [`BACKEND.md`](./BACKEND.md) - Backend patterns
- [`SECURITY.md`](./SECURITY.md) - Security architecture

---

*Last updated: 2025-01*
