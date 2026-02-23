# Section 1: Root Cause Analysis
### HealthFlow Authorization Collapse
### Prepared by: Carlos Medina, Technical Account Manager | Date: March 18, 2025 | Confidential

---

## 1.1 Executive Summary of the Problem

| Metric | Before Incident | Current | Delta |
|--------|----------------|---------|-------|
| Overall Authorization Rate | 82.0% | 67.0% | -15.0 pp |
| Monthly Transaction Volume | ~$2.8M | ~$2.19M | -~$610K |
| Affected Transaction Count (weekly) | — | ~6,748 declines | — |
| Estimated Revenue Impact | — | ~$600K/month | — |

> **Key Finding:** The authorization collapse is not a single issue — it is **four distinct, compounding problems** that emerged within a two-week window: a 3DS threshold misconfiguration interacting with dLocal's 3DS2 upgrade, a Chile regulatory compliance failure, an aggressive retry logic triggering issuer rate limits, and a Brazilian issuer policy change affecting major banks. Each requires its own targeted fix.

---

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

---

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

---

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
