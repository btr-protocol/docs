# PROPOSAL: Geographic Restriction Strategy

**Date**: January 22, 2026
**Prepared by**: BTR Legal & Growth Team
**Status**: Draft for Review

---

## Executive Summary

This document proposes a **two-tier approach** to BTR's geographic restrictions, ultimately aiming for **OFAC-only exclusion** to align with major DeFi competitors (Uniswap, Aave, SushiSwap) and expand BTR's Total Addressable Market (TAM) by an estimated **$10-50M**.

---

## Current State

### Current Restricted Territories (Section 2, ToS)

```
United States of America
Canada
China
Japan
Cuba
Members of sanctioned OFAC: Balkans, Belarus, Burma, Côte d'Ivoire, Cuba,
  Democratic Republic of Congo, Iran, Iraq, Liberia, North Korea,
  Russian Federation, Crimea, Sudan, Syria, Zimbabwe
Hong Kong
Thailand
Malaysia
```

### Impact Analysis

| Jurisdiction | Crypto Market Size | Estimated User Base | Current Access |
|-------------|-------------------|-------------------|---------------|
| **USA** | ~22% of global crypto adoption | Largest DeFi market | ❌ Blocked |
| **Canada** | Top 10 global | Major DeFi jurisdiction | ❌ Blocked |
| **Japan** | Top 15 global | Regulated, crypto-friendly | ❌ Blocked |
| **China** | Top 5 global (pre-ban) | Regulatory uncertainty | ❌ Blocked |
| **Hong Kong** | Top 20 global | Asia crypto hub | ❌ Blocked |
| **Thailand/Malaysia** | Growing SE Asia | Emerging DeFi markets | ❌ Blocked |

### Competitor Comparison

| Protocol | USA | Canada | Japan | China | HK | Thailand | Malaysia |
|----------|-----|--------|-------|-------|-----|----------|----------|
| **BTR** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Uniswap** | ❌* | ✅ | ✅ | ❌* | ✅ | ✅ | ✅ |
| **Aave** | ❌* | ✅ | ✅ | ❌* | ✅ | ✅ | ✅ |
| **SushiSwap** | ❌* | ✅ | ✅ | ❌* | ✅ | ✅ | ✅ |

*\*OFAC-sanctioned persons only, not blanket country ban*

---

## Proposed Two-Tier Approach

### Phase 1: Conservative (Current - Recommended Baseline)

**Status**: ✅ Implemented

Maintain current restricted territories list if legal counsel advises immediate regulatory concerns.

**Rationale**:
- Conservative compliance posture
- Mitigates regulatory risk during early launch
- Allows time to establish compliance infrastructure

**Implementation**: Current Terms of Service Section 2

---

### Phase 2: OFAC-Plus (Recommended Transition)

**Timeline**: 3-6 months post-launch

Expand access to **Canada, Japan, Hong Kong, Thailand, Malaysia** while maintaining USA/China ban.

**Restricted Territories**:
```
United States of America
China
Cuba; Iran; North Korea; Syria
Crimea, Donetsk, Luhansk regions of Ukraine
Russian Federation; Belarus; Burma; Democratic Republic of Congo;
  Côte d'Ivoire; Iraq; Liberia; Sudan; Zimbabwe
OFAC-sanctioned individuals and entities
```

**Rationale**:
- **Canada**: Clear crypto regulations, major DeFi jurisdiction
- **Japan**: Highly regulated, crypto-friendly, sophisticated users
- **Hong Kong**: Recent Web3 hub, pro-crypto regulations (effective 2025)
- **Thailand/Malaysia**: Growing SE Asia markets, manageable compliance

**TAM Impact**: ~$5-15M estimated expansion

**Compliance Requirements**:
1. Geo-IP blocking implementation
2. KYC-free but location verification
3. Self-attestation of jurisdiction
4. Clear jurisdiction FAQ/docs
5. Regular compliance review

**Legal Considerations**:
- Canada: Clear securities laws, DeFi guidance available
- Japan: Crypto exchange service provider registration (not applicable for non-custodial protocols)
- Hong Kong: New licensing regime (effective June 2025) - protocols may be exempt
- Thailand/Malaysia: Emerging regulations, monitoring required

---

### Phase 3: OFAC-Only (Long-Term Goal)

**Timeline**: 12-18 months post-launch (subject to legal review)

Transition to **OFAC-only exclusion**, aligning with Uniswap/Aave industry standard.

**Restricted Territories**:
```
Cuba; Iran; North Korea; Syria
Crimea, Donetsk, Luhansk regions of Ukraine
Russian Federation; Belarus; Burma; Democratic Republic of Congo;
  Côte d'Ivoire; Iraq; Liberia; Sudan; Zimbabwe
OFAC-sanctioned individuals and entities (any jurisdiction)
```

**Rationale**:
- Industry standard (Uniswap, Aave, 1inch, Curve)
- Maximum TAM: ~$10-50M additional addressable market
- Simplified compliance framework
- Competitive parity

**TAM Impact**: ~$10-50M estimated expansion

**Compliance Requirements**:
1. OFAC list integration (real-time screening)
2. Self-attestation of non-sanctioned status
3. Jurisdiction self-declaration (no enforcement needed for OFAC-only)
4. Regular OFAC list updates (daily/weekly)
5. US person specific exclusion (Regulation S under Securities Act of 1933)

**Legal Considerations**:
- **US Market Access**:
  - **Option A**: Full block (current) - safest, no regulatory exposure
  - **Option B**: OFAC-only - requires securities law review
  - **Option C**: Hybrid - allow sophisticated users only (accredited investors)

- **Securities Law Analysis Required**:
  - Whether AIMM protocol is a security
  - Whether sLP/sBTR tokens are securities
  - Regulation S exemptions applicability
  - DeFi-specific SEC guidance

---

## Recommended Phased Implementation

### Immediate (Launch)

✅ **Deploy Phase 1** (current conservative list)

**Actions**:
1. Update Terms of Service with current restricted list
2. Implement geo-IP blocking for restricted countries
3. Publish clear FAQ explaining rationale
4. Prepare compliance infrastructure for future expansion

### 3-6 Months Post-Launch

🔄 **Deploy Phase 2** (Canada, Japan, HK, Thailand, Malaysia)

**Actions**:
1. Legal counsel review of target jurisdictions
2. Update Terms of Service with expanded access
3. Update geo-IP blocking (remove Canada, Japan, HK, Thailand, Malaysia)
4. Add jurisdiction self-attestation to onboarding
5. Publish jurisdiction-specific FAQ
6. Monitor for regulatory developments

**Risk Assessment**:

| Risk | Mitigation |
|------|-----------|
| Regulatory changes in target jurisdictions | Regular monitoring, quick rollback capability |
| Increased compliance burden | Automated screening, clear user flow |
| Ambiguous legal interpretations | Conservative interpretation, legal review board |
| User confusion (gradual changes) | Clear communication, migration guide |

### 12-18 Months Post-Launch

🎯 **Deploy Phase 3** (OFAC-only)

**Actions**:
1. Comprehensive securities law review (external counsel)
2. Determine US market access strategy (full block vs. Regulation S)
3. If US access approved:
   - Implement Regulation S compliance flow
   - Add US person attestation
   - Exclude US persons from certain features if needed
4. Update Terms of Service to OFAC-only
5. Remove geo-IP blocking for non-OFAC countries
6. Add OFAC real-time screening
7. Publish comprehensive jurisdiction FAQ

**Risk Assessment**:

| Risk | Probability | Impact | Mitigation |
|------|-----------|-------|-----------|
| SEC enforcement action | Medium | High | External counsel, conservative US access, rapid response plan |
| Adverse regulatory developments | Medium | Medium | Monitoring, rollback capability, jurisdiction diversification |
| OFAC list interpretation disputes | Low | Low | Conservative interpretation, legal review board |
| Increased compliance costs | High | Low | Automation, efficient screening |

---

## Recommended US Market Access Strategy

### Option A: Full Block (Safest)

**Status**: ✅ Current approach

**Details**: Complete USA exclusion (current Phase 1/2)

**Pros**:
- Zero regulatory exposure
- Simplified compliance
- Clear, defensible position

**Cons**:
- Missed largest crypto market ($10-20B TAM)
- Competitive disadvantage (Uniswap/Aave allow US access)

**Recommended**: Maintain until comprehensive legal review completed

---

### Option B: Regulation S Exemption (Recommended Long-Term)

**Status**: ⚠️ Requires legal counsel review

**Details**: Allow non-US persons (Regulation S) but exclude US persons

**Implementation**:
```
Terms of Service addtion:

"U.S. persons (as defined under Regulation S under the Securities Act of 1933,
as amended) are prohibited from using certain protocol features, including:
- Liquidity provision
- Staking
- Token bridging

Non-U.S. persons may access all Services, provided they are not OFAC-sanctioned."
```

**Pros**:
- Access to international crypto users
- Aligns with major DeFi protocols
- Clear legal framework (Regulation S)

**Cons**:
- Requires securities law analysis
- Implementation complexity (US person verification)
- Regulatory uncertainty

**Recommended**: Pursue after 6-12 months, post-launch stability

---

### Option C: Accredited Investors Only

**Status**: ⚠️ Experimental, requires legal counsel review

**Details**: Allow accredited US investors, exclude retail US users

**Implementation**:
- KYC/verification for US accredited investors
- Self-attestation for non-US persons
- OFAC screening for all users

**Pros**:
- Access to sophisticated US capital
- Reduced regulatory exposure (accredited investor exemption)
- Premium user segment

**Cons**:
- KYC requirements contradict DeFi ethos
- Implementation complexity
- User friction

**Recommended**: Explore only if Options A and B are infeasible

---

## Compliance Infrastructure Recommendations

### Phase 1 (Current) - Conservative

1. **Geo-IP Blocking**:
   - IP geolocation service (e.g., MaxMind, IP2Location)
   - Block restricted country IPs
   - Implement VPN detection (optional, see below)

2. **User Self-Attestation**:
   - Checkbox during onboarding: "I am not located in a Restricted Country"
   - Stored with user account

3. **Jurisdiction FAQ**:
   - Clear explanation of restrictions
   - Link to legal counsel rationale (if public)

---

### Phase 2 - Expanded Access

1. **Enhanced Geo-IP**:
   - Remove blocking for Canada, Japan, HK, Thailand, Malaysia
   - Maintain blocking for USA, China, OFAC

2. **Location Verification**:
   - IP-based location detection
   - Timezone-based validation
   - Browser locale detection
   - Allow user override (self-attestation)

3. **OFAC Screening**:
   - Integrate OFAC SDN list API
   - Screen wallet addresses on deposit
   - Screen user-provided jurisdictions

---

### Phase 3 - OFAC-Only

1. **Real-Time OFAC Screening**:
   - Integrate OFAC API (e.g., Chainalysis KYT, TRM Labs)
   - Block OFAC-sanctioned addresses immediately
   - Regular list updates (daily)

2. **US Person Verification** (if US access approved):
   - Regulation S self-attestation flow
   - Optional: KYC for accredited investors
   - Country of residence detection

3. **Jurisdiction Declaration**:
   - User declares jurisdiction on sign-up
   - No verification needed for non-OFAC countries
   - Regular compliance monitoring

---

## VPN Policy Recommendations

### Current Policy (Post-Sothening)

**Language**: "You must not use VPNs, proxies, or other means to circumvent geographic restrictions. Accessing the Services from a Restricted Country, including through the use of VPNs or other location-obfuscation technologies, is a violation of these Terms and may result in immediate termination of access."

**Pros**:
- Clear enforcement
- User accountability
- Legal defensibility

**Cons**:
- False positives (legitimate VPN users)
- User friction

**Recommended**: Maintain current policy (no VPN detection), as user self-attestation provides accountability

---

### Alternative: VPN Detection (Not Recommended)

**Implementation**: Block known VPN IPs via IP reputation services

**Pros**:
- Stronger enforcement
- Reduces circumvention

**Cons**:
- Privacy community backlash
- False positives (corporate VPNs, privacy tools)
- Technical complexity
- Not industry standard (Uniswap/Aave don't block VPNs)

**Recommended**: Do not implement unless circumvention becomes problematic

---

## Communications Strategy

### Internal Team

**Pre-Launch**:
1. **Legal Team**: Review compliance infrastructure
2. **Growth Team**: Prepare jurisdiction FAQ
3. **Community Team**: Prepare messaging for restricted users
4. **Dev Team**: Implement geo-IP blocking

**Post-Launch**:
1. **Monthly Compliance Review**: Monitor regulatory developments
2. **Quarterly Expansion Review**: Assess Phase 2/3 readiness
3. **Annual Legal Audit**: External counsel review

---

### External Communications

**Launch Announcement**:
```
BTR is now live on Ethereum and [other chains]!

⚠️ Geographic Restrictions:
BTR is currently not available to users in the following jurisdictions due to
regulatory considerations:
- United States of America
- Canada
- China
- Japan
- Cuba, Iran, North Korea, Syria
- OFAC-sanctioned countries and territories
- Hong Kong, Thailand, Malaysia

For full details, see our [Terms of Service](/terms-of-service) and
[Geographic Restrictions FAQ](/faq/geographic-restrictions).

We are actively reviewing our geographic policy and may expand access to additional
jurisdictions as regulatory clarity improves.
```

**Jurisdiction FAQ**:
```
Q: Why is BTR not available in my country?
A: Geographic restrictions are based on regulatory uncertainty in certain jurisdictions.
  We prioritize compliance and user protection. We are regularly reviewing our
  policy and will expand access as regulatory clarity improves.

Q: Will BTR ever be available in the USA?
A: We are actively monitoring US DeFi regulatory developments. The SEC and CFTC
  have not provided clear guidance on DeFi protocols. Once clarity emerges, we
  will assess whether we can safely offer services to US users while maintaining
  compliance.

Q: Can I use a VPN to access BTR from a restricted country?
A: No. Geographic restrictions apply regardless of VPN or proxy use. Accessing BTR
  from a restricted jurisdiction is a violation of our Terms of Service and may
  result in immediate termination.

Q: I am a US citizen living in Canada. Can I use BTR?
A: Geographic restrictions apply to your location, not citizenship. If you are
  physically located in a permitted jurisdiction, you may access BTR.
```

**Expansion Announcement (Phase 2)**:
```
BTR is Now Available in Additional Jurisdictions!

We're excited to announce that BTR is now accessible to users in:
✅ Canada
✅ Japan
✅ Hong Kong
✅ Thailand
✅ Malaysia

These jurisdictions have demonstrated clear regulatory frameworks for DeFi protocols.
We look forward to serving users in these markets.

For full details, see our updated [Terms of Service](/terms-of-service).
```

---

## Implementation Checklist

### Phase 1: Conservative (Launch Ready)

- [x] Terms of Service updated with restricted list
- [ ] Geo-IP blocking implemented for restricted countries
- [ ] VPN policy documented (current softened language)
- [ ] Jurisdiction FAQ published
- [ ] Community team trained on restricted user responses
- [ ] Legal counsel reviewed restricted list

### Phase 2: OFAC-Plus (3-6 Months)

- [ ] Legal counsel review of Canada, Japan, HK, Thailand, Malaysia
- [ ] Terms of Service updated with expanded access
- [ ] Geo-IP blocking updated (remove Phase 2 countries)
- [ ] Jurisdiction self-attestation flow implemented
- [ ] OFAC screening API integrated
- [ ] Expansion announcement prepared
- [ ] Monitoring systems in place for target jurisdictions

### Phase 3: OFAC-Only (12-18 Months)

- [ ] External securities law counsel engaged
- [ ] US market access strategy determined (A/B/C)
- [ ] Terms of Service updated to OFAC-only
- [ ] Real-time OFAC screening implemented
- [ ] US person verification flow (if US access approved)
- [ ] Comprehensive jurisdiction FAQ published
- [ ] Rollback plan documented and tested

---

## Decision Matrix

| Factor | Phase 1 (Current) | Phase 2 (OFAC-Plus) | Phase 3 (OFAC-Only) |
|--------|------------------|---------------------|-------------------|
| **Regulatory Risk** | Low | Medium | High |
| **TAM** | $5-10B | $10-25B | $20-60B |
| **Compliance Cost** | Low | Medium | Medium-High |
| **Implementation Time** | ✅ Complete | 1-2 months | 2-4 months |
| **Competitive Parity** | Disadvantage | Neutral | Advantage |
| **Legal Counsel** | Internal review | External review | External + securities |

---

## Recommendations

### Immediate Actions

1. ✅ **Deploy Phase 1** (current conservative list) - COMPLETED
2. ✅ **Soften VPN language** - COMPLETED
3. ✅ **Update opt-out to email** - COMPLETED
4. ✅ **Reformulate technical competence** - COMPLETED
5. ✅ **Enhance Risk Disclaimer** with protocol-specific risks - COMPLETED
6. [ ] Publish jurisdiction FAQ
7. [ ] Implement geo-IP blocking
8. [ ] Schedule Phase 2 legal review (3-6 months)

### Next Quarter

1. Engage external legal counsel for Phase 2 review
2. Begin monitoring target jurisdictions (Canada, Japan, HK, Thailand, Malaysia)
3. Prepare Phase 2 implementation plan
4. Draft expansion communications

### Next 6-12 Months

1. Complete Phase 2 legal review
2. Deploy Phase 2 if approved
3. Begin securities law review for Phase 3
4. Assess US market access options (A/B/C)

---

## Conclusion

BTR's geographic restriction strategy should be **phased and adaptive**, starting with a conservative approach (Phase 1) and expanding to industry-standard OFAC-only exclusion (Phase 3) as regulatory clarity improves.

**Key Milestones**:
- ✅ **Phase 1**: Conservative - NOW
- 🔄 **Phase 2**: OFAC-Plus - 3-6 months
- 🎯 **Phase 3**: OFAC-Only - 12-18 months

**Estimated TAM Expansion**:
- Phase 1: $5-10B
- Phase 2: $10-25B (+$5-15B)
- Phase 3: $20-60B (+$10-35B from Phase 2)

This approach balances regulatory compliance with growth objectives, positioning BTR for competitive parity with major DeFi protocols while minimizing legal risk.

---

**Prepared by**: Sibyl (CEO + Research Lead) with input from Draper (Growth) and Jocasta (Documentation)
**Next Review**: 30 days post-launch
**Approval Required**: Legal Counsel, Core Team
