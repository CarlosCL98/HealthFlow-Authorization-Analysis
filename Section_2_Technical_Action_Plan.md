# Section 2: Technical Action Plan
### HealthFlow Authorization Collapse
### Prepared by: Carlos Medina, Technical Account Manager | Date: March 18, 2025 | Confidential

---

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

---

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

---

## 2.3 This Week Actions (3-7 days)

### Action 4: Enrich Brazil Transaction Data for Issuer Compliance
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

### Action 5: Investigate dLocal 3DS2 Upgrade Compatibility
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

---

## 2.4 This Month Actions (1-4 weeks)

### Action 6: Implement Smart Payment Routing
- **Owner:** Yuno TAM + HealthFlow (Daniela)
- **Description:** dLocal's overall auth rate (62.7%) is significantly underperforming Adyen (74.9%), even excluding Chile. Implement smart routing rules that A/B test Adyen as primary acquirer for non-Brazil markets (Mexico, Colombia, Argentina) and automatically route to the higher-performing acquirer based on real-time performance data.
- **Expected Impact:** +3-5pp authorization rate in Mexico, Colombia, and Argentina. Estimated $40,000-$80,000/month in recovered revenue. Additionally, having active dual-routing provides automatic failover if either acquirer experiences degradation.
- **Success Metric:** Non-Brazil markets achieve >78% auth rate; routing automatically shifts volume toward higher-performing acquirer per country/card type

### Action 7: Accelerate Alternative Payment Method Adoption
- **Owner:** HealthFlow (product + engineering) + Yuno TAM
- **Description:** PIX (95% auth rate) and OXXO (96% auth rate) are virtually unaffected by the card authorization issues. However, adoption is extremely low — PIX has only 180 attempts/week against 7,180 card attempts in Brazil. Implement APMs as a real-time fallback when card payments fail, and promote them in the checkout flow for applicable markets.
- **Expected Impact:** Converting even 10% of card declines to successful PIX/OXXO/PSE payments could recover $50,000-$100,000/month. Long-term, a healthier payment method mix reduces dependency on card network and issuer policies.
- **Success Metric:** PIX adoption reaches >15% of Brazil transaction volume; OXXO reaches >10% of Mexico volume; PSE launched and reaches >5% of Colombia volume within 30 days

### Action 8: Implement Network Tokenization for Recurring Charges
- **Owner:** HealthFlow (engineering) + Yuno
- **Description:** Replace raw card-on-file storage with network tokens (Visa Token Service, Mastercard MDES) for all recurring subscription charges. Network-tokenized transactions are trusted more by issuers and consistently show 3-5% higher authorization rates.
- **Expected Impact:** +3-5pp on recurring subscription auth rates across all markets. For HealthFlow's subscription-heavy model, this translates to ~$50,000-$100,000/month in additional approved transactions.
- **Success Metric:** >80% of recurring transactions use network tokens within 60 days; measurable auth rate lift of >2pp for tokenized vs. non-tokenized transactions

### Action 9: Build Automated Monitoring & Alerting System
- **Owner:** HealthFlow (Ricardo) + Yuno TAM (alerting configuration)
- **Description:** Implement automated monitoring to ensure authorization rate issues are detected within hours, not weeks. This incident went undetected for 6 days (March 4-10) and the Chile issue for 2 days (March 15-17).
- **Expected Impact:** Prevents future delayed detection. Early alerting on the Chile issue alone would have saved ~$38,000 (2 days of Chile losses).
- **Success Metric:** Alerts trigger within 4 hours of any country-level auth rate dropping >3pp day-over-day; alerts for any error code exceeding 10% of declines; weekly automated auth rate report by country/acquirer/payment method

---

## 2.5 Recovery Projection

| Timeframe | Expected Auth Rate | Cumulative Revenue Recovered |
|-----------|-------------------|------------------------------|
| After immediate fixes (48 hours) | ~78-80% | ~$470K/month (3DS revert + Chile fix + retry fix) |
| After week 1 | ~81-83% | ~$535K/month (+ Brazil data enrichment + dLocal investigation) |
| After month 1 | ~84-86% | ~$600K/month (+ smart routing + APM adoption) |
| Target steady state (month 2+) | ~86-88% | ~$650K+/month (exceeding pre-incident baseline via tokenization + optimized routing) |
