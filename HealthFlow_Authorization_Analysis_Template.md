# Technical Analysis & Action Plan Package
## The Authorization Collapse at HealthFlow
### Prepared by: Carlos Medina, Technical Account Manager | Date: March 18, 2025 | Confidential

---

# Section 1: Root Cause Analysis

## 1.1 Executive Summary of the Problem

| Metric | Before Incident | Current | Delta |
|--------|----------------|---------|-------|
| Overall Authorization Rate | 82.0% | 67.0% | -15.0 pp |
| Monthly Transaction Volume | ~$2.8M | ~$2.19M | -~$610K |
| Affected Transaction Count (weekly) | — | ~6,748 declines | — |
| Estimated Revenue Impact | — | ~$600K/month | — |

> **Key Finding:** The authorization collapse is not a single issue — it is **four distinct, compounding problems** that emerged within a two-week window: a 3DS threshold misconfiguration interacting with dLocal's 3DS2 upgrade, a Chile regulatory compliance failure, an aggressive retry logic triggering issuer rate limits, and a Brazilian issuer policy change affecting major banks. Each requires its own targeted fix.

## 1.2 Root Causes (Prioritized by Revenue Impact)

### Root Cause #1: 3DS Threshold Misconfiguration + dLocal 3DS2 Upgrade Interaction — Estimated Impact: ~$334,400/month
- **Description:** On March 4th, the 3DS challenge threshold was lowered from $50 to $30, sending ~35% more transactions through 3DS authentication flows. This change coincided with dLocal's 3DS2 backend upgrade (March 1st), creating an interaction where the newly upgraded — and not fully stabilized — 3DS2 flow on dLocal's side is producing high failure rates. Error code `3D01` (3DS authentication failed/incomplete) accounts for 14.6% of all declines, indicating configuration-level failures rather than legitimate customer authentication issues.
- **Evidence:**
  - Data point 1: Transactions $30-50 dropped from 81.2% → 45.0% auth rate (-36.2pp); transactions $50+ dropped from 78.5% → 36.8% (-41.7pp). Meanwhile, transactions below $30 only dropped ~3pp.
  - Data point 2: 3DS Failed (timeout/error) transactions show only 33.6% approval rate across 2,360 attempts — these are technical failures, not customer abandonment.
  - Data point 3: Error code `3D01` represents 982 declines (14.6% of all declines) — this code is near-zero in a healthy integration and is almost always a configuration issue.
- **Affected Segments:**
  - Countries: All countries (most severe in dLocal-routed markets: Mexico, Colombia, Argentina, Chile)
  - Payment Methods: All card types; Visa/Mastercard Credit (-13pp), Visa Debit (-15pp), Mastercard Debit (-24pp), Amex (-45pp)
  - Acquirers: dLocal (primary impact due to 3DS2 upgrade interaction), Adyen (secondary)
  - Transaction Types: All transactions above $30 USD (subscriptions $29.99+ and consultation fees $30+)

### Root Cause #2: Chile Regulatory Compliance Failure — Estimated Impact: ~$77,088/month
- **Description:** On March 15th, Banco Central de Chile issued emergency guidance requiring all foreign merchants processing CLP payments to include a `chile_tax_id` field in transaction metadata. HealthFlow was unaware of this requirement, and non-compliant transactions are being systematically rejected with system error codes. This is entirely separate from the 3DS change (which occurred 11 days earlier).
- **Evidence:**
  - Data point 1: Chile's authorization rate collapsed to 21.4% (from 81.6% baseline — a 60.2pp drop), far exceeding any other country.
  - Data point 2: Chile error codes March 15-17 show 63.1% are code `96` (System error), not fraud or risk codes — this is a technical/regulatory rejection pattern.
  - Data point 3: The collapse started exactly on March 15th (the regulatory effective date), not March 4th (the 3DS change date), confirming a different root cause.
- **Affected Segments:**
  - Countries: Chile exclusively
  - Payment Methods: All card types processed in Chile
  - Acquirers: dLocal (exclusive processor for Chile)
  - Transaction Types: All transaction types in Chile

### Root Cause #3: Aggressive Retry Logic Causing Issuer Rate Limiting — Estimated Impact: ~$60,000-$84,000/month
- **Description:** Starting March 12th, HealthFlow's in-house retry logic began retrying soft declines within 60 seconds (industry best practice is 24-48 hours). This triggers issuer rate limits and increases fraud suspicion, creating a negative feedback loop: more declines → more retries → more rate limiting → elevated "Do not honor" responses.
- **Evidence:**
  - Data point 1: Error code `65` (Activity limit exceeded) accounts for 6.7% of all declines (448 transactions/week) — this was negligible in February.
  - Data point 2: Error code `05` ("Do not honor") jumped from ~18% to 42.3% of all declines, consistent with issuers becoming suspicious of HealthFlow's transaction velocity.
  - Data point 3: Application logs confirm the aggressive retry behavior started March 12th, correlating with the escalation of codes `65` and `05`.
- **Affected Segments:**
  - Countries: All countries (most visible in high-volume markets: Brazil, Mexico)
  - Payment Methods: All card types (soft-declined cards are being re-submitted too quickly)
  - Acquirers: Both dLocal and Adyen (issuer-side rate limiting, not acquirer-specific)
  - Transaction Types: Previously soft-declined recurring subscriptions and consultations

### Root Cause #4: Brazilian Issuer Policy Change — Estimated Impact: ~$64,000/month
- **Description:** On March 8th, Visa Brasil issued new guidance requiring "enhanced cardholder verification" for recurring subscription transactions above $25 USD equivalent. Major Brazilian issuers (Itaú, Bradesco, Banco do Brasil) implemented stricter fraud rules, disproportionately affecting HealthFlow's subscription charges. This is an external industry change, not a Yuno or HealthFlow configuration issue.
- **Evidence:**
  - Data point 1: Itaú auth rate dropped -15.1pp, Bradesco -15.9pp, Banco do Brasil -16.5pp — all starting around March 8th (not March 4th).
  - Data point 2: Nubank (digital-native, less conservative) only dropped -4.8pp, confirming this is an issuer policy issue, not a systemic technical problem.
  - Data point 3: These three banks represent ~57% of HealthFlow's Brazil volume (3,950 of 7,180 weekly attempts), amplifying the impact.
- **Affected Segments:**
  - Countries: Brazil
  - Payment Methods: Visa Credit and Mastercard Credit at Itaú, Bradesco, and Banco do Brasil
  - Acquirers: Adyen (primary Brazil acquirer)
  - Transaction Types: Recurring subscription charges above $25 USD equivalent

## 1.3 Impact Breakdown

### By Country
| Country | Auth Rate Before | Auth Rate Now | Volume Affected (weekly) | Revenue Impact (monthly) |
|---------|-----------------|---------------|--------------------------|--------------------------|
| Brazil | 84.2% | 74.9% | 7,180 attempts | ~$64,000 |
| Mexico | 82.1% | 69.8% | 6,314 attempts | ~$74,000 |
| Colombia | 80.5% | 65.0% | 3,481 attempts | ~$51,600 |
| Argentina | 79.8% | 64.0% | 2,140 attempts | ~$32,300 |
| Chile | 81.6% | 21.4% | 1,335 attempts | ~$77,100 |
| **Total** | **82.0%** | **67.0%** | **20,450 attempts** | **~$600K** |

### By Payment Method
| Payment Method | Auth Rate Before | Auth Rate Now | Volume Affected (weekly) | Revenue Impact (monthly) |
|----------------|-----------------|---------------|--------------------------|--------------------------|
| Visa Credit | 83.1% | 70.0% | 8,920 attempts | ~$112,000 |
| Mastercard Credit | 82.8% | 70.0% | 6,430 attempts | ~$79,200 |
| Visa Debit | 80.2% | 65.0% | 3,110 attempts | ~$45,000 |
| Mastercard Debit | 79.1% | 54.9% | 1,420 attempts | ~$33,000 |
| Amex | 78.9% | 33.8% | 340 attempts | ~$14,700 |
| PIX (Brazil) | 96.1% | 95.0% | 180 attempts | Negligible |
| OXXO (Mexico) | 95.8% | 96.0% | 50 attempts | None (slight improvement) |

### By Acquirer
| Acquirer | Auth Rate Before | Auth Rate Now | Volume Affected (weekly) | Revenue Impact (monthly) |
|----------|-----------------|---------------|--------------------------|--------------------------|
| dLocal (MEX, COL, ARG, CHL) | 81.4% | 62.7% | 13,270 attempts | ~$474,000 |
| Adyen (BRA + backup) | 84.2% | 74.9% | 7,180 attempts | ~$126,000 |

## 1.4 Timeline of Events
| Date | Event | Impact |
|------|-------|--------|
| Feb 20 | HealthFlow marketing launches aggressive referral campaign in Colombia and Argentina (+15% volume) | Increased transaction volume (not a decline driver, but added stress to system) |
| Feb 28 | dLocal sends notification: 3DS2 integration upgrade on March 1st, "No action required" | Precursor to 3DS interaction issue |
| Mar 1 | dLocal's 3DS2 upgrade goes live | Changed underlying 3DS2 processing behavior |
| Mar 4 | Ricardo (CTO) lowers Yuno 3DS threshold from $50 to $30 (deployed 3:00 PM UTC) | Auth rate drops from 82% to 75% within the week — 3DS now triggers on 35% more transactions through dLocal's newly upgraded (and unstable) 3DS2 flow |
| Mar 6 | Daniela notices auth rate at 76%, assumes temporary volatility | Missed early warning — no automated alerting in place |
| Mar 8 | Visa Brasil issues "enhanced cardholder verification" guidance for recurring subscriptions >$25 USD; Itaú, Bradesco, Banco do Brasil implement stricter fraud rules | Brazil auth rates drop further, especially at major traditional banks (-15pp at Itaú, Bradesco, BdB) |
| Mar 10 | Auth rate falls to 71%. Daniela escalates internally | Compounding effect of 3DS + Brazil issuer changes |
| Mar 12 | HealthFlow's in-house retry logic starts retrying soft declines within 60 seconds | Triggers issuer rate limiting (error 65) and elevates issuer suspicion (error 05). Creates negative feedback loop |
| Mar 15 | Chile regulatory change: `chile_tax_id` required in transaction metadata for all foreign merchants | Chile auth rate collapses from ~80% to 21.4%. Error code 96 (System error) dominates Chile declines at 63.1% |
| Mar 17 | CFO threatens to cancel Yuno contract. Daniela contacts TAM | Overall auth rate at 67%. Four compounding issues now active simultaneously |

---

# Section 2: Technical Action Plan

## 2.1 Priority Matrix

| Priority | Action | Owner | Expected Recovery | Timeline |
|----------|--------|-------|-------------------|----------|
| P0 - Critical | Revert 3DS threshold to $50 | HealthFlow (Ricardo) + Yuno TAM | +8-10 pp / ~$334K/month | Immediate (0-24 hours) |
| P0 - Critical | Fix Chile regulatory compliance (`chile_tax_id`) | HealthFlow + dLocal | +60 pp in Chile / ~$77K/month | Immediate (0-48 hours) |
| P0 - Critical | Disable aggressive retry logic | HealthFlow (Ricardo) | +2-3 pp / ~$72K/month | Immediate (0-24 hours) |
| P1 - High | Enrich Brazil transaction data for issuer compliance | HealthFlow + Adyen + Yuno | +5-8 pp in Brazil / ~$64K/month | This Week |
| P1 - High | Investigate dLocal 3DS2 upgrade compatibility | Yuno TAM + dLocal | Validates P0 fix / prevents recurrence | This Week |
| P2 - Medium | Implement smart routing (A/B test Adyen in non-Brazil markets) | Yuno + HealthFlow | +3-5 pp in MEX/COL/ARG | This Month |
| P2 - Medium | Accelerate PIX/OXXO/PSE adoption as card decline fallback | HealthFlow + Yuno | +$50-100K/month new revenue | This Month |
| P3 - Low | Implement network tokenization for recurring charges | HealthFlow + Yuno | +3-5 pp on recurring auth rates | Next Month |
| P3 - Low | Build automated auth rate monitoring and alerting | HealthFlow + Yuno | Prevents future delayed detection | Next Month |

## 2.2 Immediate Actions (0-48 hours)

### Action 1: Revert 3DS Threshold to $50 and Plan Risk-Based 3DS
- **Owner:** HealthFlow (Ricardo) with Yuno TAM guidance
- **Description:** Immediately revert the 3DS challenge threshold from $30 back to $50 via Yuno's API configuration. This will remove the 3DS authentication step from the $30-50 transaction range, which is currently seeing a 36.2pp drop in authorization rate.
- **Steps:**
  1. Update Yuno 3DS configuration to set the challenge threshold back to $50 USD
  2. Verify the change is propagated across all country configurations
  3. Monitor auth rates for $30-50 transactions over the next 24 hours to confirm recovery
  4. Begin scoping risk-based 3DS implementation (for future deployment) that exempts trusted returning subscribers while challenging first-time and high-risk transactions
- **Expected Impact:** Recover ~8-10 percentage points of overall authorization rate (~$334,400/month). The $30-50 range should recover from 45% back toward 80%+.
- **Risk/Dependencies:** Slightly increased fraud exposure in the $30-50 range (mitigated by the fact that HealthFlow operated at this threshold for months without issues). Coordinate with Ricardo carefully — frame as a temporary step while building a smarter 3DS strategy.
- **Success Metric:** Authorization rate for $30-50 transactions returns to >75% within 48 hours

### Action 2: Fix Chile Regulatory Compliance
- **Owner:** HealthFlow (engineering) + dLocal technical support
- **Description:** Add the required `chile_tax_id` field to all CLP transaction metadata to comply with Banco Central de Chile's March 15th mandate. Non-compliant transactions are being rejected with error code `96` (System error), causing Chile's auth rate to collapse to 21.4%.
- **Steps:**
  1. Contact dLocal's technical team to confirm the exact field name, format, and placement for `chile_tax_id` in the transaction payload
  2. HealthFlow engineering implements the field in their Yuno API integration for all Chile-bound transactions
  3. Test with a small batch of Chilean transactions to verify compliance
  4. Deploy to production and monitor Chile error code distribution — `96` errors should drop to near-zero
- **Expected Impact:** Recover Chile from 21.4% back toward 80%+ authorization rate (~$77,088/month recovered). Chile's ~1,335 weekly attempts should see ~1,040+ approvals instead of ~286.
- **Risk/Dependencies:** Depends on dLocal confirming the exact field specification. If HealthFlow does not have `chile_tax_id` data for their Chilean customers, they may need to collect it (Chilean RUT number) or use a default merchant tax ID.
- **Success Metric:** Chile authorization rate returns to >75% and error code `96` drops below 5% of Chile declines

### Action 3: Disable Aggressive Retry Logic
- **Owner:** HealthFlow (Ricardo / engineering team)
- **Description:** Immediately disable the in-house retry logic that retries soft declines within 60 seconds. Replace with an intelligent, code-aware retry strategy that respects issuer rate limits and uses appropriate timing based on decline reason codes.
- **Steps:**
  1. Disable the current retry mechanism that fires within 60 seconds of a soft decline
  2. Implement the following retry schedule based on decline codes:
     - `51` (Insufficient funds): retry after 24 hours, then 72 hours, then 7 days
     - `91` (Issuer unavailable/timeout): retry after 1 hour
     - `65` (Rate limit exceeded): do NOT retry for at least 24 hours
     - `05` (Do not honor): retry after 48 hours with enriched transaction data
     - Hard declines (`14`, `41`, `43`, `54`): **never retry** — flag card for update/dunning flow
  3. Add retry attempt counter — maximum 3 retries per transaction over 7 days
  4. Monitor error code `65` distribution to confirm rate limiting subsides
- **Expected Impact:** Eliminate error code `65` declines (~448/week) and reduce elevated `05` declines caused by issuer suspicion. Estimated recovery: +2-3pp overall (~$60,000-$84,000/month).
- **Risk/Dependencies:** Low risk. The current retry logic is actively harmful. Any delay in implementation means continued self-inflicted damage.
- **Success Metric:** Error code `65` drops below 1% of total declines; overall `05` code percentage decreases from 42% toward the historical ~18%

## 2.3 This Week Actions (3-7 days)

### Action 3: Enrich Brazil Transaction Data for Issuer Compliance
- **Owner:** HealthFlow (engineering) + Adyen technical team + Yuno TAM
- **Description:** In response to Visa Brasil's March 8th "enhanced cardholder verification" guidance, enrich the transaction data sent through Adyen for Brazilian recurring subscription charges. Providing richer context helps issuers (especially Itaú, Bradesco, Banco do Brasil) approve transactions they would otherwise decline under the new stricter rules.
- **Steps:**
  1. Add Merchant Initiated Transaction (MIT) indicators with proper recurring payment credentials to all Brazilian subscription charges — this tells issuers "the customer previously authorized this recurring billing"
  2. Include subscriber tenure (months active) and previous successful payment count in transaction metadata
  3. Send device fingerprint and IP consistency data where available
  4. Work with Adyen to request Visa's Recurring Payment indicator exemption for HealthFlow's subscription transactions
  5. For Itaú/Bradesco/Banco do Brasil BINs specifically: implement network token-based recurring credentials where available
- **Expected Impact:** Recover +5-8pp in Brazil authorization rate, particularly at Itaú (currently 70.0%), Bradesco (69.0%), and Banco do Brasil (67.0%). Estimated ~$64,000/month recovered.
- **Risk/Dependencies:** Requires Adyen's cooperation on exemption requests. MIT indicator implementation needs testing to ensure correct SCA (Strong Customer Authentication) signaling. Network token availability varies by issuer.
- **Success Metric:** Brazil overall auth rate recovers to >80%; Itaú, Bradesco, and BdB BIN-level auth rates recover to >78%

### Action 4: Investigate dLocal 3DS2 Upgrade Compatibility
- **Owner:** Yuno TAM + dLocal technical support
- **Description:** Open a technical investigation with dLocal regarding their March 1st 3DS2 upgrade and its interaction with HealthFlow's integration. The high volume of `3D01` errors (14.6% of all declines) and dLocal's disproportionate auth rate drop (-18.7pp vs Adyen's -9.3pp) suggest potential incompatibilities.
- **Steps:**
  1. Request dLocal's technical team provide detailed 3DS2 authentication flow logs for HealthFlow's transactions since March 1st
  2. Compare 3DS2 message format and field requirements before and after the March 1st upgrade
  3. Identify any new required fields, changed authentication flows, or timeout configurations that may be causing `3D01` failures
  4. If incompatibilities are found, coordinate fixes between HealthFlow's integration and dLocal's requirements
- **Expected Impact:** Validates and reinforces the P0 3DS threshold revert. If configuration mismatches are found and fixed, this could enable re-lowering the 3DS threshold in the future (with proper risk-based logic) without the auth rate penalty.
- **Risk/Dependencies:** Depends on dLocal's responsiveness. If dLocal's upgrade introduced a bug, the fix is on their side.
- **Success Metric:** `3D01` error code drops below 2% of total declines; dLocal's auth rate gap vs Adyen narrows to <5pp (excluding Chile)

## 2.4 This Month Actions (1-4 weeks)

### Action 5: Implement Smart Payment Routing
- **Owner:** Yuno TAM + HealthFlow (Daniela)
- **Description:** dLocal's overall auth rate (62.7%) is significantly underperforming Adyen (74.9%), even excluding Chile. Implement smart routing rules that A/B test Adyen as primary acquirer for non-Brazil markets (Mexico, Colombia, Argentina) and automatically route to the higher-performing acquirer based on real-time performance data.
- **Expected Impact:** +3-5pp authorization rate in Mexico, Colombia, and Argentina. Estimated $40,000-$80,000/month in recovered revenue. Additionally, having active dual-routing provides automatic failover if either acquirer experiences degradation.
- **Success Metric:** Non-Brazil markets achieve >78% auth rate; routing automatically shifts volume toward higher-performing acquirer per country/card type

### Action 6: Accelerate Alternative Payment Method Adoption
- **Owner:** HealthFlow (product + engineering) + Yuno TAM
- **Description:** PIX (95% auth rate) and OXXO (96% auth rate) are virtually unaffected by the card authorization issues. However, adoption is extremely low — PIX has only 180 attempts/week against 7,180 card attempts in Brazil. Implement APMs as a real-time fallback when card payments fail, and promote them in the checkout flow for applicable markets.
- **Expected Impact:** Converting even 10% of card declines to successful PIX/OXXO/PSE payments could recover $50,000-$100,000/month. Long-term, a healthier payment method mix reduces dependency on card network and issuer policies.
- **Success Metric:** PIX adoption reaches >15% of Brazil transaction volume; OXXO reaches >10% of Mexico volume; PSE launched and reaches >5% of Colombia volume within 30 days

### Action 7: Implement Network Tokenization for Recurring Charges
- **Owner:** HealthFlow (engineering) + Yuno
- **Description:** Replace raw card-on-file storage with network tokens (Visa Token Service, Mastercard MDES) for all recurring subscription charges. Network-tokenized transactions are trusted more by issuers and consistently show 3-5% higher authorization rates.
- **Expected Impact:** +3-5pp on recurring subscription auth rates across all markets. For HealthFlow's subscription-heavy model, this translates to ~$50,000-$100,000/month in additional approved transactions.
- **Success Metric:** >80% of recurring transactions use network tokens within 60 days; measurable auth rate lift of >2pp for tokenized vs. non-tokenized transactions

### Action 8: Build Automated Monitoring & Alerting System
- **Owner:** HealthFlow (Ricardo) + Yuno TAM (alerting configuration)
- **Description:** Implement automated monitoring to ensure authorization rate issues are detected within hours, not weeks. This incident went undetected for 6 days (March 4-10) and the Chile issue for 2 days (March 15-17).
- **Expected Impact:** Prevents future delayed detection. Early alerting on the Chile issue alone would have saved ~$38,000 (2 days of Chile losses).
- **Success Metric:** Alerts trigger within 4 hours of any country-level auth rate dropping >3pp day-over-day; alerts for any error code exceeding 10% of declines; weekly automated auth rate report by country/acquirer/payment method

## 2.5 Recovery Projection

| Timeframe | Expected Auth Rate | Cumulative Revenue Recovered |
|-----------|-------------------|------------------------------|
| After immediate fixes (48 hours) | ~78-80% | ~$470K/month (3DS revert + Chile fix + retry fix) |
| After week 1 | ~81-83% | ~$535K/month (+ Brazil data enrichment + dLocal investigation) |
| After month 1 | ~84-86% | ~$600K/month (+ smart routing + APM adoption) |
| Target steady state (month 2+) | ~86-88% | ~$650K+/month (exceeding pre-incident baseline via tokenization + optimized routing) |

---

# Section 3: Merchant-Facing Communication

## Email to Daniela Torres, Head of Payments at HealthFlow

**Subject:** HealthFlow Authorization Rate — Root Causes Identified, Recovery Plan in Progress

---

Dear Daniela,

Thank you for escalating this to us — I want to share our complete findings and the concrete recovery plan we've built.

### What Happened

Starting March 4th, your authorization rates declined from 82% to 67%, impacting approximately $600K in monthly transaction volume. The decline accelerated over two weeks because it was driven by four separate issues that compounded, not a single failure.

### Why It Happened

Our analysis identified **4 contributing factors**, each with distinct evidence and timing:

1. **3DS configuration change (March 4th) — largest impact (~55% of the decline):** The 3DS threshold lowered to $30 interacted poorly with a backend upgrade at dLocal (your primary acquirer), causing authentication failures on transactions above $30. This is the main driver and the fastest to fix.

2. **Chile regulatory requirement (March 15th) — Chile-specific (~13% of the decline):** A new Chilean regulation requires a specific tax ID field in payment metadata. Without it, transactions are being rejected as system errors — this is why Chile dropped to 21% and is unrelated to the 3DS change.

3. **Retry logic creating rate limits (March 12th) — self-inflicted (~12% of the decline):** Your retry system was re-attempting failed payments within 60 seconds, triggering issuer rate limits and fraud suspicion. This amplified the other issues.

4. **Brazilian bank policy change (March 8th) — external (~10% of the decline):** Three major Brazilian banks (Itaú, Bradesco, Banco do Brasil) tightened fraud rules for recurring subscriptions. This requires enriching the data we send to help these banks recognize your legitimate subscription charges.

### What We're Doing About It

We've prioritized a phased recovery plan:

1. **Reverting the 3DS threshold to $50** — In progress now. Expected to recover the bulk of lost authorization rate (~8-10pp) within 48 hours.
2. **Fixing Chile compliance** — Adding the required regulatory field. Should restore Chile from 21% back toward 80%+ within 48-72 hours.
3. **Replacing the retry logic** — Implementing intelligent, code-aware retry timing to stop triggering rate limits. Immediate deployment.
4. **Enriching Brazil transaction data** — Working with Adyen to add recurring payment credentials that satisfy the new bank requirements. Expected results within 1 week.
5. **Optimizing payment routing** — A/B testing acquirer performance by country to automatically route transactions to the best-performing processor.
6. **Accelerating PIX/OXXO adoption** — These methods have 95%+ approval rates and are immune to card-issuer issues. We recommend offering them as immediate fallbacks when card payments fail.

### Expected Recovery Timeline

- **By March 20th (48 hours):** Authorization rate should recover to ~78-80% (from current 67%)
- **By March 25th (1 week):** Target rate of ~82-83%, matching pre-incident performance
- **By April 18th (1 month):** Target rate of ~86-88%, **exceeding** pre-incident levels through routing optimization, tokenization, and APM expansion
- **Estimated revenue recovery:** ~$470K/month within 48 hours, full ~$600K/month recovery within 2 weeks, with potential to exceed baseline by $50K+/month

### How We'll Keep You Updated

- **Daily** status updates via Slack until auth rate recovers to 80%+
- **Twice-weekly** detailed progress reports with auth rate breakdowns by country, acquirer, and payment method
- Access to a real-time monitoring dashboard tracking all key metrics
- Dedicated point of contact: [Your Name], [email], [phone] — available for calls anytime this week

We take this situation seriously. Our analysis shows that switching providers would not resolve the underlying issues (the Chile regulation, Brazilian bank policies, and retry logic are provider-agnostic), and would reset the optimization work we can start immediately. We're committed to not only restoring your authorization rates but improving them beyond previous levels.

I'd welcome a call tomorrow to walk through these findings in detail with you and Ricardo. Would 10 AM work?

Best regards,
[Your Name]
Technical Account Manager | Yuno
[Contact Information]

---

# Section 4: Proactive Optimization Recommendations

## 4.1 Optimization Opportunities Overview

| Opportunity | Category | Estimated Impact | Effort | Priority |
|-------------|----------|-----------------|--------|----------|
| Risk-based 3DS (replace threshold-based) | Authentication | +5-8 pp for >$30 txns | Medium | 1 |
| Smart routing across acquirers | Routing | +3-5 pp in non-Brazil markets | Medium | 2 |
| PIX/OXXO/PSE as card decline fallback | Payment Method Mix | +$50-100K/month new revenue | Low-Medium | 3 |
| Network tokenization for recurring | Tokenization | +3-5 pp on recurring auth rates | High | 4 |
| Intelligent retry with dunning | Retry Logic | Recover 15-20% of soft declines | Medium | 5 |
| BIN-level routing optimization | Routing | +2-3 pp in Brazil | Medium | 6 |
| Automated monitoring & alerting | Operations | Prevents future delayed detection | Low | 7 |

## 4.2 Routing Optimization
- **Current State:** Brazil → Adyen (primary), dLocal (backup). All other countries → dLocal (primary), Adyen (backup). Static routing with no performance-based logic.
- **Recommendation:** Implement dynamic smart routing that considers real-time acquirer performance by country, card type, and BIN range. A/B test Adyen as primary in Mexico (where dLocal dropped 12.3pp) and Colombia (15.5pp drop). Implement automatic failover with a performance threshold — if an acquirer's auth rate drops >5pp below baseline for a segment, automatically shift volume.
- **Expected Impact:** +3-5pp authorization rate improvement in non-Brazil markets. Additionally, BIN-level routing in Brazil (e.g., routing Itaú/Bradesco BINs through an acquirer with better issuer relationships) could add +2-3pp.
- **Implementation:** Configure Yuno's smart routing engine with country-acquirer performance rules. Start with a 70/30 split (incumbent/challenger) and optimize based on 2 weeks of data.

## 4.3 Payment Method Mix Optimization
- **Current State:** Card transactions dominate (>97% of volume). PIX: 180 attempts/week (2.5% of Brazil), OXXO: 50 attempts/week (<1% of Mexico), PSE: not launched in Colombia.
- **Recommendation:** Implement a three-pronged APM strategy: (1) Add PIX, OXXO, and PSE as primary checkout options (not hidden behind card form), (2) Offer APMs as real-time fallback when card payments are declined, (3) For customers with repeated card declines, proactively suggest switching to APMs via email/in-app notification.
- **Expected Impact:** PIX at 95% auth rate and OXXO at 96% can dramatically reduce revenue loss from card declines. Converting just 10% of card declines to APM successes = ~$50,000-$100,000/month. Long-term, a 20-30% APM mix in Brazil/Mexico creates resilience against card issuer policy changes.
- **Key Markets:** Brazil (PIX — highest priority, largest market), Mexico (OXXO), Colombia (PSE)

## 4.4 Retry Logic Improvements
- **Current State:** In-house retry logic retries all soft declines within 60 seconds, with no differentiation by decline code. No retry attempt limits. No exponential backoff.
- **Recommendation:** Implement intelligent, decline-code-aware retry strategy with proper timing:
  - `51` (Insufficient funds): 24h → 72h → 7 days (customer may receive paycheck)
  - `91` (Issuer unavailable): 1 hour → 4 hours (transient issue)
  - `65` (Rate limited): 24 hours minimum (respect issuer limits)
  - `05` (Do not honor): 48 hours with enriched data (retry with additional context)
  - Hard declines: Never retry — trigger dunning flow (email customer to update card)
  - Maximum 3 retry attempts per transaction over 7 days
- **Expected Impact:** Recover 15-20% of soft declines (~$30,000-$50,000/month) while eliminating rate-limiting penalties
- **Decline Codes to Target:** `51` (highest recovery potential), `91` (transient, high success on retry), `05` (with enriched data)

## 4.5 Regional Strategy Enhancements
- **Current State:** dLocal handles all non-Brazil markets with no performance benchmarking. No local acquiring strategy. Chile lacks regulatory compliance monitoring.
- **Recommendation:** (1) Implement multi-acquirer capability in all markets for resilience and performance optimization. (2) Establish a regulatory monitoring process — subscribe to payment industry bulletins for all 5 markets (the Chile issue was published in an industry bulletin that HealthFlow never saw). (3) For Argentina specifically, consider adding a local acquirer that specializes in ARS transactions given the country's complex currency controls.
- **Expected Impact:** +3-5pp across non-Brazil markets through competitive routing; prevention of future regulatory compliance gaps (Chile-type incidents)
- **Markets to Prioritize:** Chile (regulatory monitoring), Mexico (largest dLocal market, A/B test with Adyen), Argentina (complex market, may benefit from specialized local acquirer)

## 4.6 Tokenization Strategy
- **Current State:** HealthFlow uses card-on-file tokens through Yuno. No network tokenization (Visa Token Service, Mastercard MDES) is in place.
- **Recommendation:** Implement network tokenization for all recurring subscription charges. Network tokens are provisioned directly by the card networks and are automatically updated when cards are renewed/replaced (reducing involuntary churn). Issuers trust network tokens more than raw card-on-file charges, resulting in higher approval rates.
- **Expected Impact:** +3-5pp on recurring subscription authorization rates. Additionally, automatic card updates via network tokenization reduce involuntary churn from expired cards (estimated 2-4% of subscription base per month).
- **Applicable Segments:** All recurring subscription charges across all markets and card networks (Visa, Mastercard). Amex tokenization available in Mexico and Brazil.

## 4.7 Cost Optimization
- **Current State:** Processing costs not analyzed in this incident review, but the current single-acquirer-per-market strategy provides no cost competition.
- **Recommendation:** Once smart routing is implemented, use acquirer competition to negotiate better rates. Route lower-risk, higher-volume transactions to the lower-cost acquirer while reserving the higher-performance acquirer for challenging segments (debit cards, high-value transactions). Additionally, PIX and OXXO typically have lower processing costs than card transactions — the APM strategy also reduces average cost per transaction.
- **Expected Savings:** Estimated $15,000-$30,000/month in processing cost optimization once dual-acquirer routing is active and APM volume increases.
- **Trade-offs:** Always prioritize authorization rate over processing cost — a declined transaction costs infinitely more than a few basis points in processing fees.

## 4.8 Additional Recommendations
- **3DS Optimization:** Replace the current threshold-based 3DS trigger with Yuno's risk-based 3DS engine. Exempt returning subscribers with >3 successful payments and consistent device/IP patterns. Challenge first-time purchases, high-value transactions, and flagged-risk profiles. Expected: reduce 3DS challenge rate from ~35% to ~10-15% of transactions while maintaining fraud protection.
- **Fraud Rule Tuning:** Review false positive rate on fraud rules. The jump in `05` codes may indicate overly aggressive velocity checks. Analyze the `05` declines to determine if legitimate subscribers are being flagged due to subscription renewal patterns.
- **Data-Driven Insights:** Establish a weekly authorization rate review cadence with Daniela's team. Provide a Yuno dashboard with real-time views by country, acquirer, payment method, 3DS status, and error code. Set up automated alerting for >3pp day-over-day drops at any segment level.

---

# Appendix

## A. Data Sources & Methodology
- Transaction data period: February 19 — March 17, 2025 (30 days)
- Total transactions analyzed: ~77,860 (4 weeks of data)
- Data sources: Yuno analytics dashboard (transaction-level data), dLocal operations email (3DS2 upgrade notification), Visa Brasil industry newsletter (enhanced verification guidance), Chilean payments industry bulletin (regulatory change), HealthFlow application logs (retry logic behavior), Git commit logs (3DS configuration change)
- Analysis methodology: Week-over-week authorization rate comparison segmented by country, payment method, acquirer, transaction amount, 3DS status, and error code. BIN-level analysis for Brazil to isolate issuer-specific behavior. Timeline correlation analysis to map each decline pattern to its specific causal event.

## B. Glossary
| Term | Definition |
|------|-----------|
| Authorization Rate | Percentage of attempted transactions approved by the issuer |
| pp | Percentage points (e.g., a drop from 82% to 67% is -15pp) |
| Soft Decline | Temporary decline that may succeed on retry (e.g., insufficient funds, rate limiting) |
| Hard Decline | Permanent decline that will never succeed with that card (e.g., stolen card, invalid number) |
| BIN | Bank Identification Number — first 6-8 digits of a card number identifying the issuing bank |
| 3DS/3DS2 | 3D Secure authentication protocol that adds identity verification to card payments |
| MIT | Merchant Initiated Transaction — a flag indicating the merchant is charging a pre-authorized recurring payment |
| Network Token | A card credential provisioned by Visa/Mastercard that replaces raw card numbers for recurring charges |
| APM | Alternative Payment Method — non-card payment options like PIX, OXXO, PSE |
| Dunning | The process of communicating with customers to update failed payment methods |
| SCA | Strong Customer Authentication — regulatory requirement for verifying cardholder identity |

## C. Supporting Data & Charts

### Error Code Distribution (March 11-17)
| Error Code | Description | Count | % of Declines | Category |
|------------|-------------|-------|---------------|----------|
| 05 | Do not honor (generic issuer decline) | 2,840 | 42.3% | Soft Decline |
| 51 | Insufficient funds | 1,104 | 16.4% | Soft Decline |
| 3D01 | 3DS authentication failed or incomplete | 982 | 14.6% | Configuration/Technical |
| N7 | CVV mismatch | 531 | 7.9% | Hard Decline |
| 65 | Activity limit exceeded (rate limiting) | 448 | 6.7% | Soft Decline |
| 14 | Invalid card number | 312 | 4.6% | Hard Decline |
| 91 | Issuer unavailable (timeout) | 289 | 4.3% | Soft Decline |
| 96 | System error | 142 | 2.1% | Configuration/Technical |
| **Total** | | **6,748** | **100%** | |

### Chile Error Code Breakdown (March 15-17 only)
| Error Code | Description | Count | % of Chile Declines |
|------------|-------------|-------|---------------------|
| 96 | System error | 128 | 63.1% |
| 05 | Do not honor | 42 | 20.7% |
| 51 | Insufficient funds | 22 | 10.8% |
| 91 | Issuer unavailable | 11 | 5.4% |

### Brazil BIN-Level Analysis (March 11-17)
| Issuer (BIN Range) | Attempts | Approved | Auth Rate | Feb Baseline | Change |
|--------------------|----------|----------|-----------|-------------|--------|
| Nubank (Mastercard) | 2,150 | 1,720 | 80.0% | 84.8% | -4.8pp |
| Itaú (Visa) | 1,680 | 1,176 | 70.0% | 85.1% | -15.1pp |
| Bradesco (Visa) | 1,290 | 890 | 69.0% | 84.9% | -15.9pp |
| Banco do Brasil (Mastercard) | 980 | 657 | 67.0% | 83.5% | -16.5pp |
| Santander (Visa) | 720 | 518 | 72.0% | 83.8% | -11.8pp |
| Other BINs | 360 | 288 | 80.0% | 83.0% | -3.0pp |

### 3DS Status Breakdown (March 11-17)
| 3DS Status | Attempts | Approved | Auth Rate |
|------------|----------|----------|-----------|
| 3DS Not Triggered | 13,200 | 10,560 | 80.0% |
| 3DS Challenged (user authenticated) | 4,890 | 2,348 | 48.0% |
| 3DS Failed (auth timeout or error) | 2,360 | 794 | 33.6% |
