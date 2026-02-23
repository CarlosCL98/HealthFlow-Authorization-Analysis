# Section 4: Proactive Optimization Recommendations
### HealthFlow Authorization Collapse
### Prepared by: Carlos Medina, Technical Account Manager | Date: March 18, 2025 | Confidential

---

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

---

## 4.2 Routing Optimization
- **Current State:** Brazil → Adyen (primary), dLocal (backup). All other countries → dLocal (primary), Adyen (backup). Static routing with no performance-based logic.
- **Recommendation:** Implement dynamic smart routing that considers real-time acquirer performance by country, card type, and BIN range. A/B test Adyen as primary in Mexico (where dLocal dropped 12.3pp) and Colombia (15.5pp drop). Implement automatic failover with a performance threshold — if an acquirer's auth rate drops >5pp below baseline for a segment, automatically shift volume.
- **Expected Impact:** +3-5pp authorization rate improvement in non-Brazil markets. Additionally, BIN-level routing in Brazil (e.g., routing Itaú/Bradesco BINs through an acquirer with better issuer relationships) could add +2-3pp.
- **Implementation:** Configure Yuno's smart routing engine with country-acquirer performance rules. Start with a 70/30 split (incumbent/challenger) and optimize based on 2 weeks of data.

---

## 4.3 Payment Method Mix Optimization
- **Current State:** Card transactions dominate (>97% of volume). PIX: 180 attempts/week (2.5% of Brazil), OXXO: 50 attempts/week (<1% of Mexico), PSE: not launched in Colombia.
- **Recommendation:** Implement a three-pronged APM strategy: (1) Add PIX, OXXO, and PSE as primary checkout options (not hidden behind card form), (2) Offer APMs as real-time fallback when card payments are declined, (3) For customers with repeated card declines, proactively suggest switching to APMs via email/in-app notification.
- **Expected Impact:** PIX at 95% auth rate and OXXO at 96% can dramatically reduce revenue loss from card declines. Converting just 10% of card declines to APM successes = ~$50,000-$100,000/month. Long-term, a 20-30% APM mix in Brazil/Mexico creates resilience against card issuer policy changes.
- **Key Markets:** Brazil (PIX — highest priority, largest market), Mexico (OXXO), Colombia (PSE)

---

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

---

## 4.5 Regional Strategy Enhancements
- **Current State:** dLocal handles all non-Brazil markets with no performance benchmarking. No local acquiring strategy. Chile lacks regulatory compliance monitoring.
- **Recommendation:** (1) Implement multi-acquirer capability in all markets for resilience and performance optimization. (2) Establish a regulatory monitoring process — subscribe to payment industry bulletins for all 5 markets (the Chile issue was published in an industry bulletin that HealthFlow never saw). (3) For Argentina specifically, consider adding a local acquirer that specializes in ARS transactions given the country's complex currency controls.
- **Expected Impact:** +3-5pp across non-Brazil markets through competitive routing; prevention of future regulatory compliance gaps (Chile-type incidents)
- **Markets to Prioritize:** Chile (regulatory monitoring), Mexico (largest dLocal market, A/B test with Adyen), Argentina (complex market, may benefit from specialized local acquirer)

---

## 4.6 Tokenization Strategy
- **Current State:** HealthFlow uses card-on-file tokens through Yuno. No network tokenization (Visa Token Service, Mastercard MDES) is in place.
- **Recommendation:** Implement network tokenization for all recurring subscription charges. Network tokens are provisioned directly by the card networks and are automatically updated when cards are renewed/replaced (reducing involuntary churn). Issuers trust network tokens more than raw card-on-file charges, resulting in higher approval rates.
- **Expected Impact:** +3-5pp on recurring subscription authorization rates. Additionally, automatic card updates via network tokenization reduce involuntary churn from expired cards (estimated 2-4% of subscription base per month).
- **Applicable Segments:** All recurring subscription charges across all markets and card networks (Visa, Mastercard). Amex tokenization available in Mexico and Brazil.

---

## 4.7 Cost Optimization
- **Current State:** Processing costs not analyzed in this incident review, but the current single-acquirer-per-market strategy provides no cost competition.
- **Recommendation:** Once smart routing is implemented, use acquirer competition to negotiate better rates. Route lower-risk, higher-volume transactions to the lower-cost acquirer while reserving the higher-performance acquirer for challenging segments (debit cards, high-value transactions). Additionally, PIX and OXXO typically have lower processing costs than card transactions — the APM strategy also reduces average cost per transaction.
- **Expected Savings:** Estimated $15,000-$30,000/month in processing cost optimization once dual-acquirer routing is active and APM volume increases.
- **Trade-offs:** Always prioritize authorization rate over processing cost — a declined transaction costs infinitely more than a few basis points in processing fees.

---

## 4.8 Additional Recommendations
- **3DS Optimization:** Replace the current threshold-based 3DS trigger with Yuno's risk-based 3DS engine. Exempt returning subscribers with >3 successful payments and consistent device/IP patterns. Challenge first-time purchases, high-value transactions, and flagged-risk profiles. Expected: reduce 3DS challenge rate from ~35% to ~10-15% of transactions while maintaining fraud protection.
- **Fraud Rule Tuning:** Review false positive rate on fraud rules. The jump in `05` codes may indicate overly aggressive velocity checks. Analyze the `05` declines to determine if legitimate subscribers are being flagged due to subscription renewal patterns.
- **Data-Driven Insights:** Establish a weekly authorization rate review cadence with Daniela's team. Provide a Yuno dashboard with real-time views by country, acquirer, payment method, 3DS status, and error code. Set up automated alerting for >3pp day-over-day drops at any segment level.
