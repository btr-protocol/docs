---
title: "Security & Audit Best Practices"
description: "Note: BTR currently deploys on EVM chains only. SVM (Solana) and MoveVM (Aptos/Sui) sections are included for future cross-chain expansion reference."
audience: tech
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# Security & Audit Best Practices

**Focus**: BTR AIMM Protocol -EVM (Solidity) smart contract security

> **Note**: BTR currently deploys on EVM chains only. SVM (Solana) and MoveVM (Aptos/Sui) sections are included for future cross-chain expansion reference.

> **Architecture note (post Phase 42H)** -examples below that show `PoolProxy`, Diamond storage namespacing, `delegatecall`-based facets, or ERC-7201 helpers are **generic Solidity security pattern references**. The BTR DEX repo no longer uses any of these -it uses standalone singletons + EIP-1167 `Pool` clones with default storage. Treat the patterns here as "what to do **if** you ever encounter that pattern" rather than as a description of current BTR code.

---

## Table of Contents

1. [Scope & Threat Model](#1-scope--threat-model)
2. [Universal Security Rules](#2-universal-security-rules)
3. [EVM Security Controls](#3-evm-security-controls)
4. [SVM Security (Future)](#4-svm-security-solana-future)
5. [MoveVM Security (Future)](#5-movevm-security-aptossui-future)
6. [Cross-Chain Security](#6-cross-chain-security)
7. [Security Checklist](#7-security-checklist)
8. [Audit Process](#8-audit-process)
9. [Severity Classification](#9-severity-classification)
10. [Incident Response](#10-incident-response)

---

## 1. Scope & Threat Model

### 1.1 Protected Assets

| Asset | Description | Trust Boundary |
|-------|-------------|----------------|
| **User Funds** | LP deposits, swap amounts, staked tokens | External calls, oracle reads |
| **Protocol Fees** | Protocol share of swap fees, flash fees | Admin functions, fee collection |
| **Oracle Integrity** | Internal TWAP, external oracle fallback | External calls, stale data |
| **Upgrade Authority** | Module updates, proxy upgrades | Timelock, multisig |
| **Cross-Chain Mint/Burn** | Bridge LayerZero operations | Peer verification, LZ endpoint |
| **Base Token** | Pool's reserve currency (e.g., USDC) | Migration timelock (7 days) |

### 1.2 Adversary Model

| Adversary | Capabilities | Mitigations |
|-----------|--------------|-------------|
| **External User** | Anyone can call public functions | Access control, input validation |
| **MEV Searcher/Builder** | Can reorder/insert transactions | Deadlines, slippage, commit-reveal |
| **Malicious Token** | Custom transfer logic, reentrancy | SafeERC20, balance checks |
| **Compromised Relayer** | Can send arbitrary LayerZero messages | Peer verification, failed message queue |
| **Compromised Admin** | Single-owner key currently | Timelock, multisig recommendation |
| **Oracle Manipulator** | Flash loan price manipulation | TWAP smoothing, deviation checks |

### 1.3 Trust Boundaries

1. **External Calls (EVM)**
   - Token transfers (`IERC20.transfer`, `transferFrom`)
   - Oracle reads (`IOracle.getFeed`, `isFeedFresh`)
   - Hook calls (`IPoolHooks` on swap/stake)
   - LayerZero endpoint messaging

2. **Cross-Chain Messages**
   - Peer contract addresses on remote chains
   - LayerZero endpoint authenticity
   - Failed message recovery queue

3. **Upgrade Surface**
   - Module selector mapping updates
   - UUPS upgrade authorization
   - Timelock duration configuration

4. **Oracle Trust**
   - Primary oracle → Secondary oracle fallback
   - **Warning**: Fallback bypasses deviation checks (see Base:M-03)

---

## 2. Universal Security Rules

### 2.1 External Interaction Policy

**Treat every external interaction as adversarial:**

```solidity
// ❌ VULNERABLE: Trusting external call return value
function badExternalCall(address target) external {
    (bool success, bytes memory data) = target.call("");
    require(success, "Call failed");
    // Assuming data is valid without validation
}

// ✅ SECURE: Validate all external inputs
function goodExternalCall(address target) external {
    (bool success, bytes memory data) = target.call("");
    if (!success) revert ExternalCallFailed();
    if (data.length != 32) revert InvalidReturnValue();
    // Validate decoded data before use
}
```

### 2.2 Invariants Over Heuristics

**Prefer explicit invariant checks over "anti-flashloan" patterns:**

```solidity
// ⚠️ WEAK: Cooldown as primary defense
// Cooldowns are UX constraints, not security boundaries
function withdraw(uint256 amount) external {
    require(block.timestamp >= lastDeposit[msg.sender] + 15 seconds, "Cooldown");
    // No invariant check - can still be manipulated via other paths
}

// ✅ STRONG: Explicit invariant enforcement
function withdraw(uint256 amount) external {
    uint256 newLiabilities = liabilities - amount;
    uint256 newReserves = reserves - amount;
    // Invariant: reserves >= liabilities after withdrawal
    require(newReserves >= newLiabilities, "Insolvency risk");
    // Cooldown is UX-only, not the primary defense
    require(block.timestamp >= lastDeposit[msg.sender] + 15 seconds, "Cooldown");
    liabilities = newLiabilities;
    reserves = newReserves;
}
```

### 2.3 Privileged Surface Hardening

**Separate roles and enforce timelocks:**

| Role | Current BTR Pattern | Recommended Enhancement |
|------|---------------------|--------------------------|
| **Owner** | Single address | Multisig (3/5 or 5/7) |
| **Pause/Guardian** | Owner (via freezeAsset) | Separate timelocked role |
| **Fee Setter** | Owner (timelocked 1 day) | Treasury multisig |
| **Upgrade Authority** | Owner (timelocked 7 days) | DAO governance |

### 2.4 Ordering Assumptions

**Same-block ordering is NOT safe under MEV:**

```solidity
// ❌ VULNERABLE: Assumes sequential execution
function batchSwap(bytes[] calldata data) external {
    for (uint i = 0; i < data.length; i++) {
        // MEV searcher can sandwich the entire batch
        executeSwap(data[i]);
    }
}

// ✅ SECURE: Intent/commit-reveal for sensitive operations
function commitSwap(bytes32 commitment) external {
    commitments[msg.sender] = commitment;
    commitTime[msg.sender] = block.timestamp;
}

function revealSwap(bytes32 salt, SwapParams calldata params) external {
    bytes32 commitment = keccak256(abi.encode(salt, params, msg.sender));
    require(commitments[msg.sender] == commitment, "Invalid commitment");
    require(block.timestamp >= commitTime[msg.sender] + 1 minutes, "Too early");
    // Now execute with reduced MEV risk
}
```

---

## 3. EVM Security Controls

### 3.1 Reentrancy & Composability

#### 3.1.1 Cross-Function Reentrancy

```solidity
// ❌ VULNERABLE: Reentrancy guard only on one function
function deposit() external nonReentrant {
    balances[msg.sender] += msg.value;
}

function withdraw(uint256 amount) external {
    // No guard! Reentrancy via deposit's external call can enter here
    payable(msg.sender).transfer(amount);
    balances[msg.sender] -= amount;
}

// ✅ SECURE: Guard all state-changing functions
function deposit() external nonReentrant {
    balances[msg.sender] += msg.value;
}

function withdraw(uint256 amount) external nonReentrant {
    balances[msg.sender] -= amount;
    payable(msg.sender).transfer(amount);
}
```

#### 3.1.2 Read-Only Reentrancy

```solidity
// ❌ VULNERABLE: View function relied upon for state
function getBalance(address user) external view returns (uint256) {
    return balances[user];
}

function expensiveOperation() external {
    // Attacker reenters and changes balance between check and use
    uint256 bal = getBalance(msg.sender);
    _doSomething(bal);
}

// ✅ SECURE: Re-check state after external calls
function expensiveOperation() external nonReentrant {
    uint256 bal = balances[msg.sender];
    _doSomething(bal);
    // Re-validate after any external call in _doSomething
    require(balances[msg.sender] >= bal, "Balance changed");
}
```

#### 3.1.3 Transient Storage Guards (EIP-1153)

**Network Requirement**: Transient storage (`tload`/`tstore`) is **ONLY available on EIP-1153 networks**:
- Ethereum mainnet: Cancun upgrade (Dencun) ✓
- Arbitrum: Arbitrum One ✓
- Optimism: Ecotone ✓
- Base: Ecotone ✓
- Polygon: Not yet (as of 2025-01)
- BSC: Not yet (as of 2025-01)

**Policy for BTR:**

```text
Allowed: Custom transient guard (Base:23-45) on EIP-1153 chains ONLY
Fallback: Use Solady ReentrancyGuard (storage-based) on non-EIP-1153 chains
Default: Prefer OpenZeppelin ReentrancyGuardTransient for new code
Never: Bespoke assembly guards without explicit network gating
```

**Current Implementation (Base.sol:23-45):**

```solidity
// ✅ BTR Pattern: Transient storage guard
bytes32 private constant REENTRANCY_GUARD_SLOT =
    0xe22c27e8d25bc3725093027126bd674994df6625365bae10cf4b95c8b45f98b6;

modifier nonReentrant() {
    assembly {
        if tload(REENTRANCY_GUARD_SLOT) {
            mstore(0x00, 0x92f0d5b4)  // ReentrancyDetected() selector
            revert(0x00, 0x04)
        }
        tstore(REENTRANCY_GUARD_SLOT, 1)
    }
    _;
    assembly {
        tstore(REENTRANCY_GUARD_SLOT, 0)
    }
}
```

**For networks without EIP-1153, use storage-based guard:**

```solidity
import {ReentrancyGuard} from "solady/utils/ReentrancyGuard.sol";

contract Bridge is ReentrancyGuard {
    // Uses storage slot 0x... for guard state
    // Compatible with all EVM networks
}
```

### 3.2 Arithmetic & DeFi Math

#### 3.2.1 Checked Arithmetic

```solidity
// Solidity >= 0.8 has built-in overflow checks
// Only use unchecked with proven bounds

// ✅ SECURE: Unchecked with documented bounds
function sum(uint256[] calldata values) external pure returns (uint256) {
    uint256 total;
    unchecked {
        // Safe because we cap array length at 100
        for (uint256 i = 0; i < values.length; ++i) {
            total += values[i];
            // Reasoning: Even if all values are type(uint256).max,
            // the loop would run out of gas before overflowing
            // due to the 100-element cap enforced in caller
        }
    }
    return total;
}
```

#### 3.2.2 High-Precision Math (mulDiv)

```solidity
// ❌ VULNERABLE: Precision loss
uint256 result = (amount * fee) / 100;  // Loses precision for large amounts

// ✅ SECURE: Use mulDiv with explicit rounding
import {Math} from "openzeppelin/utils/math/Math.sol";

uint256 result = Math.mulDiv(amount, fee, 100, Math.Rounding.Floor);
// Or for ceiling: Math.Rounding.Ceil
```

#### 3.2.3 ERC-4626 Inflation Attacks

```solidity
// ❌ VULNERABLE: First depositor gets inflated shares
function deposit(uint256 assets, address receiver) external returns (uint256) {
    uint256 totalAssets = _totalAssets();
    uint256 totalSupply = totalSupply();

    if (totalSupply == 0) {
        // First depositor gets minimal assets == maximal shares
        return _mint(receiver, assets);  // WRONG!
    }

    uint256 shares = (assets * totalSupply) / totalAssets;
    _mint(receiver, shares);
    return shares;
}

// ✅ SECURE: Virtual offset to prevent inflation attack
uint256 private constant VIRTUAL_SHARES = 1e18;
uint256 private constant VIRTUAL_ASSETS = 1e6;  // Small initial amount

function _totalAssets() internal view returns (uint256) {
    return asset.balanceOf(this) + VIRTUAL_ASSETS;
}

function totalSupply() public view override returns (uint256) {
    return super.totalSupply() + VIRTUAL_SHARES;
}
```

### 3.3 Access Control, Upgrades, Initialization

#### 3.3.1 Admin & Upgrade Authority

**Current BTR Pattern (Admin.sol):**

```solidity
// ✅ Multi-tier timelock (LibTimelock)
uint48 internal constant CRITICAL_TIMELOCK = 7 days;   // Base token migration
uint16 private constant HIGH_TIMELOCK = 3 days;        // Ownership transfer, bridge update
uint16 private constant MEDIUM_TIMELOCK = 2 days;      // Staking/distribution changes
uint16 private constant BASE_TIMELOCK = 1 day;         // Oracle changes
uint16 private constant LOW_TIMELOCK = 1 day;          // Adding assets

// ✅ Request-execute pattern for all sensitive operations
function requestOwnershipTransfer(address newOwner) external onlyOwner {
    pendingOps[TIMELOCK_ID_OWNERSHIP] = TL.pack(HIGH_TIMELOCK, GRACE_PERIOD);
    pendingData[TIMELOCK_ID_OWNERSHIP] = abi.encode(newOwner);
    emit TimelockRequested(TIMELOCK_ID_OWNERSHIP, ..., block.timestamp + HIGH_TIMELOCK);
}

function executeOwnershipTransfer() external nonReentrant onlyOwner {
    TL.validate(pendingOps[TIMELOCK_ID_OWNERSHIP]);
    address newOwner = abi.decode(pendingData[TIMELOCK_ID_OWNERSHIP], (address));
    owner = newOwner;
    delete pendingOps[TIMELOCK_ID_OWNERSHIP];
    delete pendingData[TIMELOCK_ID_OWNERSHIP];
}
```

**Recommendation: Transition to Multisig**

```solidity
// ✅ SECURE: Multisig for owner role
import {Ownable} from "solady/auth/Ownable.sol";

contract PoolProxy is Ownable {
    // Owner should be a multisig (e.g., 3/5 or 5/7)
    // Recommended: Safe (Gnosis Safe) multisig wallet
}
```

#### 3.3.2 Upgradeable Initialization Risks

```solidity
// ❌ VULNERABILITY CLASS 1: Uninitialized proxy
// If an attacker calls initialize on an uninitialized proxy,
// they can become the owner and drain funds.

// ✅ SECURE: Constructor sets immutable owner
contract PoolProxy {
    constructor(address _owner) {
        owner = _owner;
        initialized = true;  // Prevent front-running initialization
    }
}

// ❌ VULNERABILITY CLASS 2: Double-initialization
function initialize(address newOwner) external {
    // No check! Can re-initialize and reset state
    owner = newOwner;
}

// ✅ SECURE: One-time initialization
function initialize(address newOwner) external {
    require(!initialized, "Already initialized");
    initialized = true;
    owner = newOwner;
}

// ❌ VULNERABILITY CLASS 3: Unsafe upgrade executor
// If upgrade implementation reverts, storage may be corrupted

// ✅ SECURE: Validate implementation before upgrade
function executeUpgrade() external onlyOwner {
    TL.validate(pendingOps[pendingUpgrade]);
    address newImpl = pendingImplementation;

    // Validate implementation exists and has code
    uint256 size;
    assembly { size := extcodesize(newImpl) }
    require(size > 0, "Invalid implementation");

    delete pendingUpgrade;
    delete pendingImplementation;
    _authorizeUpgrade(newImpl);
}
```

#### 3.3.3 Delegatecall & Proxy Hazards

```solidity
// ❌ VULNERABLE: Arbitrary delegatecall
function execute(address target, bytes calldata data) external {
    (bool success,) = target.delegatecall(data);
    require(success, "Delegatecall failed");
}

// ✅ SECURE: Constrained delegatecall via UUPS
import {UUPSUpgradeable} from "solady/utils/UUPSUpgradeable.sol";

contract Bridge is UUPSUpgradeable {
    // Only owner can call upgradeToAndCall
    function _authorizeUpgrade(address) internal override onlyOwner {}
}

// Storage collision prevention via ERC-7201:
bytes32 private constant CORE_STORAGE_LOC =
    0x2a4bc94277959482747982870127b44df203291a4a265859162bc78bc4dd2ad9;
```

### 3.4 MEV / Ordering

#### 3.4.1 Deadline & Slippage (Current BTR Pattern)

```solidity
// ✅ BTR Exchange: Includes deadline and slippage
struct SwapParams {
    address tokenIn;
    address tokenOut;
    uint256 amountIn;
    uint256 minAmountOut;  // Slippage protection
    uint256 deadline;      // Time-based MEV protection
    ...
}

function swap(SwapParams calldata params) external {
    require(block.timestamp <= params.deadline, "Expired");
    uint256 amountOut = _executeSwap(params);
    require(amountOut >= params.minAmountOut, "Slippage exceeded");
}
```

#### 3.4.2 Intent/RFQ for MEV-Sensitive Operations

```solidity
// For highly MEV-sensitive operations, consider intent-based patterns
// See CoW Swap, UniswapX, 1inch Fusion for reference

// ⚠️ NOTE: BTR chose NOT to implement intent-based swapping
// to maintain permissionlessness. See Foundations.md §13-15.

// For cross-chain operations where MEV is severe:
interface RFQProvider {
    function quote(uint256 amount, address token) external returns (uint256 price, bytes signature);
}

function executeWithRFQ(uint256 amount, address provider) external {
    (uint256 price, bytes memory sig) = RFQProvider(provider).quote(amount, token);
    // Verify signature and execute at guaranteed price
    require(_verifySignature(price, sig), "Invalid quote");
}
```

#### 3.4.3 transfer() Gas Stipend Warning

```solidity
// ⚠️ WARNING: transfer() 2300 gas stipend is NOT a security boundary
// Do NOT rely on it for reentrancy prevention

// ❌ WRONG ASSUMPTION
function badWithdraw() external {
    payable(msg.sender).transfer(amount);  // Assumes this prevents reentrancy
    balances[msg.sender] -= amount;  // State change AFTER - still vulnerable!
}

// ✅ CORRECT: Use CEI pattern + reentrancy guard
function goodWithdraw() external nonReentrant {
    balances[msg.sender] -= amount;  // State change BEFORE
    payable(msg.sender).transfer(amount);
}
```

### 3.5 Oracle Security

#### 3.5.1 Staleness Checks (Current BTR Pattern)

```solidity
// ✅ BTR InternalOracle: Dual-window TWAP with staleness protection
function _readOracle(IPool.PoolStorage storage $, address token) internal returns (IOracle.FeedData memory data) {
    // Try transient cache first
    (bool found, data) = TCache.tryLoadOracleFeed(token);
    if (found) return data;

    // Try primary oracle with try/catch for DoS resilience
    try IOracle(cfg.primary).getFeed(cfg.feedId) returns (IOracle.FeedData memory feedData) {
        try IOracle(cfg.primary).isFeedFresh(cfg.feedId) returns (bool fresh) {
            if (fresh) {
                TCache.cacheOracleFeed(token, feedData);
                return feedData;
            }
        } catch { /* treat as stale */ }
    } catch { /* primary failed, try fallback */ }

    // ⚠️ SECURITY NOTE (Base:M-03):
    // Fallback oracle does NOT perform deviation checks!
    // If primary is DoS'd and secondary is manipulated, system accepts bad prices.
    // Mitigation: Use reputable oracle providers with independent infrastructure.
}
```

#### 3.5.2 TWAP Manipulation Resistance

```solidity
// ✅ BTR InternalOracle: Multi-window protection
uint32 private constant FAST_WINDOW = 300;   // 5 minutes
uint32 private constant SLOW_WINDOW = 3600;  // 1 hour

// CRITICAL-7 FIX: Staleness threshold prevents accumulator corruption
uint256 private constant MAX_FEED_HB = 604800;  // 7 days (matches shared Constants.sol:27)

function _updateFeeds(...) private {
    uint256 dt = currentTime - acc.lastUpdate;

    // Skip same-block updates to prevent manipulation
    if (dt == 0) return;

    // Cap dt to prevent accumulator corruption on stale data
    if (dt > MAX_STALENESS) {
        dt = MAX_STALENESS;
        // Reset snapshots for clean TWAP rebuild
        acc.fastSnapB64 = acc.priceAccB64;
        acc.slowSnapB64 = acc.priceAccB64;
    }
    // ... TWAP calculation
}
```

#### 3.5.3 Deviation Circuit Breakers

```solidity
// ⚠️ BTR NOTE: Deviation checks NOT currently implemented at AMM level
// Future consideration: Add price deviation circuit breaker

// Recommended pattern:
function _checkPriceDeviation(uint256 newPrice, uint256 oldPrice) internal pure {
    uint256 deviation = newPrice > oldPrice
        ? (newPrice - oldPrice) * 1e18 / oldPrice
        : (oldPrice - newPrice) * 1e18 / oldPrice;

    uint256 MAX_DEVIATION = 10e16;  // 10%
    require(deviation <= MAX_DEVIATION, "Price deviation too high");
}
```

### 3.6 Flow Guards (JIT Protection)

**IMPORTANT**: Flow guards are **UX constraints**, NOT primary security boundaries.

```solidity
// ✅ BTR Base: Correct flow guard pattern
mapping(address => mapping(address => uint32)) public lastDepositTime;
uint16 public flowCooldownSeconds = 15;  // Configurable, 0 = disabled

function _recordDeposit(address user, address asset) internal {
    _fg().lastDepositTime[user][asset] = uint32(block.timestamp);
}

function _checkWithdrawCooldown(address user, address asset) internal view {
    uint16 cooldown = _s().flowCooldownSeconds;
    if (cooldown == 0) return;  // Flow guard disabled

    uint32 lastDeposit = _fg().lastDepositTime[user][asset];
    if (lastDeposit == 0) return;  // No prior deposit

    unchecked {
        if (block.timestamp < lastDeposit + cooldown) {
            revert CooldownActive(lastDeposit + cooldown - uint32(block.timestamp));
        }
    }
}

// ⚠️ SECURITY NOTE:
// Cooldowns prevent SAME-BLOCK deposit→withdraw (JIT sandwiche attacks)
// They do NOT prevent:
// - Oracle manipulation attacks
// - Price impact via multi-block transactions
// - Cross-pool arbitrage
//
// Primary defense must be: invariant enforcement + oracle staleness checks
```

---

## 4. SVM Security (Solana - Future)

> **Status**: Not currently deployed. Reference for future Solana expansion.

### 4.1 PDA + Bump Canonicalization

```rust
// ❌ VULNERABLE: Deriving bump on-the-fly without validation
#[derive(Accounts)]
pub struct UnsafeSwap<'info> {
    #[account(
        seeds = [b"pool", authority.key().as_ref()],
        bump
    )]
    pub pool: Account<'info, Pool>,
}

// ✅ SECURE: Store and validate canonical bump
#[account]
pub struct Pool {
    pub bump: u8,  // Canonical bump stored at initialization
    pub authority: Pubkey,
}

#[derive(Accounts)]
pub struct SafeSwap<'info> {
    #[account(
        seeds = [b"pool", authority.key().as_ref()],
        bump = pool.bump,  // Validate stored canonical bump
        has_one = authority
    )]
    pub pool: Account<'info, Pool>,
    pub authority: Signer<'info>,
}

// At initialization:
pub fn create_pool(ctx: Context<CreatePool>) -> Result<()> {
    let (pool_key, bump) = Pubkey::find_program_address(
        &[b"pool", ctx.accounts.authority.key().as_ref()],
        ctx.program_id,
    );
    ctx.accounts.pool.bump = bump;  // Store canonical bump
    // ...
}
```

### 4.2 Account Validation (Beyond Owner)

```rust
// ❌ VULNERABLE: Only checking owner
#[account(constraint = user_token.owner == user.key())]
pub user_token: Account<'info, TokenAccount>,

// ✅ SECURE: Full token account validation
#[account(
    constraint = user_token.owner == user.key() @ ErrorCode::InvalidOwner,
    constraint = user_token.mint == token_mint.key() @ ErrorCode::InvalidMint,
    constraint = user_token.state == AccountState::Initialized @ ErrorCode::UninitializedAccount,
)]
pub user_token: Account<'info, TokenAccount>,

// ✅ SECURE: Validate ATA when design assumes it
#[account(
    constraint = user_token.owner == user.key() @ ErrorCode::InvalidOwner,
    constraint = user_token.mint == token_mint.key() @ ErrorCode::InvalidMint,
    constraint = Pubkey::find_program_address(
        &[&[
            user_token.owner.as_ref(),
            TOKEN_PROGRAM_ID.as_ref(),
            user_token.mint.as_ref(),
        ]],
    ).0 == user_token.key() @ ErrorCode::ExpectedATA,
)]
pub user_token: Account<'info, TokenAccount>,
```

### 4.3 CPI Safety

```rust
// ❌ VULNERABLE: Unvalidated CPI program ID
let cpi_context = CpiContext::new(
    token_program.to_account_info(),  // Could be malicious program!
    Transfer { from, to, authority },
);

// ✅ SECURE: Constrain program IDs
#[account(
    constraint = token_program.key() == TOKEN_PROGRAM_ID @ ErrorCode::InvalidTokenProgram,
    constraint = associated_token_program.key() == ASSOCIATED_TOKEN_PROGRAM_ID @ ErrorCode::InvalidATAPProgram,
    constraint = system_program.key() == SYSTEM_PROGRAM_ID @ ErrorCode::InvalidSystemProgram,
)]
pub struct SafeTransfer<'info> {
    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub system_program: Program<'info, System>,
}
```

### 4.4 Sysvar Validation

```rust
// ❌ VULNERABLE: Fake sysvar account
#[account(address = sysvar::clock::ID)]  // Only checks address!
pub clock: Sysvar<'info, Clock>,

// ✅ SECURE: Verify sysvar properly
pub fn validate_sysvar<'info>(info: &AccountInfo<'info>, expected: &Pubkey) -> Result<()> {
    require!(
        info.key() == expected && *info.owner == solana_program::system_program::ID,
        ErrorCode::InvalidSysvar
    );
    Ok(())
}
```

### 4.5 Lifecycle & Closing

```rust
// ❌ VULNERABLE: No close authority check
pub fn close_pool(ctx: Context<ClosePool>) -> Result<()> {
    let pool = &ctx.accounts.pool;
    let authority = &ctx.accounts.authority;

    // Anyone can close! Funds can be drained.
    let pool_lamports = pool.to_account_info().lamports();
    **pool.to_account_info().lamports.borrow_mut() = 0;
    **authority.lamports.borrow_mut() += pool_lamports;
    Ok(())
}

// ✅ SECURE: Explicit close authority
#[account(
    close = authority,  // Only authority can close and receive lamports
    constraint = pool.total_assets == 0 @ ErrorCode::HasAssets,
    constraint = pool.total_liabilities == 0 @ ErrorCode::HasLiabilities,
)]
pub struct ClosePool<'info> {
    #[account(mut)]
    pub pool: Account<'info, Pool>,
    pub authority: Signer<'info>,
}

// ✅ SECURE: Force-defund pattern
pub fn force_defund(ctx: Context<ForceDefund>) -> Result<()> {
    let pool = &mut ctx.accounts.pool;
    let authority = &ctx.accounts.authority;

    // Zero out all state first
    pool.total_assets = 0;
    pool.total_liabilities = 0;

    // Then close
    let pool_lamports = pool.to_account_info().lamports();
    **pool.to_account_info().lamports.borrow_mut() = 0;
    **authority.lamports.borrow_mut() += pool_lamports;
    Ok(())
}
```

---

## 5. MoveVM Security (Aptos/Sui - Future)

> **Status**: Not currently deployed. Reference for future MoveVM expansion.

### 5.1 Capability Security

```move
// ❌ VULNERABLE: No signer check
public entry fun withdraw(pool_addr: address, amount: u64) acquires Pool {
    let pool = borrow_global_mut<Pool>(pool_addr);
    // Anyone can call!
}

// ✅ SECURE: Require signer
public entry fun withdraw(account: &signer, pool_addr: address, amount: u64) acquires Pool {
    let pool = borrow_global_mut<Pool>(pool_addr);
    assert!(signer::address_of(account) == pool.owner, ENOT_AUTHORIZED);
}

// ✅ SECURE: Signer-local storage pattern (Aptos best practice)
public entry fun withdraw(account: &signer, amount: u64) acquires Pool {
    let pool_addr = signer::address_of(account);
    let pool = borrow_global_mut<Pool>(pool_addr);  // Only access signer's own pool
    // Safe because only owner can access their own pool
}
```

### 5.2 Resource Forgery Prevention

```move
// ❌ VULNERABLE: No existence check
public fun get_balance(pool_addr: address): u64 acquires Pool {
    let pool = borrow_global<Pool>(pool_addr);  // Aborts if doesn't exist
}

// ✅ SECURE: Check before borrow
public fun get_balance(pool_addr: address): u64 acquires Pool {
    assert!(exists<Pool>(pool_addr), EPOOL_NOT_FOUND);
    let pool = borrow_global<Pool>(pool_addr);
    pool.balance
}
```

### 5.3 Capability Policy per Module

```move
/// Capability Policy for Staking Module
///
/// Capability Creation:
/// - Only `create_cap()` can create StakingCapability
/// - Must be called by pool owner
///
/// Capability Storage:
/// - Stored at `signer::address_of(signer)`
/// - Only owner can borrow their own capability
///
/// Capability Borrow:
/// - Functions require `&StakingCapability` parameter
/// - Must be borrowed from signer's address
///
module aimm::staking {
    struct StakingCapability has key {
        owner: address,
        pool_addr: address,
    }

    public entry fun create_cap(account: &signer, pool_addr: address) {
        let cap = StakingCapability {
            owner: signer::address_of(account),
            pool_addr,
        };
        move_to(account, cap);
    }

    public entry fun stake(
        account: &signer,
        amount: u64
    ) acquires StakingCapability {
        let cap = borrow_global_mut<StakingCapability>(signer::address_of(account));
        // Verify capability ownership
        assert!(cap.owner == signer::address_of(account), EUNAUTHORIZED);
        // Perform stake
    }
}
```

### 5.4 Sui Object Ownership

```move
// For Sui specifically, object ownership changes the threat model

// ✅ SECURE: Owned object - only owner can modify
public struct PoolAsset has key {
    id: UID,
    owner: address,
    amount: u64,
}

// ⚠️ SHARED: Anyone can call (need access control)
public struct SharedPool has key {
    id: UID,
    total_assets: u64,
}

// Shared objects require explicit access control:
public entry fun deposit_shared(pool: &mut SharedPool, account: &signer, amount: u64) {
    // Must verify authorization since object is shared
    assert!(is_authorized(account), EUNAUTHORIZED);
    pool.total_assets += amount;
}
```

---

## 6. Cross-Chain Security

### 6.1 Message Domain Separation

**Current BTR Pattern (Bridge.sol):**

```solidity
// ✅ BTR binds source chain, peer, nonce, and payload
function lzReceive(
    LZEndpointV2.Origin calldata _origin,
    bytes32 _guid,
    bytes calldata _message,
    address,
    bytes calldata
) external payable nonReentrant {
    // Verify sender is LayerZero endpoint
    if (msg.sender != LZ_ENDPOINT) revert Unauthorized();

    // Verify peer (source chain bridge contract)
    bytes32 trustedPeer = peers[_origin.srcEid];
    if (trustedPeer == bytes32(0) || trustedPeer != _origin.sender) {
        _queueFailedMessage(_guid, to, token, amount, _origin.srcEid, 1);
        return;
    }

    // GUID provides replay protection (unique per LZ message)
    (bytes32 receiver, address token, uint256 amount) = abi.decode(_message, ...);
    // ...
}
```

**Recommended: Explicit domain separation for future bridges:**

```solidity
// ✅ SECURE: Bind all domain parameters
struct CrossChainMessage {
    uint256 sourceChainId;     // Source chain
    address sourceSender;       // Original sender
    uint256 destChainId;        // Destination chain
    address destReceiver;       // Final recipient
    uint256 nonce;              // Unique nonce
    bytes32 payloadHash;        // Hash of actual payload
}

bytes32 MESSAGE_TYPEHASH = keccak256(
    "CrossChainMessage(uint256 sourceChainId,address sourceSender,uint256 destChainId,address destReceiver,uint256 nonce,bytes32 payloadHash)"
);

function hashMessage(CrossChainMessage memory msg) internal pure returns (bytes32) {
    return keccak256(abi.encode(
        MESSAGE_TYPEHASH,
        msg.sourceChainId,
        msg.sourceSender,
        msg.destChainId,
        msg.destReceiver,
        msg.nonce,
        msg.payloadHash
    ));
}
```

### 6.2 Finality Assumptions

```solidity
// ⚠️ SECURITY NOTE: LayerZero uses optimistic verification
// - Messages are delivered after ~20 seconds (typical)
// - No ZK or light-client proof
// - Relayer network honesty assumption
// - Circuit breakers (rate limits, caps) are critical

// BTR Bridge: Per-token rate limits
struct TokenConfig {
    uint64 limitOutB64;        // Daily outbound limit
    uint64 bridgedOutB64;      // Today's outbound
    uint64 bridgedInB64;       // Today's inbound
    uint8 inRatio;             // Inbound ratio (as % of outbound)
    uint8 flags;               // SUPPORTED, PAUSED, UNLIMITED
}

// Recommended: Explicit finality parameter
enum FinalityMode {
    Optimistic,     // Current LayerZero (fast, trust relayers)
    ZKProof,        // zkSync/hyperchains (slow, trustless)
    LightClient,    // Polyhedrot/CCIP (medium, trust validators)
    Multisig        // AXELAR/wormhole (slow, trust guardians)
}
```

### 6.3 Circuit Breakers

**Current BTR protections:**

```solidity
// ✅ Per-token daily limits
function _checkAndUpdateLimit(address token, uint256 amount, Direction dir) internal {
    // Reset if new day
    uint16 today = uint16(block.timestamp / 1 days);
    if (cfg.day != today) {
        cfg.day = today;
        cfg.bridgedOutB64 = 0;
        cfg.bridgedInB64 = 0;
    }

    // Check limit
    uint64 newTotal = M.add64(cfg.bridgedOutB64, amountB64);
    if (M.gt64(newTotal, cfg.limitOutB64)) {
        revert ExcessiveAmount(...);
    }
    cfg.bridgedOutB64 = newTotal;
}

// ✅ Emergency pause (instant)
function pauseToken(address token, bool paused) external onlyOwner {
    if (paused) cfg.flags |= FLAG_PAUSED;
    else cfg.flags &= ~FLAG_PAUSED;
}
```

**Recommended additions:**

```solidity
// ✅ Per-asset caps (total bridgeable)
mapping(address => uint256) public maxSupply;  // Cap on total bridged supply

function _checkSupplyCap(address token, uint256 amount) internal view {
    uint256 currentSupply = IERC20(token).totalSupply();
    require(currentSupply + amount <= maxSupply[token], "Supply cap exceeded");
}

// ✅ Velocity limits (per-window, not just daily)
struct VelocityState {
    uint256 amount1h;
    uint256 amount24h;
    uint256 lastUpdate1h;
    uint256 lastUpdate24h;
}
```

### 6.4 Failed Message Recovery (CRITICAL-6 FIX)

**Current BTR Pattern (Bridge.sol:49-70, 179-196, 248-287):**

```solidity
// ✅ Queue failed messages instead of reverting
struct FailedMessage {
    address recipient;
    address token;
    uint256 amount;
    uint32 srcEid;
    uint64 failureTime;
    uint8 failureCode;  // 1=peer_removed, 2=token_paused, 3=rate_limit, 4=mint_failed
}

mapping(bytes32 guid => FailedMessage) public failedMessages;

function lzReceive(...) external payable nonReentrant {
    // ... validation ...

    // Non-reverting: queue if any step fails
    try IERC7802(token).crosschainMint(to, amount, ...) {
        emit Bridged(...);
    } catch {
        _queueFailedMessage(_guid, to, token, amount, _origin.srcEid, 5);
    }
}

function recoverFailedMessage(bytes32 guid) external onlyOwner nonReentrant {
    FailedMessage memory failed = failedMessages[guid];
    if (failed.amount == 0) revert InvalidState();

    // Delete before external call (CEI pattern)
    delete failedMessages[guid];

    IERC7802(failed.token).crosschainMint(
        failed.recipient,
        failed.amount,
        abi.encode(failed.srcEid, uint64(0), guid)
    );
}
```

---

## 7. Security Checklist

### 7.1 Pre-Deployment

#### Smart Contract Code

- [ ] All external calls use Try/Catch or low-level call with return checks
- [ ] Reentrancy guards on **all** state-changing functions (not just some)
- [ ] Cross-function reentrancy considered (guards on all functions in reentrancy path)
- [ ] Read-only reentrancy mitigated (re-validate state after external calls)
- [ ] Access control modifiers on privileged functions
- [ ] Input validation on all public/external functions
- [ ] Integer overflow/underflow protection (Solidity ≥0.8 or explicit checks)
- [ ] Front-running mitigation (deadlines, slippage limits)
- [ ] Emergency pause mechanism (instant for critical, timelocked for other)
- [ ] Safe ERC20/ERC721 operations (SafeTransferLib or equivalent)
- [ ] No `tx.origin` authentication
- [ ] Proper event emission for all state changes

#### DeFi Specific

- [ ] Oracle staleness checks (with configurable TTL)
- [ ] Price deviation limits (consider for future)
- [ ] Flash loan protection (invariant enforcement, not just flow guards)
- [ ] Precision loss analysis (use mulDiv for ratio math)
- [ ] Rounding direction analysis (specify floor/ceil)
- [ ] Liquidity invariant checks (assets ≥ liabilities)
- [ ] Safe ratio calculations (multiply before divide)
- [ ] Asset decimals handled correctly

#### Admin & Upgrades

- [ ] Authority model documented (owner vs guardian vs fee setter)
- [ ] Timelock durations specified per operation class
- [ ] Multisig recommended (3/5 or 5/7)
- [ ] Break-glass procedures defined
- [ ] Roles separated (pause vs upgrade vs fee setting)
- [ ] Initialization safety (constructor sets immutable state)
- [ ] Double-initialization prevented
- [ ] Upgrade executor validated
- [ ] Storage layout collision prevention (ERC-7201)
- [ ] Delegatecall constrained (UUPS pattern)

#### Cross-Chain (Bridge)

- [ ] Peer validation per source chain
- [ ] Rate limiting per token (daily + velocity)
- [ ] Per-asset caps (total bridged supply)
- [ ] Emergency pause (instant per token)
- [ ] Failed message recovery queue
- [ ] Message domain separation (source + dest + nonce)
- [ ] Finality assumptions documented
- [ ] LayerZero endpoint validation
- [ ] Non-reverting message handling

#### Testing

- [ ] Unit tests for all functions
- [ ] Integration tests with mainnet fork
- [ ] Fuzz testing for edge cases
- [ ] Invariant tests for system properties
- [ ] Gas optimization review
- [ ] Test coverage > 90%

### 7.2 Post-Audit

- [ ] All audit findings addressed or documented
- [ ] Bug bounty program launched
- [ ] Monitoring/alerting configured
- [ ] Incident response plan created
- [ ] Upgrade mechanism tested
- [ ] Timelock configured for admin actions
- [ ] Multisig deployed for owner role

---

## 8. Audit Process

### 8.1 Pre-Audit Preparation

```bash
# Deliverables for auditors
1. ✅ Complete source code (contracts/src/)
2. ✅ Test files (test/)
3. ✅ Deployment scripts (scripts/)
4. ✅ NatSpec documentation (inline comments)
5. ✅ Architecture overview (docs/concepts/foundations.md)
6. ✅ Previous audits (if any)
7. ⚠️ Threat model document (this file)
8. ⚠️ Known issues list (audit findings)
```

### 8.2 During Audit

```bash
# Provide to auditors
1. Complete source code
2. Test files
3. Deployment scripts
4. Documentation (README, NatSpec)
5. Architecture overview
6. Previous audits (if any)
7. This security document
8. Known issues from internal review
```

### 8.3 Post-Audit

- Create GitHub issues for each finding
- Categorize by severity (Critical, High, Medium, Low, Informational)
- Fix or document each finding
- Get audit confirmation for fixes
- Update documentation
- Retest fixes

---

## 9. Severity Classification

| Severity | Definition | Example |
|----------|------------|---------|
| **Critical** | Loss of user funds, protocol breakage | Reentrancy, stolen admin keys, oracle bypass |
| **High** | Significant fund loss risk | Access control bypass, price manipulation, upgrade exploit |
| **Medium** | Fund loss under specific conditions | Business logic flaw, griefing, DoS |
| **Low** | Minor issues, no fund loss | Gas optimization, unused code |
| **Informational** | Best practice suggestions | Documentation improvements |

---

## 10. Incident Response

### 10.1 Incident Classification

1. **Active Exploit**: Funds currently being drained
2. **Paused**: Protocol paused, investigating
3. **Resolved**: Issue fixed, funds recovered/returned

### 10.2 Response Steps

```bash
1. PAUSE: Trigger emergency freeze (freezeAsset)
2. ASSESS: Determine scope and impact
3. NOTIFY: Inform team and community
4. MITIGATE: Stop bleeding (if active)
5. INVESTIGATE: Root cause analysis
6. FIX: Deploy patch (via timelocked upgrade)
7. RECOVER: Attempt fund recovery
8. POST-MORTEM: Document and improve
```

### 10.3 Emergency Contacts (TODO)

- Security Team: [TODO]
- Bug Bounty: [TODO]
- Audit Firm: [TODO]
- Exchange Contacts: [TODO]

---

## Appendix A: BTR-Specific Security Notes

### A.1 Current Security Mechanisms

| Mechanism | Implementation | Status |
|-----------|----------------|--------|
| **Reentrancy Guard** | Transient storage (Base:23-45) | EIP-1153 chains only |
| **Reentrancy Guard (Bridge)** | Solady storage-based | All chains |
| **Timelock** | Multi-tier (1-7 days) | Operational |
| **Oracle** | Internal TWAP + external fallback | Operational |
| **Flow Guard** | 15-second cooldown | UX constraint, not primary |
| **Bridge** | LayerZero with failed message queue | Operational |
| **Access Control** | Single owner | ⚠️ Recommend multisig |

### A.2 Known Security Considerations

1. **Base:M-03**: Oracle fallback bypasses deviation checks
   - **Mitigation**: Use reputable oracle providers with independent infrastructure
   - **Future**: Add deviation circuit breaker at AMM level

2. **Single Owner**: No multisig currently
   - **Recommendation**: Deploy Safe multisig as owner
   - **Timelock**: Provides some protection against hasty decisions

3. **Transient Storage**: Not available on all EVM chains
   - **Mitigation**: Use storage-based guard on non-EIP-1153 chains
   - **Documentation**: Clearly document network compatibility

### A.3 Architecture-Specific Risks

**Diamond-Lite Pattern (PoolProxy)**
- Risk: Storage collision if modules use same slot
- Mitigation: ERC-7201 storage namespaces enforced

**Internal Oracle (InternalOracle)**
- Risk: TWAP manipulation via low-liquidity swaps
- Mitigation: Dual-window (5min + 1hr) + staleness threshold

**LayerZero Bridge (Bridge)**
- Risk: Optimistic verification assumes honest relayers
- Mitigation: Rate limits, failed message queue, emergency pause

---

## Internal References

- [`Foundations`](../../concepts/foundations.md) - Intellectual lineage and design choices
- [`GIT.md`](./GIT.md) - Commit conventions
- [`FRONTEND.md`](./FRONTEND.md) - Frontend development
- [`BACKEND.md`](./BACKEND.md) - Backend development
- [`SMART_CONTRACTS.md`](./SMART_CONTRACTS.md) - Smart contract development
- [`QUANT.md`](./QUANT.md) - Quant/research

---

*Last updated: 2025-01*
