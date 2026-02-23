# Section 3: Merchant-Facing Communication
### HealthFlow Authorization Collapse
### Prepared by: Carlos Medina, Technical Account Manager | Date: March 18, 2025 | Confidential

---

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
- Dedicated point of contact: Carlos Medina, [email], [phone] — available for calls anytime this week

We take this situation seriously. Our analysis shows that switching providers would not resolve the underlying issues (the Chile regulation, Brazilian bank policies, and retry logic are provider-agnostic), and would reset the optimization work we can start immediately. We're committed to not only restoring your authorization rates but improving them beyond previous levels.

I'd welcome a call tomorrow to walk through these findings in detail with you and Ricardo. Would 10 AM work?

Best regards,
Carlos Medina
Technical Account Manager | Yuno
