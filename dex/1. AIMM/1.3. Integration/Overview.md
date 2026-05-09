# Integration Overview

> AMM-specific integration patterns for pools, swaps, and liquidity

---

## 1. Overview

This section covers **AMM-specific integration** with BTR Protocol pools. For protocol-wide features (treasury, bridging, white-labelling), see [Section 5: Developer Guides](../../../5.\ Developer\ Guides/).

### 1.1. Scope

| This Section (1.3) | Section 5 (Developer Guides) |
|--------------------|------------------------------|
| Swaps and quotes | Treasury fee collection |
| Liquidity provision | Cross-chain bridging |
| Pool deployment | White-labelling |
| Hooks system | |
| Staking mechanics | |

---

## 2. Integration Guides

| Guide | Description |
|-------|-------------|
| [Composability](./1.3.1.\ Composability.md) | Standard interfaces for swaps, liquidity, flash loans |
| [Hooks](./1.3.2.\ Hooks.md) | Per-asset customization via hook contracts |
| [Pool Deployment](./1.3.3.\ Curation\ &\ Pool\ Deployment.md) | Deploying and configuring custom pools |

---

## 3. Quick Start

### 3.1. TypeScript SDK

```typescript
import { Pool } from '@btr-protocol/sdk';

const pool = new Pool(poolAddress);

// Get swap quote
const quote = await pool.getSwapQuote(tokenIn, tokenOut, amount);

// Execute swap
const tx = await pool.swap(tokenIn, tokenOut, amount, minOut);
```

### 3.2. Direct Solidity

```solidity
import { IPoolV1 } from '@btr/interfaces';

IPoolV1 pool = IPoolV1(poolAddress);

// Get quote
SwapQuote memory quote = pool.getSwapQuote(tokenIn, tokenOut, amountIn);

// Execute swap
uint256 amountOut = pool.swap(tokenIn, tokenOut, amountIn, minOut, recipient);
```

---

## 4. Core Interfaces

| Interface | Purpose |
|-----------|---------|
| `ICoreV1` | Swaps, deposits, withdrawals |
| `IStakingV1` | LP and gov token staking |
| `IFlashV1` | ERC-3156 flash loans |
| `IPoolHooks` | Per-asset hooks |

---

## 5. Related Documentation

- [Architecture Overview](../../Overview.md) — Pool design and modules
- [Spread & Fees](../1.1.\ Pricing/1.1.4.\ Spread\ &\ Fees.md) — Fee calculation
- [Parametrization](../1.1.\ Pricing/1.1.7.\ Parametrization.md) — Parameter reference
