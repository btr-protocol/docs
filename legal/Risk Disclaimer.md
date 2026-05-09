### BTR RISK DISCLAIMER
*Last Updated: January 22, 2026*

#### 1. DEFINITION

At BTR ("BTR", "we", "us", "our"), we prioritize risk transparency. This disclaimer applies to all services provided through BTR's decentralized exchange protocol and front-end interfaces (collectively "Services"):

#### 1.1. BTR DEX Protocol
- **AIMM Protocol**: Balanced Automated Market Maker - a multi-asset DEX with hub-and-spoke routing, dynamic fees, and coverage ratio-based active liquidity management
- **Web Interface** ([https://btr.supply](https://btr.supply)): Front-end for interacting with the AIMM protocol and BTR ALM vaults
- **Smart Contracts**: Open-source upgradeable smart contracts using Diamond proxy pattern (DEX) and immutable / beacon-style adapters (ALM), deployed on Ethereum and compatible EVM chains

#### 1.1bis. BTR ALM (Active Liquidity Management) Protocol
- **Vault**: Hybrid ERC-4626 / ERC-7540 vault. Synchronous deposit, synchronous-fallback redeem, asynchronous redeem-ticket cohort processing. Up to eight adapters per vault.
- **Dex Adapter**: Concentrated-liquidity wrapper over a single third-party pool (UniV3 / UniV4 / PancakeV3 / PancakeInfinity / Algebra / Ramses). Multi-holder share-weighted via internal `principalOf` accounting.
- **Swapper**: Chain-singleton router for atomic swaps (LiFi) and ERC-7683 intents. Used by adapters for rebalancing and by vaults for salvage.
- **PriceProvider**: Chainlink USD-feed normalizer with L2 sequencer guard, 6-hour timelock on feed rotation, and 24-hour default heartbeat per feed.
- **AccessControl**: Single source of truth for owner / treasury / swapper / factory / keeper / adapter authority. 2-step ownership handover, timelocked rotations, daily kill cluster cap.

#### 1.2. Core Protocol Activities

The BTR protocol enables the following activities:
- **Token swaps**: Trading between supported digital assets via the AIMM DEX
- **AIMM liquidity provision**: Depositing tokens into AIMM coverage-ratio pools to earn trading fees and incentives
- **ALM vault deposits**: Depositing assets into ERC-4626 / ERC-7540 hybrid vaults that allocate across third-party concentrated-liquidity pools
- **Flash loans**: Uncollateralized borrowing for single-transaction use cases (ERC-3156 compliant)
- **Staking**: Locking LP tokens (sLP) or governance tokens (sBTR) to earn rewards and voting rights
- **Governance**: Participating in protocol decisions via BTR token voting (forward-looking)
- **Cross-chain intents**: Initiating ERC-7683 intent-based swaps via the Swapper singleton (forward-looking; multi-chain ALM pending)

You should carefully assess whether using BTR's services matches your risk tolerance and financial capacity for potential losses.

---

### 2. PROTOCOL RISKS

#### 2.1. Smart Contract Risks

The protocol relies on smart contracts that are **open-source** and available for public inspection on GitHub. While the code undergoes internal testing, **no third-party security audit has been completed**. Undiscovered vulnerabilities could result in partial or total loss of deposited funds.

**Diamond Proxy Architecture**: The protocol uses a Diamond proxy pattern (ERC-2535), which allows modular upgrades through facet replacement. Unlike traditional upgrade patterns, Diamond upgrades enable granular changes to specific functionality. Upgrades require a 3-7 day timelock but introduce centralization risk:
- **Module upgrade control**: Protocol owners can add, replace, or remove Diamond facets
- **Upgrade backdoor**: Even with timelocks, centralized upgrade authority could potentially bypass community consensus
- **Storage collision risks**: Diamond patterns require careful ERC-7201 slot management; collisions could corrupt data

For details on upgradeability and governance processes, see [Protocol Architecture](/specs/PROTOCOL.md).

**Reentrancy Protection**: Critical modules implement ReentrancyGuard patterns, but external calls to other contracts could enable reentrancy attacks if third-party contracts are malicious.

**Open-Source Status**: All smart contracts are open-source and available for third-party security review. Users and security researchers are encouraged to audit the code. However, absence of third-party audit findings does not guarantee absence of vulnerabilities.

#### 2.2. Market and Financial Risks

Digital assets are highly volatile. Token prices can fluctuate dramatically due to market conditions, adoption, speculation, regulatory changes, or technical factors. Trading and providing liquidity can result in losses exceeding your initial deposit. You should only use funds you can afford to lose entirely.

BTR does not provide investment, legal, or tax advice. You are solely responsible for evaluating the suitability of these services for your circumstances and should consult qualified advisors as needed.

#### 2.3. Liquidity Provider Risks

**Coverage Ratio Haircuts**: The protocol tracks coverage ratio (reserves/liabilities) per asset. When coverage falls below 1.0, withdrawals receive proportionally reduced amounts based on the **power-law formula**:

```
haircut (h) = (1 - coverage)^η
```

Where `η` (eta) is a power-law parameter that determines the severity of haircuts. As coverage approaches 0%, haircuts approach 100%, meaning LPs could face **total loss** of their claims. LPs share losses pro-rata without additional penalties. For details, see [Coverage Ratio Documentation](/specs/ALM_AND_COVERAGE.md).

**Undercollateralization Risk**: If an asset's coverage ratio reaches 0% due to depletion, LPs in that pool lose 100% of their deposits. No insurance or backstop exists for undercollateralized pools.

**Liability Time Decay**: Assets with prolonged underwater coverage may undergo linear liability reduction to eliminate bad debt. This socialized loss mechanism gradually reduces LP claim values over time to ensure long-term pool sustainability.

**Index Manipulation**: Donations to liquidity pools increase the pool index disproportionately, effectively diluting existing LPs. While this is intended to reward capital providers, it can be abused.

**Insufficient Liquidity**: Swaps and withdrawals are blocked if reserves fall below `minLiquidity` thresholds, temporarily trapping funds until conditions improve or new deposits arrive.

**Dynamic Fees**: Swap fees adjust based on coverage ratio, volatility, and price divergence. High fees during imbalances may reduce trading volume and LP earnings.

**Impermanent Divergence**: Unlike traditional AMMs, AIMM uses single-sided liquidity (deposit USDC, withdraw USDC). While this eliminates classic impermanent loss, coverage ratio fluctuations can cause similar effects if reserves deplete via swaps.

**Flow Guard Bypass Prevention**: Deposits have a 15-second cooldown before withdrawal can be initiated to prevent Just-In-Time (JIT) manipulation attempts. LPs should understand this delay when planning entry/exit strategies.

#### 2.4. Pricing Mechanics Risks

The AIMM protocol uses sophisticated pricing mechanisms that carry specific risks:

**Extreme Volatility Spikes**: The protocol maintains fast and slow Exponential Moving Averages (EMAs) for volatility metrics. During extreme market events, these EMAs can spike to **10,000%** (100×), causing massive spreads and temporarily making swaps prohibitively expensive. This is an intentional protective measure but can cause temporary illiquidity.

**Spline Manipulation**: Piecewise bonding curves are defined by spline parameters controllable by the protocol owner. If set adversarially, these could create unfavorable pricing profiles (e.g., excessive spreads, asymmetric price impact). Owners are expected to act in protocol interest, but centralization risk exists.

**Decimal Overflow**: Prices are encoded in B64 format which has limits for high-price tokens (e.g., BTC at $50,000+). The protocol mitigates this by using 18-decimal accumulators, but edge cases where tokens appreciate dramatically could cause encoding issues.

**No Automated Circuit Breaker**: Circuit breakers are **manual** and require owner or governance action. There is no automated pause on extreme price divergence. If operators are unavailable during crisis, manual intervention may be delayed.

**Pricing Error Propagation**: Incorrect oracle inputs (TWAP divergence) can lead to incorrect mid-price calculations, causing adverse trade execution for users.

#### 2.5. Oracle Architecture Risks

The protocol uses **internal oracles** based on Time-Weighted Average Price (TWAP) with specific vulnerabilities:

**TWAP Manipulation**: Large swaps can shift the fast and slow EMA windows, distorting price inputs. While this requires significant capital, sophisticated attackers could manipulate oracles to:
- Trigger false circuit breaker freezes
- Manipulate dynamic fee calculations
- Distort volatility metrics

**Oracle Staleness**: If no swaps occur for **7+ days**, fast and slow TWAPs diverge from market prices. During low-liquidity periods, oracles may become stale and outdated:
- Fast TWAP (short window) reflects no new data
- Slow TWAP (long window) lags significantly behind market
- Price-dependent operations use stale data

**Keeper Bot Dependence**: The protocol relies on keeper bots to update oracles and maintain TWAP accuracy. If keeper services go offline:
- Oracles only update when swaps occur
- Stale oracle periods extend beyond 7 days during low activity
- Volatility calculations become unreliable
- Dynamic fees may use incorrect inputs

**B64 Overflow for High-Price Tokens**: High-value tokens (e.g., BTC, ETH at high prices) require careful accumulator decimal handling (18 decimals) to avoid B64 encoding overflow. Configuration errors could cause pricing malfunctions.

**No Oracle Circuit Breaker**: There is no automated mechanism to pause operations when internal TWAPs diverge excessively from external market prices. Manual intervention is required.

**External Oracle Reliance**: If the protocol integrates external oracles (Chainlink, Pyth, etc.) as fallbacks, attacks on those external oracle networks propagate to BTR. The protocol currently relies primarily on internal TWAPs but may use external sources.

For technical details, see [Oracle Architecture](/specs/ORACLE.md).

#### 2.6. Slippage and Execution Risks

Large trades may experience significant price impact due to piecewise bonding curves. Actual execution prices may differ from quoted prices if market conditions change between quote and execution. Front-running or MEV (Maximal Extractable Value) by other users or validators may disadvantage your transactions.

**No Native MEV Protection**: The protocol does not implement Flashbots Protect, FPI (Flashbots Protect Interface), or batch auctions. All transactions are visible in the mempool and vulnerable to:
- Front-running
- Sandwich attacks
- Back-running
- Arbitrage

Users are responsible for setting appropriate slippage tolerances and understanding that execution quality is not guaranteed.

#### 2.7. Staking and Governance Risks

**21-Day Unlock Period**: Staked tokens (sLP, sBTR) have a mandatory 21-day unlock period. During this period:
- Tokens earn no rewards
- Tokens cannot be transferred or sold (soulbound)
- Tokens are locked even if protocol suffers catastrophic failure or exploit

If the protocol fails or is exploited during the 21-day unlock window, staked funds are **not recoverable** until the unlock period expires, after which assets may have zero value.

**BTR Token Minting**: The Treasury contract can mint unlimited BTR tokens. While intended for protocol governance and rewards, this creates **dilution risk**:
- Governance could approve excessive minting
- Large mint events dilute existing BTR holders
- No hard cap on BTR supply is enforced by code

**Soulbound Limitations**: sLP and sBTR tokens are soulbound—they cannot be sold, transferred, or traded. Users must unstake and wait 21 days to access liquidity, creating opportunity costs and trapping funds during emergencies.

**Voting Centralization**: BTR governance uses token-weighted voting. Large BTR holders (whales, Treasury, early adopters) can unilaterally pass proposals, including those that may be detrimental to smaller users. Governance power is proportional to BTR holdings, not user count.

**Unstake Blocking**: If the StakingV1 contract is paused (via circuit breaker or emergency action), users cannot unstake. Funds remain locked indefinitely until the contract is unpaused, which requires owner or governance action.

**Opportunity Cost**: Staking involves locking capital for extended periods. During volatile market conditions, opportunity costs (missed gains from other investments) may exceed staking rewards.

#### 2.8. Flash Loan Risks

Flash loans are uncollateralized and must be repaid within a single transaction. Failure to repay causes the entire transaction to revert, potentially resulting in gas fee losses.

**Oracle Manipulation via Flash Loans**: Sophisticated attackers can combine flash loans with swaps to manipulate internal TWAP oracles:
1. Borrow large amounts via flash loan
2. Execute large swap to distort fast/slow EMA windows
3. Use distorted oracle data to exploit other protocol operations
4. Repay flash loan within same transaction

While this requires deep technical knowledge and capital, the absence of oracle-specific flash loan protection creates vulnerability.

**Liquidity Drain**: Rapid flash loans could drain reserves below `minLiquidity` thresholds, blocking normal swaps and withdrawals for other users. If multiple pools are affected simultaneously, protocol-wide illiquidity could occur.

**Fee Underpricing**: The `flashFeeBps` parameter controls flash loan fees. If set too low by governance, the protocol may lose money on flash loans (net negative after gas costs for keeper operations).

**Flash Loan Attacks**: Borrowers may use flash loans to orchestrate complex attacks involving multiple protocols or steps to drain liquidity. The protocol has no dedicated flash loan mitigation beyond standard reentrancy guards.

#### 2.9. Circuit Breaker Risks

The protocol implements **manual** circuit breakers to halt trading when:
1. **Reserve price floor breach**: Swap price falls below configured minimum
2. **Deviation freeze**: Fast/slow TWAP ratio divergence exceeds thresholds (detects depegs, exploits)

**Manual Trigger Only**: Circuit breakers are **not automated**. They require:
- Owner action via multisig
- Governance proposal and execution
- Keeper bot monitoring and trigger

If operators are unavailable during a crisis, circuit breakers may not trigger in time to prevent losses.

**False Positives**: Circuit breakers may trigger false positives during extreme volatility, temporarily freezing trading even when markets are functioning normally. Frozen assets cannot be swapped until conditions normalize or governance intervenes.

**Extended Freezes**: Once triggered, circuit breakers may remain active for extended periods if governance is slow to respond or if uncertainty about market conditions persists. Users cannot withdraw or swap frozen assets.

#### 2.10. Counterparty and Custody Risks

BTR does not act as a counterparty, broker, or custodian. All transactions occur peer-to-contract. You retain custody of your wallet and keys—lost or compromised keys result in permanent, irreversible loss of funds. Transactions on blockchains are final and irreversible.

---

### 3. BRIDGING AND MULTI-CHAIN RISKS

#### 3.1. Bridge Architecture Risks

BTR supports cross-chain transfers via LayerZero messaging protocol. This introduces additional risks:

**Bridge Failure**: If the LayerZero bridge fails:
- Tokens sent across chains are burned on source chain
- Tokens may never arrive on destination chain
- Recovery requires manual intervention by bridge operators
- No automatic refund mechanism exists

**Bridge operators (relayer, oracle, or owner)** must manually process recovery, which may be delayed or denied depending on circumstances.

**Rate Limit Exhaustion**: Daily rate limits on bridge transfers can block cross-chain transactions. If rate limits are exhausted (e.g., during high-volume events), users must wait for the limit to reset (typically 24 hours) before transferring funds.

**Bridge Upgradeability**: The bridge contract is UUPS upgradeable with a 7-day timelock. Upgrades could theoretically introduce vulnerabilities, backdoors, or malicious behavior. While intended for security improvements, centralized upgrade control creates risk.

**Peer Misconfiguration**: The bridge requires correct peer addresses for each supported chain. If misconfigured:
- Messages are lost (tokens burned, never received)
- Cross-chain functionality fails
- Recovery may be impossible without redeployment

**LayerZero Dependency**: BTR relies on LayerZero's decentralized oracle and relayer network. If LayerZero's infrastructure fails, suffers a 51% attack, or is otherwise compromised, BTR's bridge functionality is impacted.

#### 3.2. Multi-Chain Liquidity Fragmentation

When tokens are bridged across chains, liquidity becomes fragmented:
- Price discrepancies between chains create arbitrage opportunities
- Low liquidity on destination chains increases slippage
- Cross-chain arbitrage may not be profitable due to bridge fees and delays
- Liquidity depth varies significantly per chain

Users should understand that providing liquidity on multiple chains exposes them to risks on each chain independently.

---

### 4. SECURITY AND MALICIOUS ACTOR RISKS

#### 4.1. Protocol Upgrade Risks

The AIMM protocol uses upgradeable smart contracts via Diamond proxy pattern (ERC-2535). Protocol upgrades, modifications, or parameter changes may be executed by governance, contract owners, or keeper bots to:
- Fix bugs or security vulnerabilities
- Add new features or improve protocol functionality
- Respond to changing market conditions or security incidents
- Adjust pricing algorithms, coverage ratios, or other protocol parameters
- Upgrade oracle configurations or circuit breaker settings
- Replace or add Diamond facets

**Diamond Proxy Specific Risks**:
- **Facet replacement**: Individual functions can be upgraded without redeploying entire contract
- **Storage layout changes**: Incorrect facet upgrades can corrupt existing state
- **Facet selection attacks**: Malicious facets could be added if upgrade authority is compromised
- **Hidden backdoors**: Granular upgrades enable subtle changes difficult to detect

**Timelock Protections**: Upgrades require a 3-7 day timelock, providing window for community review. However, during emergencies, governance could accelerate upgrades or use emergency controls.

**Centralized Upgrade Authority**: While intended to be decentralized via governance, current upgrade permissions may vest in multisig owners or DAO contracts. Centralization creates risks of:
- Unilateral upgrades without community approval
- Delayed or rejected necessary upgrades
- Capture by malicious actors

You are solely responsible for reviewing upgrade proposals and their potential impact before participating in governance decisions or continuing to use the protocol.

#### 4.2. Malicious Attack Risks

DeFi protocols face ongoing threats from malicious actors seeking to exploit vulnerabilities. Specific risks include:

**Smart Contract Exploits**: Attackers may discover and exploit undiscovered vulnerabilities in smart contract code, potentially resulting in partial or total loss of protocol funds. Common exploit vectors include:
- Reentrancy attacks
- Integer overflow/underflow
- Logic errors in complex pricing or coverage calculations
- Signature replay attacks

**Oracle Manipulation**: Despite internal oracle protections, sophisticated attackers may manipulate price feeds by:
- Executing large swaps to distort TWAPs
- Flash loan + swap combinations
- Timing attacks during keeper oracle updates
- Exploiting stale oracle periods during low activity

**Flash Loan Attacks**: Borrowers may use flash loans to orchestrate complex attacks involving multiple protocols or steps to drain liquidity, manipulate oracles, or exploit arithmetic relationships.

**MEV Extraction**: Validators or searchers may front-run, sandwich, or otherwise profit from your transactions, reducing your execution quality. The protocol has no native MEV protection.

**Governance Attacks**: Malicious actors may attempt to gain control of governance tokens or voting power to pass proposals detrimental to protocol users, including:
- Excessive token minting (dilution)
- Fee parameter manipulation
- Unauthorized protocol upgrades
- Drain of treasury funds

BTR employs multiple security measures including internal testing, circuit breakers, and monitoring, but no system can be 100% secure. **No third-party security audit has been completed.** You acknowledge these inherent risks.

#### 4.3. Cryptographic and Quantum Computing Risks

The protocol relies on widely-accepted cryptographic standards (ECC, SHA-256, etc.) currently considered secure. However, you acknowledge that:
- Future advances in computing technology, including quantum computing, could potentially compromise current cryptographic standards
- Such advances could theoretically enable private key extraction or transaction forgeries
- The protocol may require upgrades or migrations to quantum-resistant cryptography if such threats materialize

While these risks are currently theoretical and distant, they represent a long-term uncertainty in digital asset security.

---

### 5. OPERATIONAL RISKS

#### 5.1. Frontend and Interface Risks

The web interface at [https://btr.supply](https://btr.supply) is a convenience layer—you can interact with smart contracts directly if the interface is unavailable.

**Frontend Downtime**: If the web interface goes offline due to:
- Server failures
- DDoS attacks
- DNS issues
- Malicious compromise

Users must interact directly with smart contracts using alternative interfaces (e.g., Etherscan, Tenderly, custom dApps) or CLI tools. This requires technical knowledge and increases risk of errors.

**Phishing and Malicious Interfaces**: Always verify URLs and contract addresses. Malicious sites mimicking BTR's interface can:
- Steal private keys
- Redirect transactions to attacker contracts
- Display fake balances and states
- Trick users into approving malicious spenders

**Client-Side Errors**: Browser extensions, wallet issues, or network connectivity problems can cause transaction failures, gas losses, or incorrect state display.

#### 5.2. Keeper Bot Failures

The protocol relies on keeper bots to perform critical maintenance operations:
- Oracle updates (TWAP calculations)
- Fee collection and redistribution
- Liquidity rebalancing
- Coverage ratio monitoring

If keeper services go offline:
- **Oracles become stale**: TWAPs only update when swaps occur
- **Fees uncollected**: Protocol revenue may be lost
- **Manual intervention required**: Protocol operations degrade
- **Extended delays**: If no swaps occur, oracles may diverge from market for weeks

Keeper reliability is critical but not guaranteed. Users should understand that keeper failures degrade protocol functionality without necessarily causing catastrophic loss.

#### 5.3. Oracle Stoppage

If oracles stop updating (due to no swaps + no keepers):
- TWAPs diverge significantly from market prices
- Dynamic fee calculations use stale volatility data
- Circuit breakers may misfire (false positives or missed triggers)
- Liquidity shaping becomes suboptimal

Recovery requires manual oracle reset or governance intervention, which may be delayed.

#### 5.4. Blockchain Congestion

Periods of high network activity (e.g., NFT mints, DeFi events) can cause:
- **Increased gas fees**: Swaps and deposits become uneconomic
- **Failed transactions**: Insufficient gas limits cause reverts and lost fees
- **Delayed confirmations**: Mempool congestion slows execution
- **Slippage degradation**: Long confirmation windows increase price impact

BTR is not responsible for losses incurred due to network congestion. Users should adjust gas prices and slippage tolerances accordingly.

---

### 6. USER ACTION RISKS

#### 6.1. Contract Address Verification

Users must verify they are interacting with genuine BTR contracts:
- Sending tokens to malicious or fake contracts results in **irreversible loss**
- Contract addresses can be found in official documentation and GitHub repositories
- Always cross-reference addresses from multiple trusted sources
- Be wary of unofficial "forks" or clones claiming to be BTR

#### 6.2. Slippage and Price Impact

Large trades suffer significant price impact due to liquidity depth limits:
- Setting slippage tolerance too low = failed transactions
- Setting slippage tolerance too high = accepting unfavorable prices
- Unexpected price movements between quote and execution = unfavorable fills

Users should understand their trade size relative to pool depth and set appropriate slippage tolerances.

#### 6.3. Front-Running and MEV

All transactions are visible in the mempool before confirmation, exposing users to:
- **Front-running**: Bots see your transaction and execute similar transactions ahead of yours
- **Sandwich attacks**: Bots place orders before and after your transaction to profit from price impact
- **Back-running**: Bots execute transactions after yours to profit from state changes
- **Arbitrage**: Bots extract value from price inefficiencies your transaction reveals

The protocol has no MEV protection. Users accept the risk of inferior execution quality.

#### 6.4. Gas Failures

Transactions can fail for various reasons:
- **Insufficient gas limit**: Transaction runs out of gas mid-execution, all work rolled back, gas fee lost
- **Gas price too low**: Transaction stays stuck in mempool, eventually drops or becomes outdated
- **Revert conditions**: Smart contract logic fails (e.g., slippage exceeded), all gas spent is lost
- **Nonce issues**: Transaction ordering issues cause reverts

Users are responsible for setting appropriate gas parameters and understanding that failed transactions result in gas fee losses with no value transferred.

#### 6.5. Approval and Allowance Risks

Users must approve contracts to spend tokens:
- **Infinite approvals**: Approving unlimited spend increases risk if contract is compromised
- **Revocation difficulties**: Some approval patterns are difficult to revoke
- **Multiple approvals**: Users may accidentally approve malicious contracts

Best practice: approve only the amount needed, or use permit-based approvals when available.

---

### 7. THIRD-PARTY AND EXTERNAL RISKS

#### 7.1. Regulatory and Compliance Risks

Digital asset regulations vary significantly by jurisdiction and are rapidly evolving. You are solely responsible for ensuring your use of BTR services complies with all applicable laws and regulations in your jurisdiction. BTR does not provide legal advice. Regulatory actions against digital assets, DeFi protocols, or specific tokens may render the protocol illegal or unusable in your region. Citizenship and geographic restrictions are detailed in [our Terms of Service](/terms-of-service).

#### 7.2. Blockchain Network Risks

The protocol operates on Ethereum and other EVM-compatible blockchains. You acknowledge and accept the following risks associated with blockchain infrastructure:

- **Network Forks**: Hard forks, soft forks, or chain reorganizations may occur on the underlying blockchain networks. Such events may result in transaction disputes, blockchain splits, or loss of funds. BTR has no control over network forks and cannot guarantee how the protocol will behave during or after a fork event.

- **Protocol Rules Changes**: The underlying blockchain networks may change their rules, consensus mechanisms, or fee structures through governance or upgrades. Such changes may impact the usability, cost, or functionality of the Services.

- **Network Congestion**: Periods of high network activity may cause transaction delays, failed transactions, or increased gas fees. BTR is not responsible for losses incurred due to network congestion.

- **Network Outages**: The underlying blockchain networks may experience temporary outages, downtime, or degradation of service. During such periods, you may be unable to interact with the protocol.

- **51% Attacks**: Theoretical attacks on blockchain consensus mechanisms could result in transaction reversals, double-spending, or other security compromises. While rare, you acknowledge this risk.

- **Irreversibility**: Blockchain transactions are irreversible—errors cannot be corrected. BTR cannot reverse or refund any transaction once it has been confirmed on-chain.

#### 7.3. External Oracle Risks (Optional Integrations)

While the protocol primarily relies on internal oracles, external oracle integrations (e.g., Chainlink, Pyth) may be used as fallback price sources. External oracles carry risks including:
- Data feed failures or staleness
- Oracle network attacks or manipulation
- Latency in price updates
- Incorrect data from off-chain sources

For external oracle terms and disclosures:
- Chainlink: https://chain.link/terms
- Pyth Network: https://pyth.network/disclaimer

#### 7.4. Token-Specific Risks

Tokens listed on the protocol may have unique risks:
- **Fee-on-transfer tokens**: May cause reserve/liability tracking discrepancies
- **Rebasing tokens**: Supply changes may affect accounting
- **Upgradeable tokens**: Token contract changes could break integrations
- **Low liquidity tokens**: Higher slippage and potential for manipulation
- **Experimental tokens**: Unproven security, high volatility, potential for total loss
- **Blacklisted tokens**: Contracts may block addresses (including BTR protocol contracts)

The protocol does not endorse or guarantee the safety of any listed token. Users should research token projects before providing liquidity.

---

### 7.5. Third-Party DEX Pool Risks (ALM-Specific)

ALM Dex adapters wrap external concentrated-liquidity pools whose code, governance, and operational state are entirely outside BTR's control. Each integrated DEX family carries distinct risk characteristics that you accept by depositing into vaults that allocate to that family:

- **Uniswap v3**: Battle-tested but historically vulnerable to fee-tier mispricing and tick-range exit drag during volatile transitions. Fee-tier and tick-spacing immutability per pool — vault adapter cannot adjust these post-deployment without reallocation.
- **Uniswap v4**: Hooks introduce arbitrary on-chain code paths during pool operations. Hook bugs, malicious hook upgrades (where applicable), or hook-induced reverts can degrade or freeze vault adapters allocated to v4 pools. The v4 StateView dependency (`sv` in code) introduces an additional indirection — staleness or misconfiguration of StateView could mis-mark NAV.
- **PancakeSwap V3 / Infinity**: Inherit Uniswap v3 / v4 risk classes plus PancakeSwap-specific governance and operator decisions.
- **Aerodrome Slipstream**: Inherits ve(3,3) governance dynamics; fee tier and emission incentives depend on vote outcomes outside BTR control.
- **Algebra / Ramses / others**: Each integration carries the integrating DEX's specific risk profile — adaptive fees, hook behavior, oracle integration, and governance.

#### Concentrated-Liquidity Specific Risks

- **Impermanent Loss (IL)**: Concentrated-liquidity LPs face amplified IL relative to full-range LPs. As underlying prices move, the in-range token mix rotates; under sustained drift the position can be entirely converted into the underperforming asset.
- **Loss-Versus-Rebalancing (LVR)**: Arbitrageurs systematically extract value from LP positions by trading against stale on-chain prices when external markets move. Vault NAV's pessimistic worst-of(pool, oracle) mark partially mitigates *measurement* of LVR but does not eliminate the underlying *economic* loss.
- **Tick-Range Exit Drag**: When a position is fully out-of-range, withdrawing requires either rebalancing through the Swapper (incurring slippage and fees) or accepting an asymmetric token mix on redemption. The configurable rebalance `interval` (default 1 hour, max 30 days, keeper-tightenable by 1 day per ratchet) bounds how quickly drag can be remediated.
- **Fee-Tier Mismatch**: A vault adapter configured against an inappropriate fee tier may capture insufficient swap fees to compensate for IL / LVR. Tier selection is a curation decision; review live tier configuration before depositing.

#### Defensive Parameters and Circuit Breakers (ALM)

Each Dex adapter holds the following defensive parameters, each independently bounded in code:
- `deviationBps` (10–5000 bps): max accepted divergence between pool tick price and oracle price before `assetValue` triggers worst-of pessimism penalties; default 100 bps.
- `slipBps` (10–5000 bps): max accepted swap slippage on rebalance; default 100 bps.
- `heartbeatDriftBps` (≤5000 bps): scales the exit-fee floor with oracle staleness; default 50 bps.
- `interval` (≤30 days): rebalance cadence; default 1 hour.
- `KILL_RATE_LIMIT` (1 hour): per-adapter minimum between keeper-initiated kills.
- `FREEZE_TIMELOCK` (1 day): minimum duration before a frozen adapter can be unkilled.
- Cluster-wide `KEEPER_KILL_DAILY_CAP` (2): max keeper-initiated adapter kills per UTC day across all adapters; prevents 1-keeper × 8-adapter cascade freezes.

You acknowledge that:
- Defensive ratchets (depositCap↓, slip↓, interval↑, kill) are intended to *protect* holders during anomalies but may also *trap* funds during transient volatility.
- A killed adapter holds funds inert until owner action (no per-adapter automated unkill).
- Manual intervention is required to recover from edge cases the automated rules do not handle.

### 7.6. ALM Vault Accounting and NAV Risks

- **Pessimistic worst-of NAV**: Share price is computed from the worst of pool-derived and oracle-derived marks per adapter, summed across the vault. This may understate NAV during normal operation; depositors entering at a pessimistic mark and exiting at a pessimistic mark do not lose to the asymmetry, but mid-stream events (rebalances, kills, oracle updates) can produce non-monotone share prices.
- **HWM perf-fee crystallization**: Performance fees crystallize only when both (assets, supply) jointly exceed the previous high-water-mark pair. This protects against double-charging on volatility but means that early depositors carry the unrealized NAV erosion of late depositors when prices recover. The `hwmInit` flag explicitly decouples first-init from numeric state to prevent first-deposit edge cases.
- **Exit-fee floor scales with adapter risk**: `exitPip ≥ 1.2 × max_i(deviationBps + heartbeatDriftBps) × 10`. Exiting a vault during a high-deviation / high-drift episode incurs a higher exit fee than exiting during normal operation. Exit fees are not refundable.
- **Async-redeem cohort haircut**: When a vault processes an asynchronous-redeem cohort, a haircut may be applied if the vault is short of liquid assets. The haircut is broadcast in the `Processed(cohort, haircutBps)` event. Pending tickets bear the cohort's haircut at claim time.
- **Multi-holder Dex adapter externalities**: Adapters are shared across multiple vaults / EOAs / multisigs. Rebalance timing is a shared resource; one holder's deposit / withdraw can shift the timing window observed by another. Adapters charge keeper-kill counts cluster-wide.
- **Salvage operations**: Tokens accidentally sent to or accumulated by the vault outside the asset whitelist may be salvaged by the keeper into the vault's asset; salvage routes through the Swapper and incurs slippage. Salvageable accounting is intentionally narrow to prevent abuse.

### 7.7. Swapper, Intent, and Atomic-Swap Risks (ALM-Specific)

ALM rebalances and salvages flow through the chain-singleton `Swapper`. You accept the following risks:

- **Pinned router / settler rotation**: The LiFi router and ERC-7683 settler are pinned addresses, rotated under a 7-day timelock. During rotation, in-flight intents drain via `forceRefund` and `claimLateRefund`; this drain is not instantaneous and may temporarily lock late-refund credits in the Swapper.
- **Intent windows**: Intents have a TTL between 5 minutes and 24 hours. While an intent is active, the in-flight `amountIn` is held in Swapper escrow and the vault's `assetValue` may temporarily under-report.
- **Per-tokenOut serialization**: At most one active intent per `tokenOut` exists globally. This prevents settle-order contamination but caps intent throughput per token.
- **Settler counterparty risk**: An ERC-7683 settler that defaults, mis-fills, is sanctioned, or returns funds asynchronously can leave the vault in late-refund limbo. Refunds are claimable via `claimLateRefund` once the settler returns funds.
- **`minOut` enforcement only**: The Swapper enforces only the `minOut` constraint and balance-delta sweep. Slippage *authority* (the actual envelope) lives in the calling adapter's pre/post-swap pessimistic worst-of check. A misconfigured adapter `slipBps` weakens this envelope.
- **Third-party aggregator fan-out**: LiFi routes may transit Unizen, Squid, Rango, Socket, or other aggregators; each adds counterparty hops outside BTR's audit perimeter.
- **Owner rescue boundary**: `ownerRescue` may sweep balances *exceeding* active-intent obligations + outstanding lateRefund credits. While bounded, the rescue authority is centralized.

### 7.8. PriceProvider and L2 Sequencer Risks (ALM-Specific)

- **Chainlink dependency**: ALM oracle marks depend on Chainlink USD feeds. A failed, paused, or deprecated Chainlink feed reverts adapter NAV reads with `FeedStale` or `BadFeed`, freezing the vault until governance rotates the feed (instant first bind; 6-hour timelock on swap; 6-hour timelock on removal).
- **Feed-rotation timelock**: A compromised feed rotation cannot be patched faster than 6 hours.
- **L2 sequencer outage**: When deployed on an L2 with a sequencer feed, the PriceProvider reverts with `SeqDown` if the sequencer is reported down or within a 1-hour grace window post-recovery. During this window vault deposits, redemptions, and rebalances may all revert.
- **Heartbeat staleness**: Default 24-hour heartbeat. Per-feed `heartbeat` is configurable; an aggressively low heartbeat causes spurious `FeedStale` reverts during low-update periods, while an aggressively high heartbeat tolerates stale data.
- **Unconfigured tokens**: `getPrice(token)` returns 0 for unconfigured tokens; the calling adapter's worst-of math then degrades to pool-only marking, weakening pessimism guarantees.

### 7.9. AccessControl, Keeper, and Operator Risks (ALM-Specific)

- **Multisig owner authority**: AccessControl owner controls treasury, swapper rotation (7-day timelock), factory rotation (14-day timelock; bootstrap-permitted once before timelock), and adapter / keeper registry. Compromise or insider abuse of the owner key, while mitigated by timelocks, is an unavoidable centralization risk.
- **2-step handover only**: Solady's 1-step `transferOwnership` and `renounceOwnership` are explicitly disabled; ownership transfers via `requestOwnershipHandover` + `completeOwnershipHandover` (zero-target rejected to close back-door renounce). This prevents accidental key loss but does not prevent intentional transfer to a malicious party.
- **Keeper kill rate-limit + cluster cap**: Per-adapter `KILL_RATE_LIMIT` of 1 hour and cluster `KEEPER_KILL_DAILY_CAP` of 2 prevent runaway keeper activity but also bound how quickly a compromised keeper can be contained.
- **Keeper privileges**: Keepers can rebalance, ratchet defensive parameters, kill (rate-limited), settle / refund intents, and call `claimLateRefund`. A malicious keeper cannot exfiltrate funds directly but can grief by repeatedly killing adapters or by setting hostile defensive parameters within bounds.
- **Adapter deregistration**: Owner can deregister a permanently-killed or compromised adapter, blocking re-add. This is a one-way action.

### 7.10. Cross-Chain and Multi-Chain Allocation Risks (Forward-Looking)

BTR ALM may extend to cross-chain allocation in future releases. Forward-looking risks include:
- **Bridge solvency**: Cross-chain ALM positions depend on bridge solvency for principal repatriation.
- **Cross-chain message latency**: Allocation rebalance signals across chains carry latency proportional to bridge / messaging-layer finality.
- **Finality differences**: Reorg behavior on optimistic L2s vs. zk L2s vs. L1 differs; cross-chain accounting must absorb finality differentials.
- **Cross-chain governance key custody**: Authority over cross-chain allocation must be held in keys whose compromise would expose multiple chains simultaneously; the security perimeter of the weakest custody point dominates.
- **Sequencer outages on L2**: Already covered in §7.8; cross-chain amplifies single-L2 outages into multi-vault impact.

### 7.11. BTR Token and Governance Risks (Forward-Looking)

The BTR governance token is forward-looking. If issued, additional risks apply:
- **Regulatory classification risk**: BTR token treatment may vary by jurisdiction. Securities, commodity, derivative, or other classification can affect your ability to hold, transfer, or transact in BTR token.
- **Market risk**: Like any liquid token, BTR token will be subject to price volatility and may trade below intrinsic value.
- **Dilution risk**: Treasury minting authority (already noted in §2.7 for the existing protocol) extends to ALM governance distributions and incentive programs. Dilution can occur via approved minting actions.
- **Vesting schedules**: Contributor / advisor / treasury allocations may be subject to vesting; cliff and unlock events can produce predictable supply pressure.
- **Voting weight implications**: Token-weighted voting concentrates governance authority among large holders; small holders effectively delegate decisions to whales.
- **Treasury controls**: BTR-token-controlled treasury parameters (fee splits, allocation between products, emergency reserves) may affect ALM economics. Vote outcomes you do not participate in are nonetheless binding.

### 7.12. No Fiduciary, No Advice, No Guarantee (Reaffirmation Across All Activities)

For both AIMM DEX and ALM vault deposits, governance participation, intent submission, swap execution, and any other interaction with the Protocol:
- BTR is not your fiduciary, advisor, broker, asset manager, or investment company.
- Nothing on the Interface, in documentation, in marketing materials, in social posts, in community channels, or in keeper / contributor public communications constitutes financial, legal, tax, regulatory, accounting, or investment advice.
- No backtest, simulation, or historical performance — including yields displayed on the Interface — predicts future results.
- BTR makes no guarantee of vault performance, no guarantee of redemption-on-demand within any specific timeframe (subject to the vault's redemption mechanics), no guarantee of intent fill, and no guarantee of token price.

### 8. USER RESPONSIBILITIES AND ACKNOWLEDGMENT

By using BTR services, you acknowledge and accept:

1. **Full Responsibility**: You bear sole responsibility for all decisions and outcomes related to your use of the protocol. You are solely liable for any losses, whether financial or otherwise.

2. **No Guarantees**: BTR makes no warranties regarding accuracy, reliability, security, or performance of the protocol. Services are provided "as is" and "as available."

3. **Technical Competence**: You possess sufficient technical knowledge to understand blockchain systems, smart contracts, and DeFi risks. You have conducted your own research and evaluation.

4. **No Recourse**: BTR shall not be liable for lost deposits, lost profits, lost opportunities, data inaccuracies, technical failures (client-side, server-side, or blockchain-side), or any consequential damages.

5. **Indemnification**: You agree to indemnify and hold BTR and its contributors harmless from any claims, damages, or liabilities arising from your use of the services.

6. **Information Only**: Documentation, specifications, and materials provided by BTR are for informational purposes only and do not constitute financial, legal, or technical advice.

7. **Independent Verification**: You should always conduct your own research, review all specifications, verify smart contract code, and consult with qualified professionals before using the services.

8. **Audit Status**: You acknowledge that no third-party security audit has been completed for either the DEX protocol or the ALM vault / adapter / Swapper / PriceProvider / AccessControl contracts. Smart contracts are open-source and available for public review, but absence of third-party audit findings does not guarantee absence of vulnerabilities.

9. **Third-Party DEX Acknowledgment** (ALM): You acknowledge that BTR ALM allocates capital to third-party concentrated-liquidity pools (Uniswap v3 / v4, PancakeSwap V3 / Infinity, Aerodrome Slipstream, Algebra, Ramses, and others as added) whose code, governance, and operational decisions are entirely outside BTR's control. BTR does not warrant the security, performance, or continued availability of any third-party pool.

10. **Operator and Keeper Acknowledgment** (ALM): You acknowledge that BTR or its delegates operate keeper services and hold multi-sig governance keys subject to on-chain timelocks and rate limits described in §7.5–§7.9. You accept that compromise, mismanagement, or operator outage may degrade or freeze ALM functionality without necessarily causing direct loss.

11. **Forward-Looking Acknowledgment**: You acknowledge that cross-chain allocation, additional DEX integrations, BTR token issuance, and governance frameworks described as forward-looking are not commitments and may change, be delayed, or be cancelled.

For full terms and conditions, see [our Terms of Service](/terms-of-service).
