---
title: "Developer Guides"
description: "Protocol-level integration guides for external developers"
audience: tech
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# Developer Guides

> Protocol-level integration guides for external developers

---

## 1. Overview

This section covers **protocol-wide integration** with the BTR Protocol. For AMM-specific features (swaps, liquidity, hooks, pool deployment), see [Section 1.3: Integration](../../1.\ AIMM/1.3.\ Integration/).

### 1.1. Scope Separation

| This Section (5) | Section 1.3 (Integration) |
|------------------|---------------------------|
| Treasury fee collection | Swaps and quotes |
| Cross-chain bridging | Liquidity provision |
| White-labelling protocol | Pool deployment |
| | Hooks system |
| | Staking mechanics |

---

## 2. Available Guides

| Guide | Description |
|-------|-------------|
| [Treasury Integration](./5.1.\ Treasury\ Integration.md) | Protocol fee collection and treasury operations |
| [Bridging](./5.2.\ Bridging.md) | Cross-chain token transfers via Bridge |
| [White Labelling](./5.3.\ White\ Labelling.md) | Deploying branded pool instances |

---

## 3. Integration Methods

### 3.1. TypeScript SDK

```typescript
import { Pool, Treasury, Bridge } from '@btr-protocol/sdk';
```

**Recommended for**:
- Frontend applications
- Backend services
- Quick prototyping

### 3.2. Direct Solidity

```solidity
import { IPool, ITreasury, IBridge } from '@btr/interfaces';
```

**Recommended for**:
- Smart contract integrations
- Gas-optimized on-chain logic

---

## 4. Related Documentation

- [Integration](../../1.\ AIMM/1.3.\ Integration/) -AMM-specific integration patterns
- [Security Overview](../../3.\ Security/Overview.md) -Audit considerations
- [Contributing](../../6.\ Contributing/) -Internal development practices
