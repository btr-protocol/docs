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
| **Bridge** (LayerZero v2 OApp endpoint) | Semi-trusted | Limited to mint/burn operations |
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

### 4.1. Standalone-Contract Storage Isolation (Phase 42H)

Storage isolation is achieved structurally: every contract is standalone with its own default storage layout — there is no `delegatecall`, so cross-contract slot collisions are impossible by construction.

- Each `Pool` is an EIP-1167 clone with its own `PoolStorage` at slot 0. Cross-clone isolation is automatic.
- Singletons (`Admin`, `Staking`, `Distributor`, `Flash`) are key-by-`(pool, ...)`; their slot 0 holds the singleton-level state.
- The reference `Pool` impl follows an **append-only** rule on `PoolStorage`: existing fields' offsets and types are frozen across upgrades (new fields appended only).
- ERC-7201 namespaced storage is **no longer used**; the prior design dropped it once `delegatecall` was removed.

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

### 7.1. Reference-Impl Swap at PoolFactory

`Pool` clones are per-instance immutable. Upgrades happen by swapping the **reference impl** at `PoolFactory`:

```solidity
factory.requestReferenceUpgrade(newImpl);   // starts 7-day timelock
factory.executeReferenceUpgrade();          // applies after delay
factory.cancelReferenceUpgrade();           // optional cancel before exec
```

**Properties:**
- 7-day timelock (CRITICAL tier).
- No grace window — the upgrade must be explicitly cancelled if no longer desired; it does not silently expire (chosen so governance never accidentally void-cancels a queued upgrade).
- Only future clones use the new impl. Existing clones continue running their original `Pool` impl (per-clone immutability).
- The `PoolStorage` layout is append-only across reference-impl versions to keep future clones forward-compatible with off-chain integrations.

### 7.2. UUPS-Upgradeable Singletons

- `Treasury`, `Bridge`, and the shared `AccessControl` use UUPS.
- Upgrade authority gated through `AccessControl.owner()` with timelocks.
- `Bridge` adds a queue + cancel path for peer / config changes (`cancelSetPeer`, `cancelConfigChange`).

**Immutable / Replaced-Only Components:**
- All AIMM libraries (Pricing, PoolOracle, Spline, Maths, AnchorTree, etc.).
- `Admin`, `Staking`, `Distributor`, `Flash` singletons — replaced by re-deployment, not upgraded in place.
- `Pool` clones — non-upgradeable per-instance.

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

## 11. Audit Hardening Summary (Phases 42C + 42H)

This protocol has gone through two consolidated hardening sweeps. The verdicts below summarise what was closed and how — full per-finding write-ups live in the shared phase records (`docs/shared/16..33` for Phase 42C, `docs/shared/34..42` for Phase 42H).

### 11.1. Phase 42C — Audit Findings R1-R18

The third-party / internal audit produced 18 distinct findings. All are closed:

| Group | Findings | Theme | Resolution |
|---|---|---|---|
| **Pricing / Oracle** | R1, R2, R5 | TWAP same-block manipulation, EMA drift on idle pools, deviation-check ordering | Same-block rejection enforced before any state read; idle-pool EMA snap re-sync on first post-idle swap; deviation check moved before fee/skew application. |
| **Coverage / ALM** | R3, R7, R11 | Withdrawal-haircut rounding, decay-trigger off-by-one, liability-swap coverage check | Haircut math uses WAD precision then floors at the boundary; decay activates strictly when coverage `<` start ratio (was `<=`); liability swap re-checks both legs' coverage post-transfer. |
| **Fees** | R4, R9 | Protocol/LP split rounding loss, flash-fee under-charge on extreme amounts | Rounded-up protocol share; flash-fee math switched to mulDivUp. |
| **Routing** | R6, R10 | LCA cycle in adversarial anchor configs, hop-count under-count on degenerate trees | Anchor-tree mutation now revalidates no-cycle + depth invariants atomically; hop count counted from path length, not step list. |
| **Hooks** | R8, R12 | Hook bypass via flag bit-pack collision, missing post-hook on revert path | Hook flag packing widened to 32 bits with explicit per-stage masks; post-hooks gated by success of the wrapped op. |
| **Reentrancy / Flash** | R13, R14, R16 | Cross-asset reentrancy on flash-callback path, reserve-vs-ledger desync, campaign-id-zero ambiguity | Transient reentrancy guard upgraded to span all asset-touching paths; `flashAccount` writes ledger before `flashSend`; campaign ids start at 1 (lazy-bump). |
| **Access / Governance** | R15, R17, R18 | Owner-takeover via uninitialised AC ref, freeze bypass through stale module trust, treasury skim on update | Singleton `AccessControl` checked non-zero at every singleton constructor; freeze-bit honoured pre-route in every entry path; treasury collect requires explicit per-pool whitelist. |

**Outcome:** all 18 closed under the singleton refactor (Phase 42H) — no remaining open audit items.

### 11.2. Phase 42H Architectural Hardening

Phase 42H replaced the Diamond-lite proxy + ERC-7201 module pattern with standalone singletons + EIP-1167 `Pool` clones. The hardening rounds layered on top of that refactor:

- **G18 — UUPS transient-flag pattern.** `Treasury` / `Bridge` / `AccessControl` upgrade authorisation flag is held in EIP-1153 transient storage and asserted/cleared per upgrade call. Removes any window where a stale "upgrade pending" flag could be exploited by an unrelated state-changing path.
- **G19 — Singleton AccessControl wiring.** Every singleton (`Admin`, `Staking`, `Distributor`, `Flash`, `PoolFactory`) holds an `immutable AC` set at constructor and reverts on zero address. Eliminates the class of bugs where a singleton could be deployed pointing at the wrong owner.
- **G20 — Distributor leaf domain separation.** Merkle leaves embed a static `BTR_DISTRIBUTOR_v1` tag plus `(pool, campaignId)` so proofs cannot be replayed across pools, campaigns, or protocol versions.
- **G21 — Bridge cancel paths.** `Bridge.cancelConfigChange` and `Bridge.cancelSetPeer` were added so an in-flight queued upgrade can be explicitly retired without waiting for an indefinite grace window (and without ever silently expiring).

Soft residuals are tracked separately:
- **G6** — divergent Timelock encodings across 4 callsites; see [ADR-002 Timelock Encodings](/docs/shared/43-ADR-002-Timelock-Encodings) for the decision record.
- **G17** — alm `Vault` async withdraw queue (ERC-7540) is exposed through the generic `ERC7540_ABI` in `sdk/src/eth/erc7540.ts`; no per-contract ABI regen is required.

---

## 12. Related Documentation

- [Deployment & Upgrades](/docs/3.2-Deployment-&-Upgrades) — Upgrade procedures and deployment
- [Access Control](/docs/3.3-Access-Control) — Roles and permissions
- [Flow Guards](/docs/3.4-Flow-Guards) — JIT protection, cooldowns
- [Oracles](/docs/3.5-Oracles) — Oracle security and fallbacks
- [Admin](/docs/1.2.3-Admin) — Administrative functions
