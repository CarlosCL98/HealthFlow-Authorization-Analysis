# Appendix
### HealthFlow Authorization Collapse
### Prepared by: Carlos Medina, Technical Account Manager | Date: March 18, 2025 | Confidential

---

## A. Data Sources & Methodology
- **Transaction data period:** February 19 — March 17, 2025 (30 days)
- **Total transactions analyzed:** ~77,860 (4 weeks of data)
- **Data sources:**
  - Yuno analytics dashboard (transaction-level data)
  - dLocal operations email (3DS2 upgrade notification)
  - Visa Brasil industry newsletter (enhanced verification guidance)
  - Chilean payments industry bulletin (regulatory change)
  - HealthFlow application logs (retry logic behavior)
  - Git commit logs (3DS configuration change)
- **Analysis methodology:** Week-over-week authorization rate comparison segmented by country, payment method, acquirer, transaction amount, 3DS status, and error code. BIN-level analysis for Brazil to isolate issuer-specific behavior. Timeline correlation analysis to map each decline pattern to its specific causal event.

---

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
| Acquirer | Financial institution that processes card payments on behalf of the merchant (e.g., dLocal, Adyen) |
| Issuer | Bank that issued the customer's credit/debit card (e.g., Itaú, Nubank, Bradesco) |
| CLP | Chilean Peso — official currency of Chile |
| RUT | Rol Único Tributario — Chilean tax identification number |

---

## C. Supporting Data & Charts

### Overall Authorization Rates by Week
| Week | Attempts | Approved | Auth Rate | Change vs. Previous |
|------|----------|----------|-----------|---------------------|
| Feb 19-25 | 18,420 | 15,104 | 82.0% | — |
| Feb 26-Mar 3 | 19,103 | 15,654 | 81.9% | -0.1% |
| Mar 4-10 | 19,887 | 14,920 | 75.0% | -6.9% |
| Mar 11-17 | 20,450 | 13,702 | 67.0% | -8.0% |

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

### Authorization Rates by Transaction Amount (March 11-17)
| Amount Range | Attempts | Approved | Auth Rate | Feb Baseline | Change |
|-------------|----------|----------|-----------|-------------|--------|
| $0-15 | 4,280 | 3,424 | 80.0% | 83.1% | -3.1pp |
| $15-30 | 8,920 | 7,136 | 80.0% | 82.6% | -2.6pp |
| $30-50 | 5,780 | 2,601 | 45.0% | 81.2% | -36.2pp |
| $50+ | 1,470 | 541 | 36.8% | 78.5% | -41.7pp |

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

### Authorization Rates by Acquirer (March 11-17)
| Acquirer | Country Distribution | Attempts | Approved | Auth Rate | Feb Baseline | Change |
|----------|---------------------|----------|----------|-----------|-------------|--------|
| dLocal | MEX, COL, ARG, CHL | 13,270 | 8,325 | 62.7% | 81.4% | -18.7pp |
| Adyen | BRA (+ backup) | 7,180 | 5,377 | 74.9% | 84.2% | -9.3pp |
