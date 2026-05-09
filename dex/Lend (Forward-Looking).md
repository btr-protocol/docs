# Lend (Forward-Looking)

> Design notes for the future `modules/Lend.sol` impl. Stub lives at `dex/evm/src/modules/Lend.sol`. Phase 43.

## Design philosophy

Internal money market where:

1. Each AIMM asset has its own isolated lending market.
2. Reserves (idle liquidity) are borrowable.
3. Liabilities (LP positions) serve as collateral (valued at `liabilities × coverage ratio`).
4. Enables leverage trading WITHOUT perpetual contracts.

**NOT for reserves hypothecation** — that's handled by hooks (see `AaveYieldHook`, `MorphoYieldHook`).

## Advantage: LP-collateralized vs perpetuals

**Traditional perpetuals (GMX, Hyperliquid, dYdX):**
- Zero-sum game: trader profit = LP loss.
- Historical losses: GMX -$12M, Hyperliquid -$4M + $130M withdrawals.
- Adversarial: traders bet AGAINST LPs.

**LP-collateralized (this module):**
- Positive-sum: both earn from real trading fees.
- Example: 10x leverage, 8% LP APY, 5% borrow → +35% APR (vs -47% for Contango).
- Zero slippage: no DEX swaps needed.
- Aligned incentives: leveraged traders ARE LPs, want pool health.
- Organic TVL growth: each leveraged position increases reserves.

## Leverage mechanism: flash loan loop

User opens 10x position with $100K:

```
1. Flash loan $900K from AIMM Flash module
2. Deposit $1M total → receive LP tokens (liabilities)
3. Supply LP tokens to THIS module as collateral
4. Borrow $900K from THIS module
5. Repay flash loan

Result:
- User owns: $1M LP position (10x leverage)
- User owes: $900K debt to Lend module
- Collateral: LP tokens worth $1M × coverage ratio
- Cost: 0% slippage, earns LP fees while leveraged
```

## Collateral valuation (coverage-adjusted)

```solidity
// LP collateral value accounts for coverage ratio
lpValue = lpAmount × underlyingPrice × coverageRatio / PRECISION;

// Example: coverage drops to 90%
collateralValue = $1M LP × 0.9 = $900K;
debtValue = $900K;
healthFactor = $900K / $900K = 1.0; // liquidation threshold
```

**Key insight**: coverage ratio drop = collateral haircut → leveraged traders incentivized to maintain pool health.

## Accounting neutrality

Opening leveraged position doesn't hurt coverage:

```
Before:
  Reserves: $5M, Liabilities: $5M, Coverage: 100%

After user opens 10x on $100K:
  Reserves: $5.9M (+$900K borrowed deposited)
  Liabilities: $5.9M (+$900K LP claim)
  Coverage: 100% (unchanged!)
```

## Isolated markets (per-asset)

Each asset operates independently:

- USDC market: borrow USDC against USDC LP collateral.
- WETH market: borrow WETH against WETH LP collateral.
- No cross-asset initially (can add Phase 2).

Benefits: simpler risk management, no contagion between assets, clear liquidation logic.

## Risk parameters

Per-asset configuration:

```solidity
struct LendConfig {
    uint16 maxLTV;              // Max loan-to-value (e.g., 8000 = 80%)
    uint16 liquidationThreshold; // Liquidation LTV (e.g., 8600 = 86%)
    uint16 liquidationBonus;     // Liquidator incentive (e.g., 500 = 5%)
    uint32 borrowRate;           // Annual borrow rate (e.g., 5% APR)
    uint128 maxBorrowPerUser;    // Position size cap
}
```

Conservative initial values:

```
USDC (low volatility):
  maxLTV: 80%, liquidationThreshold: 86%, maxLeverage: 5x

WETH (medium volatility):
  maxLTV: 75%, liquidationThreshold: 81%, maxLeverage: 4x
```

## Liquidation mechanics

Partial liquidations minimize user losses:

```
healthFactor = collateralValue × liquidationThreshold / debtValue;

If healthFactor < 1.0:
  1. Calculate shortfall.
  2. Liquidate minimum required (50% position).
  3. Liquidator receives 5-10% bonus (from collateral).
  4. Remaining position restored to healthy ratio.
```

## Interest rate model

Simple fixed rate initially, can upgrade to utilization-based:

```
// Phase 1: fixed rate
borrowAPR = 5% (constant);

// Phase 2: utilization-based (like Aave)
utilization = totalBorrowed / totalReserves;
borrowAPR = baseRate + (utilization × slope);

Example:
  0% util → 2% APR
  50% util → 5% APR
  80% util → 10% APR (kink)
  90% util → 20% APR (steep slope)
```

## Return profile example

User with 10x leverage on USDC:

```
Capital: $100K
LP position: $1M (10x)
Debt: $900K

Revenue:
  LP fees: $1M × 8% APY = $80K/year

Costs:
  Borrow interest: $900K × 5% = $45K/year

Net profit: $35K on $100K capital = 35% APR
(Profitable even if USDC price flat!)
```

Compare to Contango (spot leverage):

```
Same 10x position with 0% price movement:
  Revenue: $0 (no LP fees)
  Costs: $45K borrow + $2K slippage = $47K
  Net: -47% (requires 7%+ price appreciation to break even)
```

## Integration with other modules

- **Flash**: provides flash loans for leverage loop.
- **Core**: handles deposits/withdrawals (LP token creation).
- **Admin**: configures risk parameters per asset.
- **Hooks**: NOT used for hypothecation (that's `AaveYieldHook`/`MorphoYieldHook`).

## Reserves hypothecation (separate concern)

Reserves hypothecation to external protocols (Aave, Morpho) is done via hooks:

- `AaveYieldHook.sol`: deploy reserves to Aave V3 → earn interest.
- `MorphoYieldHook.sol`: deploy reserves to Morpho vaults → earn yield.
- `EulerYieldHook.sol`: deploy reserves to Euler V2 → earn interest.

**Why separate?** Hooks are modular, composable, per-asset configurable. Don't need to build our own lending protocol for yield. Focus `Lend` module on leverage trading UX.

## Implementation phases

**Phase 1 — basic lending:**

```solidity
function borrow(address token, uint256 amount) external returns (uint256 debtId);
function repay(address token, uint256 amount) external;
function supplyCollateral(address token, uint256 lpAmount) external;
function withdrawCollateral(address token, uint256 lpAmount) external;
```

**Phase 2 — leverage helpers:**

```solidity
function openLeveragedPosition(address token, uint256 capital, uint256 targetLeverage) external;
function closeLeveragedPosition(uint256 positionId) external;
function adjustLeverage(uint256 positionId, uint256 newLeverage) external;
```

**Phase 3 — liquidations:**

```solidity
function liquidate(address borrower, address token, uint256 amount) external;
function getHealthFactor(address user, address token) external view returns (uint256);
function getLiquidationPrice(address user, address token) external view returns (uint256);
```

**Phase 4 — advanced features:**

- Cross-asset margining (borrow USDC against WETH LP).
- Utilization-based interest rates.
- Isolated lending pools with custom risk params.
- Position transfer/delegation.

## Storage additions

To be added to `IPool.PoolStorage`:

```solidity
mapping(address token => LendConfig) lendConfigs;
mapping(address user => mapping(address token => Collateral)) collateral;
mapping(address user => mapping(address token => Debt)) debts;
mapping(address token => uint256) totalBorrowed;
mapping(address token => uint256) debtIndex; // interest accrual
```

## Security considerations

1. **Oracle manipulation**: use TWAP for LP valuation, verify coverage ratio.
2. **Liquidation cascades**: cap per-user position size (% of total reserves).
3. **Flash loan attacks**: `ReentrancyGuard`, verify coverage before/after.
4. **Coverage ratio gaming**: liquidation threshold > coverage drop speed.
5. **Interest accrual bugs**: test debt index updates thoroughly.

## Monitoring & risk management

Key metrics:

```solidity
utilizationRate = totalBorrowed / totalReserves;
avgHealthFactor = average across all positions;
liquidationRisk = % of positions near liquidation;
collateralConcentration = largest position / total collateral;
```

Alerts:

- Utilization > 90%: reduce `maxLTV`, increase borrow rate.
- `avgHealthFactor` < 1.2: increase liquidation bonus.
- Coverage ratio drops: auto-liquidate underwater positions.

## References

- Specs: `old/specs/LEVERAGE_TRADING.md` (LP-collateralized design).
- Specs: `old/specs/RESERVES_HYPOTHECATION.md` (for hooks, not this module).
- Comparison: GMX (perpetuals), Contango (spot leverage), Aave (lending).
- Standards: ERC-3156 (flash loans for leverage loop).
