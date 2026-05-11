# Governance Overview

> BTR tokenomics, staking, and reward distribution

---

## 1. Token Structure

### 1.1. BTR (Governance Token)

| Property | Value |
|----------|-------|
| **Role** | Protocol governance and reward boosting |
| **Supply** | Fixed maximum (enforced by Treasury) |
| **Minting** | Controlled by Treasury module |

### 1.2. sBTR (Staked BTR)

Receipt token for staked BTR.

| Property | Description |
|----------|-------------|
| **Voting Power** | Used for governance proposals |
| **Reward Boosting** | Required to earn full LP staking rewards |
| **Cooldown** | Transfers trigger cooldown period (default 2 weeks) |
| **Delegation** | Balance delegated to [Staking](/docs/1.2.4-Staking) |

The cooldown prevents vote buying and reward renting -voting and earning power are disabled during the cooldown period.

### 1.3. sLP (Staked LP)

Receipt token for staked LP positions.

| Property | Description |
|----------|-------------|
| **Deployment** | CREATE2 by [Staking](/docs/1.2.4-Staking) for deterministic addresses |
| **Rewards** | Eligible for BTR and other token rewards |
| **Composability** | Can be used as collateral or in other DeFi protocols |

---

## 2. Staking & Distribution

### 2.1. Claim Power (Reward Matching)

Rewards are distributed based on **Effective Stake**, not raw LP stake:

`"Effective Stake" = "Raw Stake" × "Claim Power"`

Claim Power depends on **sBTR Coverage**:

`"Coverage" = ("sBTR Value" × "Matching Ratio") / "sLP Value"`

| Coverage | Claim Power |
|----------|-------------|
| ≥ 100% | 100% (full rewards) |
| < 100% | Interpolated between Min Claim Power and 100% |

This mechanism forces LPs to become stakeholders (acquire BTR) to maximize yield.

### 2.2. Distributor

The [Distributor](/docs/1.2.5-Distributor) module handles reward distribution:

- **Accumulator Pattern** -O(1) complexity per claim (like Aave/Sushi)
- **Multi-token support** -Multiple reward tokens per pool
- **Campaign-based** -Configurable reward periods and rates

---

## 3. Related Documentation

- [BTR Token](/docs/2.1-BTR-Token) -Token mechanics and distribution
- [DAO Voting](/docs/2.2-DAO-Voting) -Voting mechanics and proposals
- [DAO Treasury](/docs/2.3-DAO-Treasury) -Treasury management
- [Liquid Staking](/docs/2.4-Liquid-Staking) -Staking mechanics
- [Emission Control](/docs/2.5-Emission-Control) -Emission schedule
