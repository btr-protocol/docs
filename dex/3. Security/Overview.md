# Security Overview

> Defense-in-depth architecture for AIMM protocol security

---

## 1. Security Philosophy

AIMM employs a **defense-in-depth** approach with multiple independent security layers:

1. **Economic Security**  Incentive alignment and manipulation resistance
2. **Access Control**  Timelocked governance and role separation
3. **Operational Security**  Circuit breakers and emergency controls
4. **Code Security**  Audits, testing, and formal verification

---

## 2. Threat Model

### 2.1. Adversary Capabilities

| Adversary | Capabilities | Mitigations |
|-----------|--------------|-------------|
| **Flash Loan Attacker** | Unlimited capital for single transaction | Same-block oracle rejection, TWAP pricing |
| **Whale Trader** | Large positions, multi-block attacks | Dual-window TWAPs, inventory skew bounds |
| **MEV Searcher** | Transaction ordering, sandwich attacks | Slippage protection, directional fees |
| **Oracle Manipulator** | External feed compromise | Dual oracle, deviation detection |
| **Governance Attacker** | Majority token control | Timelocks, grace periods |
| **Smart Contract Exploiter** | Code vulnerability discovery | Audits, bug bounties, upgradability |

### 2.2. Trust Assumptions

| Component | Trust Level | Justification |
|-----------|-------------|---------------|
| **Owner** | Trusted | Multi-sig with timelock |
| **Treasury** | Trusted | Fee collection only |
| **Bridge** | Semi-trusted | Limited to mint/burn operations |
| **External Oracles** | Verified | Deviation checking, fallbacks |
| **Users** | Untrusted | All inputs validated |

---

## 3. Security Layers

### 3.1. Layer 1: Economic Security

**Manipulation Resistance:**
- **Dual TWAP Windows**  5-minute (fast) and 1-hour (slow) averages
- **Same-Block Rejection**  Oracle updates skip same-block trades
- **Inventory Skew Bounds**  Maximum ±100 skew caps price impact
- **Coverage Floor**  50% minimum prevents complete drainage

**Incentive Alignment:**
- **Directional Fees**  Toxic flow pays higher spread
- **LP Haircuts**  Undercollateralized withdrawals penalized
- **Decay Mechanism**  Long-term undercollateralization resolved

### 3.2. Layer 2: Access Control

**Role Separation:**
| Role | Permissions | Holder |
|------|-------------|--------|
| Owner | All admin functions (timelocked) | Multi-sig wallet |
| Treasury | Collect protocol fees | Treasury contract |
| Bridge | Mint/burn for cross-chain | Bridge contract |
| Oracles | Push price updates | Authorized keepers |

**Timelock Tiers:**
- **CRITICAL (7 days)**  Base token migration
- **HIGH (3 days)**  Ownership, modules, treasury, bridge
- **BASE (2 days)**  Oracle changes, staking config
- **LOW (1 day)**  Add asset, fee updates

**Grace Period:** 7 days to execute after timelock expires.

### 3.3. Layer 3: Operational Security

**Circuit Breakers:**
```solidity
// Feature flags (per asset)
FROZEN_BIT            = 1 << 0   // Emergency freeze
SWAP_ENABLED_BIT      = 1 << 1   // Swap operations
LIABILITY_SWAP_BIT    = 1 << 2   // LP position swaps
DECAY_ENABLED_BIT     = 1 << 3   // Liability decay
FLASH_ENABLED_BIT     = 1 << 4   // Flash loans
```

**Emergency Controls:**
- `freezeAsset()`  Immediate halt (no timelock)
- `unfreezeAsset()`  Restore operations
- Per-asset granularity

**Thresholds:**
- `coverageMin`  Minimum healthy coverage (default 50%, parametric)
- `coverageMax`  Maximum healthy coverage (default 200%, parametric)
- `decayStartRatioBps`  Decay activation threshold
- `minLiquidity`  Reserve floor per asset

### 3.4. Layer 4: Code Security

**Development Practices:**
- Solidity ^0.8.33 with overflow checks
- Extensive natspec documentation
- Custom error types (gas efficient)
- Modular architecture (isolated concerns)

**Testing:**
- Unit tests for all functions
- Integration tests for flows
- Fuzz testing for edge cases
- Invariant testing for properties

**External Review:**
- Third-party audits — Pre-launch security review by professional auditors

---

## 4. Storage Security

### 4.1. ERC-7201 Namespaced Storage

Each module uses isolated storage slots:

```solidity
bytes32 constant CORE_STORAGE_LOC =
    keccak256("pool.storage.base.v1") - 1;

bytes32 constant ORACLE_STORAGE_LOC =
    keccak256("pool.storage.oracle.v1") - 1;

bytes32 constant STAKING_STORAGE_LOC =
    keccak256("pool.storage.staking.v1") - 1;

bytes32 constant DISTRIBUTOR_STORAGE_LOC =
    keccak256("pool.storage.distributor.v1") - 1;
```

**Benefits:**
- No slot collisions between modules
- Safe upgrades without storage migration
- Clear separation of concerns

### 4.2. Transient Storage (EIP-1153)

Used for transaction-scoped data:
- Reentrancy guards
- Oracle price caching
- Flash loan state

**Security Properties:**
- Auto-cleared after transaction
- Cannot persist malicious state
- Gas efficient

---

## 5. Reentrancy Protection

### 5.1. Transient Guard Pattern

```solidity
modifier nonReentrant() {
    assembly {
        if tload(GUARD_SLOT) { revert(0, 0) }
        tstore(GUARD_SLOT, 1)
    }
    _;
    assembly {
        tstore(GUARD_SLOT, 0)
    }
}
```

**Protected Operations:**
- All external token transfers
- Flash loan callbacks
- Cross-module calls

---

## 6. Oracle Security

### 6.1. Internal Oracle

**Manipulation Resistance:**
| Attack | Defense |
|--------|---------|
| Flash loan | Same-block rejection |
| Multi-block | Dual-window TWAP |
| Low liquidity | External fallback |
| Staleness | TTL-based freshness check |

### 6.2. External Oracle

**Validation:**
- Deviation bounds checking
- Staleness detection
- Multi-source aggregation option

**Fallback Chain:**
```
1. Primary external  2. Secondary external  3. Internal  4. Revert
```

See: [Oracles](/docs/3.5-Oracles)

---

## 7. Upgrade Security

### 7.1. Diamond-Lite Proxy

**Module Registry:**
```solidity
mapping(bytes4 => address) modules;  // selector  implementation
```

**Upgrade Process:**
1. Request module update (3-day timelock)
2. Wait for delay
3. Execute within 7-day grace
4. Old module removed, new module active

**Immutable Components:**
- Proxy contract itself
- Storage slot locations
- Core libraries

See: [Deployment & Upgrades](/docs/3.2-Deployment-&-Upgrades)

---

## 8. Emergency Procedures

### 8.1. Security Incident Response

**Phase 1: Immediate (0-1 hour)**
1. Identify affected assets
2. Execute `freezeAsset()` on affected tokens
3. Alert team and security partners
4. Begin investigation

**Phase 2: Assessment (1-24 hours)**
1. Determine root cause
2. Assess damage extent
3. Identify fix requirements
4. Prepare communication

**Phase 3: Remediation (1-7+ days)**
1. Develop and test fix
2. Submit timelocked upgrade
3. Coordinate disclosure
4. Execute fix after delay

**Phase 4: Recovery**
1. Unfreeze assets
2. Monitor closely
3. Post-mortem analysis
4. Update procedures

### 8.2. Contact Information

For security disclosures:
- Security email: sec@btr.supply
- Response SLA: 24 hours

---

## 9. Known Limitations

### 9.1. Economic Bounds

| Limitation | Bound | Implication |
|------------|-------|-------------|
| Max skew | ±100 | ~100% premium/discount at extremes |
| Max depth | 4 levels | Complex routing limited |
| Max hops | 6 | Long paths split execution |

### 9.2. Oracle Limitations

| Limitation | Mitigation |
|------------|------------|
| Low volume  stale TWAP | External fallback, shorter TTL |
| Fast price movement | Wider spreads via volatility |
| External feed failure | Multi-source, internal fallback |

### 9.3. Governance Limitations

| Limitation | Mitigation |
|------------|------------|
| Owner compromise | Timelock delays, multi-sig |
| Slow emergency response | Immediate freeze (no timelock) |
| Governance attack | Grace period limits exposure |

---

## 10. Security Checklist

### 10.1. For Pool Deployers

- [ ] Multi-sig wallet for owner
- [ ] Reasonable initial parameters
- [ ] External oracle fallback configured
- [ ] Emergency contacts established
- [ ] Monitoring systems active

### 10.2. For LPs

- [ ] Understand coverage mechanics
- [ ] Monitor coverage ratios
- [ ] Understand withdrawal haircuts
- [ ] Verify pool parameters

### 10.3. For Traders

- [ ] Set appropriate slippage
- [ ] Verify pool coverage
- [ ] Check oracle freshness
- [ ] Understand fee structure

---

## 11. Related Documentation

- [Deployment & Upgrades](/docs/3.2-Deployment-&-Upgrades) — Upgrade procedures and deployment
- [Access Control](/docs/3.3-Access-Control) — Roles and permissions
- [Flow Guards](/docs/3.4-Flow-Guards) — JIT protection, cooldowns
- [Oracles](/docs/3.5-Oracles) — Oracle security and fallbacks
- [Admin Module](/docs/1.2.3-Admin) — Administrative functions
