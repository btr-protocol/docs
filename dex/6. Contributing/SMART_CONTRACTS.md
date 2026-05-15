---
title: "Smart Contract Developer Best Practices"
description: "Architecture note (post Phase 42H) -code samples below that reference PoolProxy, the Diamond pattern, ERC-7201 namespaced storage, or delegatecall-based module"
audience: tech
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# Smart Contract Developer Best Practices

**Tech Stack**: EVM (Solidity) + SVM (Rust) + MoveVM (Move)

> **Architecture note (post Phase 42H)** -code samples below that reference `PoolProxy`, the Diamond pattern, ERC-7201 namespaced storage, or `delegatecall`-based module dispatch describe **generic Solidity patterns** for educational purposes; they do **not** reflect the BTR DEX repo's current architecture. The DEX uses standalone singletons + EIP-1167 `Pool` clones (no Diamond, no delegatecall, no ERC-7201). See [3.2 Deployment & Upgrades](../../dex/3.\ Security/3.2.\ Deployment\ &\ Upgrades.md) for the current model.

---

## Quick Reference

| Area | EVM | SVM | MoveVM |
|------|-----|-----|--------|
| **Language** | Solidity 0.8.33 (exact) | Rust | Move |
| **Framework** | Foundry | Anchor | Aptos Framework / Sui Framework |
| **Testing** | Forge Tests | Unit Tests | Move Unit Tests |
| **Deploy Tool** | Forge Create | Solana CLI | Aptos CLI / Sui CLI |
| **Security** | Slither, Foundry | Semper, Anchor Security | Move Prover |

---

## 2. Solidity Version Policy

**⚠️ ALL contracts MUST use Solidity 0.8.33**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.33;  // Exact version - no ^
```

**Rules**:
- `foundry.toml`: `solc_version = "0.8.33"` (keep in sync)
- All `.sol` files: `pragma solidity 0.8.33;` (exact, no `^`)
- No floating pragma in production

---

## 3. CREATE3 Deterministic Deployments

All contracts use CreateX CREATE3 for **identical addresses across all chains** (local fork, testnet, mainnet).

### Salt Allocation

| Contract Type | Salt File | Address Pattern | Usage |
|--------------|-----------|----------------|-------|
| Core (Pools, Bridge, Treasury) | `salts/b712_b712.txt` (lower b712) | `0xb712...b712` | Pool proxies, infrastructure |
| Mock ERC20 Tokens | `salts/bbbb_bbb.txt` | `0xbbbb...bb` | Test tokens (local/testnet) |

### Environment Variables

Required in `.env` (monorepo root):

```bash
DEPLOYER=0x0a37aEc263CbA0aaBC09Bac56A0F2074a22E69A3   # Deployer address
DEPLOYER_PK=0x...                                     # Private key (SAME on all chains)
CREATEX=0xba5Ed099633D3B313e4D5F7bdc1305d3c28ba5Ed     # CreateX (available on all chains)
TREASURY=0x...                                        # Treasury address
```

### Deterministic Addresses

With the same `DEPLOYER_PK`, contracts deploy to identical addresses:

```solidity
// Pool Zero: 0xb7127AE785907441BFBC6C7bDAcC339CD7e2b712
// Pool Stable: 0xb712dCA09c4327daC7789EA34574783dC554b712
```

### Chain Metadata

Chain-specific constants in `contracts/src/utils/meta/`:

```solidity
import {BNBChainMeta} from "@utils/meta/BNBChain.sol";

contract MyContract is BNBChainMeta {
    function getUSDC() public pure returns (address) {
        return __tokens().usdc;  // 0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d
    }
}
```

### Deployment Flow

**Local Fork** (`chainId: 31337`):
```bash
bun scripts/dev.ts          # Uses DEPLOYER_PK from .env
bun scripts/dev.ts --reset  # Fresh deployment
```

**Testnet/Mainnet**:
```bash
cd contracts
forge script script/DeployBSCFork.s.sol --rpc-url $RPC_URL --broadcast
```

### Salt Verification

Test salt computations in `contracts/test/CreateX.t.sol`:

```solidity
contract CreateXTest is Test, BNBChainMeta {
    function test_ProductionSalts() public {
        // Validates all salt/address pairs from salts/*.txt
    }
}
```

---

## 4. EVM Solidity

### Project Structure

```
contracts/
├── src/
│   ├── interfaces/
│   │   ├── external/       # External protocol interfaces (ERC20, IERC3156, etc.)
│   │   └── modules/        # Internal module interfaces (ICore, IOracle, etc.)
│   ├── modules/            # Core implementation modules
│   ├── libraries/          # Reusable utility libraries
│   ├── tokens/             # Token implementations
│   ├── oracles/            # Oracle implementations
│   └── utils/              # Utility contracts
├── test/
│   ├── unit/               # Unit tests for libraries
│   ├── integration/        # Integration tests with mainnet forks
│   ├── fixtures/           # Test fixtures and helpers
│   └── simulations/        # Large-scale simulations
└── script/                 # Deployment and utility scripts
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Contracts | PascalCase | `PoolProxy`, `Exchange` |
| Interfaces | PascalCase with `I` prefix | `IPool`, `ICore` |
| Libraries | PascalCase with descriptive name | `LibMaths`, `LibConstants` |
| Functions | camelCase | `getSwapQuote`, `deposit` |
| Variables | camelCase | `userBalance`, `totalSupply` |
| Constants | UPPER_SNAKE_CASE | `MAX_FEE_BPS`, `MIN_LIQUIDITY` |
| Events | PascalCase | `SwapExecuted`, `LiquidityAdded` |
| Custom Errors | PascalCase with `Error` suffix | `InsufficientLiquidityError` |
| Structs | PascalCase | `Asset`, `SwapParams` |

### Foundry Configuration

```toml
# foundry.toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.33"
optimizer = true
optimizer_runs = 200
via_ir = true
bytecode_hash = "none"
cbor_metadata = false

[profile.default.fuzz]
runs = 256

[profile.default.invariant]
runs = 256
depth = 15

[fmt]
line_length = 100
tab_width = 4
bracket_spacing = true
int_types = "preserve"
multiline_func_header = "params_first"

# See https://github.com/foundry-rs/foundry/tree/master/config
```

### ERC-7201 Namespaced Storage Pattern

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.33;

import { IERC5267 } from "./interfaces/external/IERC5267.sol";

/// @dev Storage layout with ERC-7201 namespace
/// @custom:storage-location erc7201:btr.pool.storage
struct PoolStorage {
    mapping(address => uint256) balances;
    uint256 totalLiquidity;
    mapping(bytes32 => Asset) assets;
}

// Namespace slot calculation
// bytes32 internal constant POOL_STORAGE_LOCATION =
//     keccak256(abi.encode(uint256(keccak256("btr.pool.storage")) - 1)) & ~bytes32(uint256(0xff));

contract Pool {
    // Storage slot for namespaced storage
    bytes32 internal constant POOL_STORAGE_SLOT =
        0x...; // calculated namespace

    // Get storage with inline assembly
    function _getPoolStorage() internal pure returns (PoolStorage storage $) {
        assembly {
            $.slot := POOL_STORAGE_SLOT
        }
    }
}
```

### Modular Diamond Proxy Pattern

```solidity
// PoolProxy.sol - Lightweight proxy
contract PoolProxy {
    mapping(bytes4 => address) private _modules;

    error UnauthorizedModule();
    error FunctionNotFound();

    event ModuleRegistered(bytes4 selector, address module);
    event ModuleRemoved(bytes4 selector);

    function registerModule(bytes4 selector, address module) external onlyOwner {
        _modules[selector] = module;
        emit ModuleRegistered(selector, module);
    }

    fallback() external payable {
        bytes4 selector = msg.sig;
        address module = _modules[selector];
        if (module == address(0)) {
            revert FunctionNotFound();
        }
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), module, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```

### Gas Optimization Patterns

```solidity
// 1. Storage packing - optimize struct layout
struct Asset {
    uint96 reserve;          // 96 bits
    uint96 liability;        // 96 bits
    uint16 flags;            // 16 bits (bit fields)
    uint8 decimals;          // 8 bits
    uint32 lastUpdate;       // 32 bits
    // Total: 32 bytes (2 storage slots)
}

// 2. Use uint256 where possible (cheaper than smaller uints)
function add(uint256 a, uint256 b) internal pure returns (uint256) {
    unchecked {
        return a + b;  // Safe if overflow check not needed
    }
}

// 3. Cache storage reads in memory
function process(address user) external {
    PoolStorage storage $ = _getPoolStorage();
    uint256 balance = $.balances[user]; // SLOAD
    $.balances[user] = balance + 1;     // SSTORE
}

// 4. Use calldata instead of memory for read-only params
function execute(bytes calldata data) external {
    // data is in calldata, cheaper to read
}

// 5. Short circuit error checking
require(valid && authorized, "Unauthorized"); // Order matters!

// 6. Use custom errors instead of require strings
error InvalidAmount();
error Unauthorized();

if (amount == 0) revert InvalidAmount();
```

### Safe Math Patterns (Solidity 0.8+)

```solidity
// Solidity 0.8+ has built-in overflow checks
// Use unchecked block only when you're sure it won't overflow

function sum(uint256[] calldata values) external pure returns (uint256) {
    uint256 total;
    unchecked {
        for (uint256 i = 0; i < values.length; ++i) {
            total += values[i]; // Safe if we check preconditions
        }
    }
    return total;
}
```

### Reentrancy Protection with Transient Storage

```solidity
contract Protected {
    // EIP-1153 transient storage for reentrancy guard
    bytes32 private constant REENTRANCY_SLOT =
        0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa;

    modifier nonReentrant() {
        assembly {
            if tload(REENTRANCY_SLOT) { revert(0, 0) }
            tstore(REENTRANCY_SLOT, 1)
        }
        _;
        assembly {
            tstore(REENTRANCY_SLOT, 0)
        }
    }

    function withdraw() external nonReentrant {
        // Sensitive operation protected
    }
}
```

### Testing Patterns

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.33;

import "forge-std/Test.sol";
import { Pool } from "../../src/modules/Pool.sol";

contract PoolTest is Test {
    Pool pool;
    address owner = address(0x1);
    address user = address(0x2);

    function setUp() public {
        vm.startPrank(owner);
        pool = new Pool();
        vm.stopPrank();
    }

    function testDeposit() public {
        // Arrange
        uint256 amount = 100 ether;

        // Act
        vm.prank(user);
        pool.deposit{value: amount}();

        // Assert
        assertEq(pool.balanceOf(user), amount);
    }

    function testRevertWhen_InsufficientBalance() public {
        vm.prank(user);
        vm.expectRevert(abi.encodeWithSelector(Pool.InsufficientBalance.selector));
        pool.withdraw(100 ether);
    }

    functionFuzz_testWithdraw(uint256 amount) public {
        // Fuzz testing
        vm.assume(amount <= type(uint128).max);
        // Test logic...
    }

    function testFork_Mainnet() public {
        // Fork test
        vm.createSelectFork("mainnet", 19_500_000);
        // Test with real mainnet state
    }
}
```

### Deployment Script Patterns

```solidity
// script/Deploy.s.sol
// SPDX-License-Identifier: MIT
pragma solidity 0.8.33;

import "forge-std/Script.sol";
import { PoolProxy } from "../src/PoolProxy.sol";
import { Exchange } from "../src/modules/Exchange.sol";

contract DeployScript is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        // Deploy modules
        Exchange exchange = new Exchange();
        console.log("Exchange deployed:", address(exchange));

        // Deploy proxy
        PoolProxy proxy = new PoolProxy();
        proxy.registerModule(exchange.swap.selector, address(exchange));

        vm.stopBroadcast();
    }
}
```

---

## 5. SVM (Solana) Rust

### Project Structure

```toml
# Anchor.toml
[toolchain]

[features]
seeds = false
skip-lint = false

[programs.localnet]
my_program = "Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLn"

[registry]
url = "https://api.apr.dev"

[provider]
cluster = "Localnet"
wallet = "~/.config/solana/id.json"

[scripts]
test = "yarn run ts-mocha -p ./tsconfig.json -t 1000000 tests/**/*.ts"
```

### Anchor Program Structure

```rust
use anchor_lang::prelude::*;

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLn");

#[program]
pub mod my_program {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        let state = &mut ctx.accounts.state;
        state.authority = ctx.accounts.authority.key();
        state.bump = ctx.bumps.state;
        Ok(())
    }

    pub fn swap(ctx: Context<Swap>, amount_in: u64) -> Result<()> {
        // Validation
        require!(amount_in > 0, ErrorCode::InvalidAmount);

        // Execute swap
        // ...

        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(
        init,
        payer = authority,
        space = 8 + State::LEN,
        seeds = [b"state"],
        bump
    )]
    pub state: Account<'info, State>,
    #[account(mut)]
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Swap<'info> {
    #[account(
        mut,
        seeds = [b"state"],
        bump = state.bump
    )]
    pub state: Account<'info, State>,
    /// CHECK: Verified in handler
    pub user: AccountInfo<'info>,
    pub token_program: Program<'info, Token>,
}

#[account]
pub struct State {
    pub authority: Pubkey,
    pub bump: u8,
}

impl State {
    pub const LEN: usize = 32 + 1;
}

#[error_code]
pub enum ErrorCode {
    #[msg("Invalid amount")]
    InvalidAmount,
    #[msg("Unauthorized")]
    Unauthorized,
}
```

### SVM Security Best Practices

| Pattern | Description |
|---------|-------------|
| **PDA Validation** | Always verify PDA derivation with seeds and bump |
| **Signer Checks** | Use `#[account(signer)]` for accounts that must sign |
| **Owner Checks** | Verify token account ownership before transfers |
| **Rent Exemption** | Ensure PDAs are rent-exempt |
| **CPI Safety** | Validate all CPI calls, check program IDs |
| **Overflow Safety** | Rust's built-in overflow checks - use `checked_*` methods |

### Common SVM Vulnerabilities

```rust
// ❌ WRONG: Missing PDA validation
#[derive(Accounts)]
pub struct UnsafeSwap<'info> {
    pub user: AccountInfo<'info>,
    pub user_token: Account<'info, TokenAccount>,
}

// ✅ CORRECT: Proper validation
#[derive(Accounts)]
pub struct SafeSwap<'info> {
    #[account(mut)]
    pub user: Signer<'info>,
    #[account(
        mut,
        constraint = user_token.owner == user.key() @ ErrorCode::InvalidOwner
    )]
    pub user_token: Account<'info, TokenAccount>,
}
```

---

## 6. MoveVM (Aptos/Sui)

### Move Language Basics

```move
// Basic Move module
module my_project::pool {
    use aptos_framework::coin::{Self, Coin};
    use aptos_framework::aptos_coin::AptosCoin;
    use aptos_std::table::{Self, Table};
    use std::signer;

    /// Error codes
    const ENOT_AUTHORIZED: u64 = 1;
    const EINSUFFICIENT_BALANCE: u64 = 2;

    /// Pool resource - stored at an address
    struct Pool has key {
        owner: address,
        balances: Table<address, u64>,
        total_supply: u64,
    }

    /// Initialize pool
    public entry fun init_pool(account: &signer) {
        let owner = signer::address_of(account);
        assert!(!exists<Pool>(owner), ENOT_AUTHORIZED);

        move_to(account, Pool {
            owner,
            balances: table::new(),
            total_supply: 0,
        });
    }

    /// Deposit tokens
    public entry fun deposit(pool_addr: address, account: &signer, amount: u64) acquires Pool {
        let pool = borrow_global_mut<Pool>(pool_addr);

        let user_addr = signer::address_of(account);
        let balance = table::borrow_mut(&mut pool.balances, user_addr);
        *balance = *balance + amount;
        pool.total_supply = pool.total_supply + amount;
    }

    /// Get balance
    public fun get_balance(pool_addr: address, user: address): u64 acquires Pool {
        let pool = borrow_global<Pool>(pool_addr);
        if (table::contains(&pool.balances, user)) {
            *table::borrow(&pool.balances, user)
        } else {
            0
        }
    }
}
```

### MoveVM Security Best Practices

| Pattern | Description |
|---------|-------------|
| **Linear Types** | Resources cannot be copied - must be moved or borrowed |
| **Borrow Checker** | Compile-time memory safety prevents double-spends |
| **Capability Security** | Use `&signer` for authorized operations |
| **Resource Isolation** | Each resource type stored separately at address |
| **Formal Verification** | Use Move Prover for critical code |
| **Access Control** | Check `signer::address_of()` against owner |

### Common MoveVM Vulnerabilities

```move
// ❌ WRONG: No ownership check
public fun withdraw(pool_addr: address, amount: u64) acquires Pool {
    let pool = borrow_global_mut<Pool>(pool_addr);
    // Anyone can withdraw!
}

// ✅ CORRECT: Verify signer is owner
public entry fun withdraw(account: &signer, pool_addr: address, amount: u64) acquires Pool {
    let pool = borrow_global_mut<Pool>(pool_addr);
    assert!(signer::address_of(account) == pool.owner, ENOT_AUTHORIZED);
    // Safe to proceed
}
```

---

## 7. Cross-VM Development Guidelines

### Common Patterns Across VMs

| Concept | EVM | SVM | MoveVM |
|---------|-----|-----|--------|
| **Entry Point** | `public/external` functions | `pub fn` with `Context` | `public entry fun` |
| **State** | Storage variables | Account data | Resources |
| **Access Control** | `onlyOwner`, modifiers | Signer checks | `&signer` checks |
| **Events** | `emit Event(...)` | `msg!` macro | No native events (use events on Aptos) |
| **Errors** | `revert ErrorName()` | `err!(ErrorCode)` | `assert!(condition, ERROR_CODE)` |

### Platform-Specific Considerations

**EVM (Solidity)**:
- Gas costs dominated by SLOAD/SSTORE
- Use assembly for optimization
- Delegatecall for proxy patterns
- Transient storage (EIP-1153) for temporary data

**SVM (Rust/Anchor)**:
- Account rent exemption required
- Parallel processing for non-overlapping accounts
- CPI (Cross-Program Invocation) for composability
- PDAs for program-controlled accounts

**MoveVM (Move)**:
- Linear resource model prevents copying
- Formal verification with Move Prover
- Capability-based security model
- No reentrancy possible (by design)

---

## 8. Testing Strategy

### Unit Tests

```solidity
// EVM: Forge
function testSwap() public {
    // Test isolated logic
}

// SVM: Anchor
#[test]
fn test_swap() {
    // Test isolated logic
}

// MoveVM: Move
#[test]
fun test_swap() {
    // Test isolated logic
}
```

### Integration Tests (Mainnet Fork)

```solidity
// EVM: Forge fork
function testFork_Mainnet() public {
    vm.createSelectFork("mainnet", 19_500_000);
    // Test with real contracts
}
```

### Invariant/Fuzz Testing

```solidity
// EVM: Forge invariant
contract InvariantTest is Test {
    Pool pool;

    function invariant_totalSupplyEqualsSumBalances() public view {
        uint256 total = pool.totalSupply();
        uint256 sum;
        // Sum all user balances
        assertEq(total, sum);
    }
}
```

---

## 9. Deployment Best Practices

### Pre-Deployment Checklist

- [ ] All tests passing (unit, integration, fuzz, invariant)
- [ ] Gas optimization reviewed
- [ ] Security audit completed
- [ ] Bytecode verified with source
- [ ] Initialization parameters documented
- [ ] Emergency procedures defined
- [ ] Upgrade mechanism tested

### Deployment Steps

```bash
# EVM: Forge deployment
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --verify \
  -vvvv

# SVM: Solana deployment
anchor deploy --provider.cluster mainnet

# MoveVM: Aptos deployment
aptos move publish \
  --address my_account \
  --path aptos-move \
  --assume-yes
```

---

## 10. Code Style Guidelines

### Solidity Style

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.33;

import { IERC20 } from "./interfaces/external/IERC20.sol";

/// @title Pool Contract
/// @notice Implements liquidity pool functionality
/// @dev Uses ERC-7201 namespaced storage
contract Pool is IPool {
    /*//////////////////////////////////////////////////////////////
                                EVENTS
    //////////////////////////////////////////////////////////////*/

    event Deposit(address indexed user, uint256 amount);

    /*//////////////////////////////////////////////////////////////
                                ERRORS
    //////////////////////////////////////////////////////////////*/

    error InsufficientBalance(uint256 requested, uint256 available);

    /*//////////////////////////////////////////////////////////////
                            STATE VARIABLES
    //////////////////////////////////////////////////////////////*/

    /// @notice Pool configuration
    Config private config;

    /*//////////////////////////////////////////////////////////////
                              CONSTANTS
    //////////////////////////////////////////////////////////////*/

    uint256 private constant MAX_FEE_BPS = 10_000;

    /*//////////////////////////////////////////////////////////////
                            CONSTRUCTOR
    //////////////////////////////////////////////////////////////*/

    constructor(Config memory _config) {
        config = _config;
    }

    /*//////////////////////////////////////////////////////////////
                          EXTERNAL FUNCTIONS
    //////////////////////////////////////////////////////////////*/

    /// @notice Deposit tokens into pool
    /// @param amount Amount to deposit
    function deposit(uint256 amount) external {
        _deposit(msg.sender, amount);
    }

    /*//////////////////////////////////////////////////////////////
                          INTERNAL FUNCTIONS
    //////////////////////////////////////////////////////////////*/

    function _deposit(address user, uint256 amount) internal {
        // Implementation
    }
}
```

---

## 11. Common Patterns Reference

### Ownable Pattern

```solidity
abstract contract Ownable {
    address private _owner;

    event OwnershipTransferred(address indexed from, address indexed to);

    error NotOwner();

    constructor() {
        _owner = msg.sender;
    }

    modifier onlyOwner() {
        if (msg.sender != _owner) revert NotOwner();
        _;
    }

    function owner() public view returns (address) {
        return _owner;
    }

    function transferOwnership(address newOwner) external onlyOwner {
        _owner = newOwner;
        emit OwnershipTransferred(msg.sender, newOwner);
    }
}
```

### Pausable Pattern

```solidity
abstract contract Pausable {
    bool private _paused;

    error WhenPaused();
    error WhenNotPaused();

    event Paused(address account);
    event Unpaused(address account);

    modifier whenNotPaused() {
        if (_paused) revert WhenPaused();
        _;
    }

    modifier whenPaused() {
        if (!_paused) revert WhenNotPaused();
        _;
    }

    function pause() external {
        _paused = true;
        emit Paused(msg.sender);
    }

    function unpause() external {
        _paused = false;
        emit Unpaused(msg.sender);
    }
}
```

---

## 12. Development Workflow

```bash
# EVM / Foundry
forge build                    # Compile contracts
forge test                     # Run tests
forge test -vvv                # Verbose test output
forge test --fuzz-runs 1000    # Fuzz testing
forge fmt                      # Format code
forge snapshot                 # Update gas snapshots
forge script script/Deploy.s.sol --broadcast  # Deploy

# SVM / Anchor
anchor build                   # Build program
anchor test                    # Run tests
anchor deploy                  # Deploy to configured cluster
anchor verify <PROGRAM_ID>     # Verify on explorer

# MoveVM / Aptos
aptos move compile             # Compile Move modules
aptos move test                # Run tests
aptos move publish             # Publish to chain
```

---

## 13. Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Integer overflow/underflow | Use Solidity 0.8+ or explicit checks |
| Reentrancy | Use nonReentrant modifier or transient storage |
| Storage collisions | Use ERC-7201 namespaced storage |
| Unchecked external calls | Handle return values, use Try/Catch patterns |
| Front-running | Use commit-reveal or timelock |
| Gas griefing | Limit loop iterations, batch operations |
| Authorization bypass | Validate all inputs, check msg.sender |
| Default values | Initialize all struct fields explicitly |

---

## 14. Security Checklist

### Pre-Audit

- [ ] All functions have `@notice` and `@dev` NatSpec comments
- [ ] Custom errors defined for all revert cases
- [ ] Events emitted for all state changes
- [ ] Access control on all privileged functions
- [ ] Input validation on all external functions
- [ ] Reentrancy guards on state-changing functions
- [ ] Integer overflow protection
- [ ] Safe external call patterns
- [ ] Emergency pause/withdraw mechanisms
- [ ] Comprehensive test coverage (>90%)

### Post-Audit

- [ ] All audit findings addressed
- [ ] Bug bounty program active
- [ ] Monitoring/alerting configured
- [ ] Incident response plan documented
- [ ] Upgrade mechanism tested

---

## Internal References

- [`CONTRIBUTING.md`](../CONTRIBUTING.md) - Commit conventions
- [`SECURITY.md`](./SECURITY.md) - Security best practices

---

*Last updated: 2025-01*
