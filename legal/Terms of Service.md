# BTR TERMS OF SERVICE

*Last Updated: January 2026*

---

## ⚠️ IMPORTANT NOTICE: GEOGRAPHIC RESTRICTIONS

You must not use VPNs, proxies, or other means to circumvent geographic restrictions or access the Services from a Restricted Country. Accessing the Services from a Restricted Country, including through the use of VPNs or other location-obfuscation technologies, is a violation of these Terms and may result in immediate termination of access. If you are located in a restricted jurisdiction, you are not permitted to use the Services.

---

Please read these Terms of Service carefully as they form a legally binding contract between you and us.

These terms of service (the "Terms") govern your ("you" or "your") use of the services provided by BTR ("BTR", "the DAO", "we", "us" or "our"), including the web interface at [https://btr.supply](https://btr.supply) (the "Interface") and the underlying AIMM protocol smart contracts (collectively, the "Services"), and creates a binding contract between you and us.

BTR operates a front-end interface that allows interaction with two distinct on-chain product lines (collectively the "Protocol"):

1. **AIMM DEX Protocol** — the Automated Inventory Market Maker, a multi-asset DEX with hub-and-spoke routing, dynamic fees, and coverage-ratio-based active liquidity management. Diamond-proxy upgradeable, deployed on Ethereum and compatible EVM chains.
2. **BTR ALM (Active Liquidity Management) v1.x** — a hybrid ERC-4626 / ERC-7540 vault system that allocates user deposits across up to eight concentrated-liquidity adapters per vault. Each adapter wraps a third-party concentrated-liquidity pool (e.g., Uniswap v3 / v4, PancakeSwap V3 / Infinity, Aerodrome Slipstream, Algebra, Ramses) and may serve multiple holders share-weighted. Vaults route swaps and salvage flows through a chain-singleton Swapper that supports both LiFi atomic execution and ERC-7683 cross-chain intents.

BTR does not operate, control, or have custody over the underlying blockchains, third-party DEX pools, intent settlers, atomic-swap routers, oracles, or user assets. The smart contracts are decentralized infrastructure; BTR's role is limited to:
- Maintaining the web interface for protocol interaction
- Operating optional keeper bots for protocol health (oracle updates, circuit-breaker checks, ALM adapter rebalances, defensive-ratchet kills, intent settlement, late-refund processing)
- Curating ALM vault configurations (adapter selection, fee tiers, tick ranges, deviation / heartbeat / slippage / interval defensive parameters) within the bounds enforced by the smart contracts
- Providing documentation and specifications

**ALM Vault Custody Model**: BTR ALM vaults take custody of deposited assets *within the smart contract* for the duration of the deposit. The depositor retains an ERC-20 share token redeemable for the underlying assets, subject to the protocol's redemption mechanics (synchronous fallback or ERC-7540 asynchronous request, per vault configuration), pessimistic NAV accounting, exit-fee floor, high-water-mark performance fee, and the operational and third-party risks described in [our Risk Disclaimer](/risk-disclaimer). At no point does BTR (the entity, the team, or any individual contributor) hold private keys to user deposits; vault-level governance authority over adapter configuration, swapper selection, and emergency controls vests in the AccessControl contract, which is itself constrained by on-chain timelocks and rate-limited keeper actions.

**Smart Contract Upgradeability**: The protocol uses upgradeable smart contracts via beacon proxy pattern. While the core logic is designed for stability, upgrades may be executed by governance or contract owners to fix bugs, add features, or respond to security incidents. Upgrades are transparent and follow governance processes, but may introduce new risks.

Any reference to "Wallet" in these Terms shall mean the crypto wallet you use to interact with our Services.
We ask you please read these Terms thoroughly so you properly understand your obligations and your rights in relation to our Services and your Wallet. By installing, accessing, downloading or otherwise using the Services, you are agreeing to comply with and be bound by these Terms. If you breach or violate any of the Terms, we have the right to disable and block your access to the Services immediately.

We can make updates to these Terms at any time and will make any updated versions of the Terms available via the Services. Your continued use of the Services will be deemed acceptance of the updated Terms. However, if you do not accept the updated Terms, you are no longer permitted to use the Services. We therefore encourage you to regularly review the Terms to ensure you are fully aware of your obligations and rights with regards the Services.

## 1. CONTACT

If you have any questions about the Services, these Terms, or our Privacy Policy, please reach out to the team at [contact@btr.supply](contact@btr.supply) or [on X](https://x.com/BtrSupply).

## 2. RESTRICTIONS OF USE

BTR Supply does not allow residents or individuals located in, or entities organised under the laws of, the following countries or territories to interact with its protocol and/or otherwise use its services: United States of America; Canada; China; Japan; Cuba; members of sanctioned OFAC including the Balkans, Belarus, Burma, Cote D'Ivoire (Ivory Coast), Cuba, Democratic Republic of Congo, Iran, Iraq, Liberia, North Korea, Russian Federation, Crimea, Sudan, Syria, and Zimbabwe; or any other countries in which the services or part thereof may be restricted or considered unlawful such as Hong Kong, Thailand, Malaysia (together, the "Restricted Countries" and each a "Restricted Country").

For the avoidance of doubt, BTR Supply does not offer its Services to residents of, individuals located in, or entities organised under the laws of, any Restricted Country, and all such persons and entities are expressly restricted from accessing or using the Services. This includes any U.S. person (as defined under Regulation S under the Securities Act of 1993 (as amended)).

It is your responsibility to ensure that the country from which you access the services is not a Restricted Country and we encourage you to seek legal advice before using the Services.

## 3. ELIGIBILITY

You warrant and represent that you satisfy the following eligibility requirements to use the Services:

- **Age**: You must be at least 18 years old or the age of majority in your jurisdiction, whichever is higher. If acting on behalf of an organization, you must have legal authority to bind that entity.

- **Authorized Entity**: If using the Services on behalf of a legal entity, you represent that: (i) the entity is duly organized and validly existing under applicable laws; and (ii) you are duly authorized to act on its behalf.

- **Location**: You are not located in, resident of, or formed under the laws of a Restricted Country (section 2).

- **Previous Suspensions**: You have not been previously suspended, removed, or restricted from using BTR Services.

- **Technical Awareness**: You understand that:
  - Blockchain technology and smart contracts are experimental technologies
  - Transactions on blockchains are irreversible and final
  - You are responsible for managing your private keys and wallet security
  - Using DeFi protocols involves financial and technical risks that you should research independently
  - BTR's smart contracts are open-source and available for independent review or third-party auditing
  - You should review contract addresses and verify them through trusted sources before interacting

- **Self-Custody Understanding**: You understand that you retain full custody and control of your digital assets. You are solely responsible for the security of your private keys, wallet, and devices. Lost or compromised keys result in permanent, irreversible loss of funds.

## 4. USE OF THE SERVICES

You agree to comply with the following when using the Services:

- **No Restricted Country Access**: You must not access the Services from a Restricted Country (section 2). **Geographic restrictions are detailed in the important notice at the top of these Terms.**

- **No Market Manipulation**: You must not engage in:
  - Wash trading or self-dealing to artificially inflate volume
  - Front-running other users' transactions
  - Flash loan attacks designed to manipulate prices or drain liquidity
  - Any activity intended to manipulate oracle prices, coverage ratios, or protocol parameters
  - Sybil attacks or multi-account abuse to exploit governance or incentives

- **Wallet Usage**: Use one wallet per individual or authorized representative. Organizations should restrict wallet access to authorized individuals only.

## 5. FEES AND CHARGES

You are solely responsible for paying all fees, charges, and costs associated with your use of the Services, including but not limited to:

- **Network Fees**: You must pay all gas fees, transaction fees, and other network costs required by the underlying blockchain network for every transaction you execute through the Services. These fees are determined by network conditions and are not under our control.

- **Protocol Fees**: The AIMM protocol may charge protocol fees on certain transactions, such as swaps or liquidity operations. These fees, if any, are transparently displayed before transaction submission and are automatically deducted from transaction amounts.

- **ALM Vault Fees**: BTR ALM vaults charge three categories of fees, each parameter-bounded in the smart contract code (denominated in `pip`, base 1e5):
  - **Exit fee** (`exitPip`): floor scales with adapter risk parameters per the on-chain rule `exitPip ≥ 1.2 × max_i(deviationBps + heartbeatDriftBps) × 10`, hard-floored at 50 pip (0.05%) and capped at 30 000 pip (30%). Exit fees protect remaining holders against arbitrageur extraction at vault redemption boundaries; they are *not* refundable.
  - **Management fee** (`mgmtPip`): annualized, accrues continuously, capped at 50 000 pip (50%) per year. Crystallized on each fee-bearing operation by minting shares to the treasury.
  - **Performance fee** (`perfPip`): high-water-mark (HWM) based, capped at 20 000 pip (20%). Crystallizes only when (assets, supply) jointly exceed the previous HWM pair, computed against the pessimistic worst-of(pool, oracle) NAV.

  Vault fee parameters may be reconfigured by governance subject to on-chain bounds; you should review the live values before depositing. Underlying third-party DEX pools (Uniswap v3 / v4, PancakeSwap, Aerodrome, etc.) charge their own LP / swap fees, which accrue inside the adapter and are reflected in NAV but are not separately itemized.

- **Swap & Intent Fees**: Swaps executed through the Swapper (whether for adapter rebalance, vault salvage, or intent fill) incur the underlying router / settler fees (LiFi for atomic; ERC-7683 settler for intents) plus any third-party aggregator fees (LiFi, Unizen, Squid, Rango, Socket, etc.) embedded in the route. Slippage authority lives at the calling adapter's pre/post-swap pessimistic envelope; you accept any unfavorable fill within tolerance.

- **Fee Estimates**: Any fee estimates provided by the Interface are approximations only and may not reflect actual network conditions at the time of transaction. We are not responsible for discrepancies between estimated and actual fees.

- **Third-Party Fees**: If you interact with third-party services or protocols through the Interface, those services may impose additional fees. You are responsible for understanding and paying any such third-party fees.

## 6. GENERAL CONDITIONS

In addition to any other conditions included in these Terms which you must comply with, please also be aware of the following:

- Law: you must comply with all applicable laws, regulations and rules and not be prohibited from using the Services in any way due to the applicable laws, regulations and rules or any criminal conviction you may have, including if you are located in a Restricted Country.
- Banned Wallet: you must not use the Services if your Wallet has been banned (blacklisted) by BTR Supply in the past.
  BTR Supply will, without notice, ban Wallets identified as, or related to prohibited use of the Services or other banned Wallets.
- Wallet Security: you are responsible for keeping your device, your wallet whether hardware or software based, and its keys/codes/passwords or any access method safe and secure.
  BTR Supply Services are permissionless, and we do not store nor manage your Wallet in any way. You are responsible for any breach or security compromise of your device and/or Wallet, this is out of BTR Supply's control.

## 7. ACCEPTABLE USE

When using the Services, you must always do so for legal, authorised and acceptable purposes. When using the Services, you will not, nor permit any person to:

- use them in any unlawful manner, for any unlawful purpose, or in any manner inconsistent with or contravening these Terms;
- violate or infringe the rights of BTR Supply, any of its team members, our users, or other third parties, including rights in respect of privacy, intellectual property, or other proprietary rights;
- provide false, inaccurate or misleading information;
- engage in any potentially fraudulent or suspicious activity and/or transactions;
- refuse to cooperate with an investigation or provide any information we request;
- transmit any communication, material or other content that is illegal, obscene,
  defamatory, threatening, intimidating, harassing, hateful, racially or ethnically
  offensive, or which could instigate or encourage conduct that would be illegal
  or otherwise inappropriate, including promoting violent crimes;
- impersonate someone or an entity;
- seek to harm BTR Supply, our team members, or any of our users or customers; or
- access, use, copy, modify, distribute, display, otherwise exploit or prepare derivative works based upon and of our Services in impermissible or unauthorised manners, or in ways that harm us, our team members, our Services, systems, users, or others. In particular you must not, nor permit any person to: - reverse engineer, alter, modify, create derivative works from,
  decompile, or extract code from our Services, except to the extent
  permitted by law; - send, store, or transmit viruses or other harmful computer code
  through or onto our Services; - gain or attempt to gain unauthorised access to our systems or those of
  our team members; - create accounts for our Services through unauthorised means; - collect the information of or about our users in any impermissible or
  unauthorised manner; and/or - sell, resell, rent, or charge for the use of, our Services.

## 8. LICENCES AND INTELLECTUAL PROPERTY

### 8.1. What We Own or Control

Throughout our relationship with you, we retain ownership and/or control, as appropriate, of the Services and all intellectual property rights therein, including all copyrights, trademarks, domains, logos, trade secrets, patents, and other intellectual property rights that have existed, do, may or will exist in the future ("Intellectual Property"). You may not use our Intellectual Property unless permitted under these Terms or with our express written consent (which you can seek by contacting us using the contact details above).

### 8.2. What You Own

Throughout our relationship with you, you retain ownership of the details, information or other data you submit for your Wallet or through the use of our Services. You also have the right to grant the rights and licences in these Terms as further detailed below.

### 8.3. Licence to Us

In order for us to provide our Services to you to the best of our ability, you grant BTR Supply a worldwide, royalty-free, sublicensable, and transferable license to use, reproduce, distribute, create derivative works of, and display the information (including any content) that you upload, store, send, or receive on or through the use of the Services. We will only use such rights for the limited purpose of providing our Services.

### 8.4. Right to You

In order for you to receive the Services at the best standard, we grant you a revocable, non-exclusive, non-sublicenseable and non-transferable right to use the Services for the sole purpose for which they were created, subject to and in accordance with these Terms. No other licences or rights are granted to you by implication or otherwise.

### 8.5. Open-Source License Compliance

The Services may incorporate or be based upon open-source software components that are licensed under various open-source licenses ("OSS Licenses"). You acknowledge that certain components of the Services may be subject to OSS Licenses, which may grant you rights that differ from the rights granted under these Terms. Where OSS Licenses apply, those licenses will govern your use of those specific components. We comply with all applicable OSS Licenses and make source code available as required by such licenses. For information about open-source components and their licenses, please contact us at [contact@btr.supply](contact@btr.supply).

### 8.6. Infringement of Intellectual Property

If you believe our Intellectual Property is being infringed by a third party, please contact us immediately using the contact details provided above. If we believe you are infringing our Intellectual Property or the intellectual property of another, we may terminate your access to the Services with immediate effect.

## 9. PRIVACY

As you may know, to use the Services, we must collect, store, use, share and otherwise process your personal data. To learn more about why and how we do this, please read [our Privacy Policy](/privacy-policy). It will also provide useful information about the rights you may have in relation to your personal data and how you may be able to exercise such rights.

## 10. DISCLAIMER

You use and accept the provision of the Services at your own risk and at all times subject to the following disclaimers:

- the Services are each provided on an "as is" and "as available" basis without any express or implied warranties;
- any smart contracts you interact with are entirely your own responsibility and liability, and that we are not party to the smart contracts;
- at any time, your access to your cryptocurrency assets may be suspended or terminated or there may be a delay in your access or use of your cryptocurrency assets which may result in the cryptocurrency assets diminishing in value or you being unable to complete a Smart Contract;
- no warranty is given that any information or content provided or made available by us is accurate, up to date, complete, or useful, that the Services will be operational, free from errors, secure, safe, or that the Services will function without disruptions, delays or imperfections;
- no warranty is given regarding the availability or uninterrupted use of the Services. From time to time, access may be interrupted, suspended or restricted, including because of a fault, error or unforeseen circumstances or because we are carrying out planned maintenance;
- we do not control, and are not responsible for controlling, how or when our users use our Services; and
- we are not responsible for and are not obligated to control the actions or information (including content) of our users or other third parties.
  The foregoing disclaimers will apply to the fullest extent permitted by applicable law.
  Further risk information can be found in [our Risk Disclaimer](/risk-disclaimer)

## 11. SERVICE MODIFICATIONS AND SUSPENSION

### 11.1. Right to Modify Services

We reserve the right, at our sole discretion, to modify, update, upgrade, or change any aspect of the Services at any time with or without prior notice, including but not limited to:

- The features, functionality, or user interface of the Interface
- The smart contract code or protocol parameters
- The pricing algorithms, fee structures, or reward mechanisms
- The supported assets, liquidity pools, or trading pairs
- The keeper bot operations or oracle configurations
- The availability of specific features or functions

### 11.2. Right to Suspend or Discontinue Services

We may, at any time and without prior notice, temporarily suspend or permanently discontinue:

- Access to the Interface or any part thereof
- Operation of specific smart contracts or protocol features
- Support for particular assets, chains, or liquidity pools
- The keeper bot services
- Any other aspect of the Services

Suspensions or discontinuations may occur for any reason, including but not limited to: technical issues, security concerns, regulatory requirements, business considerations, or force majeure events.

### 11.3. No Liability for Modifications or Suspensions

We will not be liable to you or any third party for any modification, suspension, or discontinuation of the Services, or for any loss, damage, or expense resulting from such actions. You acknowledge that your ability to access or use the Services may be interrupted at any time.

## 12. REGULATORY COMPLIANCE

### 12.1. Cooperation with Authorities

You acknowledge and agree that we may cooperate with law enforcement, regulatory authorities, tax agencies, and other government entities in connection with investigations or inquiries related to your use of the Services. We may:

- Disclose information about your account, transactions, or activities to government authorities upon request, subpoena, or legal process
- Implement compliance measures required by applicable laws or regulations
- Restrict or terminate access to the Services in response to regulatory directives
- Report suspicious activities or transactions to appropriate authorities

### 12.2. Sanctions Compliance

You represent and warrant that you are not located in, resident of, or formed under the laws of any jurisdiction subject to comprehensive sanctions administered by the U.S. Office of Foreign Assets Control (OFAC), the U.S. Department of State, the United Nations Security Council, the European Union, Her Majesty's Treasury, or other relevant sanctions authorities. We may block access to the Services for individuals or entities identified on sanctions lists.

### 12.3. Anti-Money Laundering (AML)

The Services may include AML monitoring and screening mechanisms. You agree not to use the Services for any purpose related to money laundering, terrorist financing, or other illegal activities. We may report suspicious transactions to appropriate authorities.

### 12.4. Tax Compliance

You are solely responsible for complying with all applicable tax laws and reporting requirements related to your use of the Services. We do not provide tax advice and are not responsible for your tax obligations.

## 13. THIRD-PARTY RESOURCES

The Interface may contain links or references to third-party websites, applications, services, or content ("Third-Party Resources") for your convenience. We do not control, endorse, or sponsor any Third-Party Resources, and we are not responsible for:

- The accuracy, completeness, or reliability of any Third-Party Resources
- The content, products, or services available on Third-Party Resources
- The privacy practices, terms of use, or policies of third parties
- Any transactions or interactions you have with third parties

Your use of Third-Party Resources is at your own risk and is subject to the terms and conditions of those third parties. We encourage you to review the terms and privacy policies of any third-party service before using it.

## 14. LIABILITY

### 14.1. No Fiduciary Relationship

You expressly acknowledge and agree that we are not your broker, underwriter advisor, intermediary or agent, and we have no a fiduciary relationship or obligation to you regarding any decisions, actions or activities that you effect, make or undertake when using the Services. Neither our communications, nor any information or content that we provide or make available to you, is intended as, or shall be considered or construed as financial, legal or any other advice.

### 14.2. Our Liability

We accept liability for any losses you may suffer as a result of us not complying with our obligations under our Terms with you, or as a result of our negligence or fraud.

### 14.3. Limitation and Exclusion of Our Liability

Unless expressly stated otherwise in these Terms, we, our affiliates, subsidiaries, directors, employees, partners and agents (together the "BTR Supply team") will not be held liable to you, whether in contract (including under any indemnity), tort (including negligence), under statute or otherwise, under or in connection with these Terms, your activities under your Wallet, your use of the Services, and/or our provision of the Services, for any:

- consequential, indirect or special losses or damages;
- loss of profits, loss of business or loss of business opportunity;
- losses incurred as a result of abnormal or unforeseeable circumstances outside our reasonable control, including delays or failures caused by problems of another system or network, mechanical breakdown, industrial action or a pandemic;
- losses incurred as a result of unauthorised access or use of your Wallet;
- losses incurred where a law, or guidance or instruction from a governmental authority, requires us to take action, for example to terminate these Terms and cease providing you access to the Services;
- losses resulting from protocol risks, smart contract vulnerabilities, or technical failures (detailed in [our Risk Disclaimer](/risk-disclaimer));
- losses resulting from market risks, volatility, or financial losses (detailed in [our Risk Disclaimer](/risk-disclaimer));
- losses resulting from oracle errors, manipulation, or failures (detailed in [our Risk Disclaimer](/risk-disclaimer));
- losses resulting from blockchain network risks, forks, or infrastructure issues (detailed in [our Risk Disclaimer](/risk-disclaimer));
- losses resulting from malicious attacks, exploits, or security vulnerabilities (detailed in [our Risk Disclaimer](/risk-disclaimer)); or
- any other loss or damage to the extent that such loss or damage is caused or contributed to by you, or is a result of the failure by you to comply with these Terms.

For any other losses or damages for which we are found liable, the BTR Supply's team aggregate liability towards you for all claims during our relationship with you shall not exceed $100. The foregoing exclusions and cap of liability will each apply to the fullest extent permitted by applicable law.

### 14.4. Your Liability and Indemnification

To the extent permitted by law, you agree to defend, indemnify, and hold harmless the BTR Supply's team from and against all liabilities, damages, losses, and costs (including reasonable legal fees and costs) relating to, arising out of, or in any way in connection with any of the following: (a) your access to or use of our Services; (b) your breach of our Terms; (c) any misrepresentation made by you; or (d) any claim, suit, action, demand, settlement, judgment, or other proceeding brought by a third party against us arising out of or relating to your use of the Services.

**Indemnification Control**: We have the exclusive right to control the defense and settlement of any such third-party claim. You agree to cooperate fully with us in the defense of any such claim, including providing us with reasonable assistance, information, and authority to defend against the claim. You may not settle any such claim without our prior written consent. We will have the right to approve any settlement and will have the sole discretion to determine the manner of defense.

## 15. TERMINATION OF THESE TERMS

If your access to the Services is terminated in accordance with any of the following provisions or any other provision in these Terms, these Terms will automatically terminate.

### 15.1. Our Right

We may, at any time, disable your access to the Services through a ban (temporary or permanent) and/or terminate your access and use of any or all of the Services for any reason, including if any of the following occur:

- we suspect you are using the Services for, or permitting the Services to be used for, criminal or fraudulent purposes;
- we suspect you are located or established in a Restricted Country;
- we suspect someone is using your Wallet without your authority;
- we suspect or identify that you are not eligible to use the Services;
- your behaviour towards our staff makes it difficult for us to deal with you;
- you do not accept any updated versions of these Terms;
- you do not pay any moneys owed to us;
- you do not pay any interest, fees or charges owed on time;
- you go into bankruptcy, enter into an individual voluntary arrangement, have a debt relief order or trust deed lodged against you or enter into any other form of analogous circumstances;
- you die; or
- we suspect you have violated these Terms.

### 15.2. Your Right

You can cease using the Services at any time.

### 15.3. Continuing Provisions

However caused, the following provisions will survive any termination of your relationship with us: Licences And Intellectual Property, Disclaimer, Our Liability, Your Liability, Termination Of These Terms, General, Dispute Resolution, Governing Law And Jurisdiction, Statute Of Limitations, Regulatory Compliance.

## 16. LANGUAGE

These Terms were prepared in English and therefore the default language is English. Should you wish to access these Terms in a different language, please update the language setting on your device to the preferred language. Please note that not all languages are available and should there be any discrepancies or differing interpretations or meanings between the English version of these Terms and any translation, the English Terms shall prevail.

## 17. STATUTE OF LIMITATIONS

Except as otherwise prohibited by applicable law, any cause of action, claim, or suit arising out of or related to these Terms or the Services must be filed within one (1) year after the cause of action accrued. Any claim or cause of action not filed within this one-year period is waived and barred. This limitation period applies regardless of whether the claim is based on contract, tort, statute, or any other legal theory.

## 18. GOVERNING LAW AND JURISDICTION

### 18.1. Governing Law

These Terms and any dispute, controversy, or claim arising out of or in connection with these Terms or the Services shall be governed by and construed in accordance with the laws of the Cayman Islands, without regard to its conflict of law principles.

### 18.2. Jurisdiction

Any legal action, suit, or proceeding arising out of or relating to these Terms or the Services shall be brought in the courts of the Cayman Islands. You hereby irrevocably waive any objection to the exclusive jurisdiction and venue of the Cayman Islands courts and consent to personal jurisdiction in those courts. However, we may seek injunctive or other equitable relief in any court of competent jurisdiction to protect our intellectual property or other proprietary rights.

### 18.3. International Use

The Services are controlled and operated from locations in the Cayman Islands. If you access the Services from any other jurisdiction, you are responsible for compliance with all applicable local laws. We make no representation that the Services are appropriate or available for use in other jurisdictions.

## 19. DISPUTE RESOLUTION

### 19.1. Mandatory Binding Arbitration

**YOU AND BTR SUPPLY AGREE THAT ANY DISPUTES, CLAIMS, OR CONTROVERSY ARISING OUT OF OR RELATING TO THESE TERMS OR THE SERVICES, INCLUDING ANY CLAIM BASED ON CONTRACT, TORT, STATUTE, FRAUD, MISREPRESENTATION, OR ANY OTHER LEGAL THEORY, SHALL BE RESOLVED EXCLUSIVELY BY BINDING ARBITRATION, AND NOT IN COURT, EXCEPT AS OTHERWISE PROVIDED IN THIS SECTION.**

The arbitration shall be administered by JAMS (Judicial Arbitration and Mediation Services) in accordance with its Comprehensive Arbitration Rules and Procedures. The arbitration shall be conducted in the Cayman Islands, in the English language.

The arbitrator shall have the authority to award any remedies or relief that a court could award, including but not limited to damages, injunctive relief, and specific performance. The arbitrator's award shall be final and binding, and judgment on the award may be entered in any court having jurisdiction.

### 19.2. Class Action and Representative Action Waiver

**YOU AND BTR SUPPLY HEREBY WAIVE ANY RIGHT TO A JURY TRIAL.** Furthermore, **YOU AND BTR SUPPLY AGREE THAT EACH MAY BRING CLAIMS AGAINST THE OTHER ONLY IN YOUR OR ITS INDIVIDUAL CAPACITY, AND NOT AS A PLAINTIFF OR CLASS MEMBER IN ANY PURPORTED CLASS OR REPRESENTATIVE ACTION.** Unless both you and BTR Supply expressly agree otherwise in writing, the arbitrator may not consolidate more than one person's claims and may not otherwise presided over any form of a class, consolidated, or representative proceeding. If this provision is found to be unenforceable, then the entirety of this Dispute Resolution section shall be null and void.

### 19.3. Waiver of Right to Trial

Both you and BTR Supply waive all rights to trial by jury and to have any dispute adjudicated in a court of law. By agreeing to arbitration, both parties understand that they are giving up the right to seek remedies in court, including the right to a jury trial, the right to participate in class actions or representative proceedings, the right to certain discovery procedures, and the right to appeal.

### 19.4. Small Claims Exception

Notwithstanding the foregoing, either you or BTR Supply may bring an individual action in small claims court for disputes within the jurisdiction of that court, provided that the action remains in small claims court and does not seek class-wide relief.

### 19.5. Opt-Out (Limited)

Notwithstanding Section 19.1, if you are an individual (not an entity), you may opt out of the mandatory arbitration provision by sending a written notice of your election to opt out. The notice must be sent via email to:

legal@btr.supply

The notice must be sent within thirty (30) days of your first use of the Services and must include your name, email address associated with your account, and a clear statement of your election to opt out of the arbitration provision. We will confirm receipt of your opt-out notice via email. If you opt out, both you and BTR Supply retain the right to litigate disputes in court.

## 20. BTR ALM-SPECIFIC TERMS

### 20.1. Third-Party DEX Liquidity Provisioning

ALM vault adapters allocate user deposits to external concentrated-liquidity pools operated by independent protocols (Uniswap v3 / v4, PancakeSwap V3 / Infinity, Aerodrome Slipstream, Algebra, Ramses, and additional integrations as added). You acknowledge and accept that:

- The composition, behavior, security, and continued availability of those external pools are entirely outside BTR's control.
- Bugs, exploits, governance actions, fee-tier changes, hook misbehavior (V4 / Infinity), or shutdown of any underlying pool may result in partial or total loss of vault assets allocated to the affected adapter.
- Hooks (Uniswap V4 / PancakeSwap Infinity) introduce arbitrary code paths during pool operations; BTR does not audit or guarantee any third-party hook.

### 20.2. Vault Accounting and Pessimism

ALM vaults compute net asset value as the *worst-of* the pool-derived mark and the oracle-derived mark for each adapter's holdings. This is a deliberately pessimistic accounting choice that:

- May understate NAV during normal operation, leading to lower share prices and, on redemption, to slightly lower payouts than a pool-only or oracle-only mark would produce.
- Protects remaining holders against just-in-time (JIT) deposits and arbitrageur extraction across stale-price boundaries.
- Means that two consecutive deposits or redemptions at apparently identical pool prices can yield different share / asset outcomes depending on oracle freshness, sequencer state, and pool tick movement.

### 20.3. Redemption Mechanics

Vaults support synchronous redemption when liquidity is available and asynchronous redemption (ERC-7540 request / claim) otherwise. You acknowledge that:

- A redemption request is *not* a guarantee of immediate payout. Asynchronous requests are processed in cohorts and may be subject to a haircut if the vault is processing a shortfall cohort.
- During large-redemption episodes, exit fee is applied at the prevailing rate, which may be elevated relative to historical baseline if defensive ratchets have raised it.
- Operator / controller models (ERC-7540) allow a third party to act on your behalf if you have explicitly granted operator status; revoke operators you no longer trust.

### 20.4. Operator and Keeper Risk

BTR (or third parties under BTR's direction) operates keeper services that perform operational tasks against ALM vaults and adapters, including but not limited to: rebalancing positions, ratcheting defensive parameters (tightening `depositCap`, `slipBps`, `interval`, `deviationBps`), marking adapters frozen / killed under rate limits (the on-chain cluster cap is two keeper-initiated kills per UTC day), driving intent settlement / refund flows, and updating Chainlink-backed oracle configurations within the timelocked PriceProvider.

You acknowledge that:

- Keeper *actions* are constrained by the on-chain rules (timelocks, rate limits, cluster caps) but keeper *availability* is not guaranteed; keeper outage during volatility may degrade vault performance.
- Multi-sig keys held by BTR or its delegates retain governance authority over adapter configuration, swapper rotation (subject to a 7-day timelock), factory rotation (14-day timelock), feed rotation (6-hour timelock), and treasury rotation (timelocked). Compromise or misuse of these keys, although mitigated by timelocks, is an unavoidable centralization risk.
- The kill switch (per-adapter, rate-limited to once per hour for keepers, with a 1-day freeze timelock) can lock funds at the affected adapter pending owner action.

### 20.5. Swapper, Intents, and Atomic Routing

Vault salvage and adapter rebalances route through a chain-singleton Swapper that supports two paths: atomic execution via a pinned LiFi router, and ERC-7683 cross-chain intents via a pinned settler. You acknowledge that:

- Intents are not atomic; they have a deadline (5 minutes to 24 hours) during which the vault's `assetValue` may temporarily under-report (in-flight portion held in escrow at the Swapper).
- Intent fill quality depends on third-party solver / settler counterparties; a settler that defaults, mis-fills, or is sanctioned can leave funds in late-refund limbo until grace expires (6 hours post-deadline) and `forceRefund` is callable.
- Route execution may include third-party aggregators (LiFi, Unizen, Squid, Rango, Socket, others); each adds counterparty risk that BTR cannot fully attest to.

### 20.6. Cross-Chain and Multi-Chain (Forward-Looking)

BTR may extend ALM allocation across multiple EVM chains in future versions. Forward-looking cross-chain features inherit additional risks beyond same-chain operation: bridge solvency, cross-chain message latency, finality differences across chains, sequencer outages on L2s (the PriceProvider checks an L2 sequencer feed and reverts on `SeqDown` within a 1-hour grace window), and cross-chain governance key custody. You acknowledge that any forward-looking statements about future cross-chain features are not commitments.

### 20.7. BTR Token and Governance (Forward-Looking)

BTR plans to issue a governance token ("BTR token"). The eventual issuance, schedule, allocation, vesting, voting weight, and treasury controls are subject to change and are not committed by these Terms. You acknowledge that:

- Governance tokens, if issued, may be subject to regulatory classification in your jurisdiction; you alone are responsible for your tax and legal treatment.
- Token markets are volatile; participation in any sale, airdrop, or secondary purchase is at your own risk.
- Voting weight may concentrate among large holders, contributors, or treasury; on-chain governance outcomes may not align with your interests.
- Participation in governance proposals is at your informed-decision risk; BTR makes no representation that any proposal will or will not pass, or that any specific economic outcome will result.

### 20.8. No Fiduciary Relationship for ALM Depositors

Depositing into a BTR ALM vault does *not* create a fiduciary, advisory, brokerage, custodial-trust, fund-management, or partnership relationship between you and BTR. Vault strategies, adapter selections, fee tiers, tick ranges, and risk parameters are published in code and on the Interface; you are deemed to have reviewed and accepted them by depositing. Nothing on the Interface, in documentation, in marketing materials, in social media posts, or in community channels constitutes financial, legal, tax, or investment advice or a guarantee of return. Past performance, simulated performance, and back-tested performance — if displayed — do not predict future results.

### 20.9. Operator Location & Jurisdiction

BTR's contributors operate from various jurisdictions. The Services and ALM vaults are controlled, configured, and monitored from locations outside of any Restricted Country. Jurisdiction over disputes is set in Section 18 of these Terms. Routing of intent / atomic swaps may transit chains and infrastructure across multiple jurisdictions; this transit does not establish BTR presence in any such jurisdiction.

## 21. GENERAL

In addition to any other terms and conditions within these Terms, please be aware of the following:

- Entire Agreement: unless stated otherwise by BTR Supply, these Terms make up the entire agreement between you and BTR Supply regarding your use of the Services.
- Restricted Countries: our Services should not be accessed or otherwise used in any Restricted Country.
- Waiver: any waiver of any Terms or obligations or rights hereunder is not permitted without our written consent.
- Assignment: we may assign or otherwise transfer all or some of our rights and obligations under these Terms to another party. You may assign or otherwise transfer all or some of our rights and obligations under these Terms to another party with our prior written consent.
- Third Parties: except as stated or contemplated herein, these Terms do not give any third party any rights.
- Severance: if any provision of these Terms is deemed illegal, unlawful, void, or for any reason unenforceable, then that provision shall be deemed severable from our Terms and shall not affect the validity and enforceability of the remaining provisions.
- Headings: the headings in these Terms are for convenience only and do not affect interpretation.
- Electronic Communications: You agree to receive electronic communications from us and that these communications satisfy any legal requirement that such communications be in writing.

---

**By using the Services, you acknowledge that you have read, understood, and agree to be bound by these Terms of Service.**
