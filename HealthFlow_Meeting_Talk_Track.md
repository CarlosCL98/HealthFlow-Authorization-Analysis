# Meeting Talk Track: HealthFlow Authorization Recovery
### Prepared by: Carlos Medina, Technical Account Manager | Date: March 18, 2025
### Attendees: Daniela Torres (Head of Payments), Ricardo Mendoza (CTO)

---

## Opening (1 minute)

> "Daniela, Ricardo — thank you for your time. I've completed a full analysis of the authorization decline. The good news: we've identified exactly what's happening, and **the majority of the impact is recoverable within 48 hours**. The key takeaway is this: you don't have one problem — you have four separate ones that hit within the same two-week window, which is why it looked like a single cascading failure."

---

## The 4 Problems — Simple Version (5 minutes)

### Problem 1 — The 3DS change hit at the wrong time (biggest impact: ~$335K/month)

> "Ricardo, your instinct to lower the 3DS threshold was actually sound — tighter authentication does reduce fraud. The issue is timing. dLocal upgraded their 3DS2 backend on March 1st. When you lowered the threshold on March 4th, you pushed 35% more transactions through a system that wasn't fully stable yet. The proof? Transactions under $30 are fine — only 3% drop. Transactions over $30 collapsed by 36 points. That tells us it's the 3DS trigger, not your customers."

### Problem 2 — Chile broke for a completely different reason (~$77K/month)

> "Chile dropped to 21% — but this has nothing to do with 3DS. On March 15th, Chile's central bank started requiring a tax ID field in every foreign merchant transaction. You weren't notified, and without that field, 63% of Chile declines are coming back as 'system error.' Once we add that field, Chile recovers immediately."

### Problem 3 — Your retry logic is making things worse (~$72K/month)

> "When declines started rising, your system started retrying failed payments within 60 seconds. That's way too fast — issuers see rapid retries as suspicious behavior. They started rate-limiting you and flagging your transactions. This turned a bad situation into a worse one. The fix is simple: smarter retry timing based on why the payment was declined."

### Problem 4 — Brazilian banks changed their rules (~$64K/month)

> "Three major Brazilian banks — Itaú, Bradesco, Banco do Brasil — tightened their fraud rules for recurring subscriptions on March 8th. Nubank barely changed (-5%), but these traditional banks dropped 15-16 points. We need to send them richer data — subscriber history, recurring payment flags — so they recognize your charges as legitimate."

---

## The Fix — What Happens Next (3 minutes)

> "Here's the recovery plan in three phases:"

| When | What we do | Expected result |
|------|-----------|----------------|
| **Next 48 hours** | Revert 3DS to $50, fix Chile tax ID field, stop aggressive retries | Back to ~78-80% (recover ~$470K/month) |
| **This week** | Enrich Brazil transaction data, investigate dLocal's 3DS2 upgrade | Back to ~82-83% (match pre-incident) |
| **This month** | Smart routing, PIX/OXXO expansion, network tokenization | Target 86-88% (**exceed** pre-incident) |

---

## Addressing the Elephant in the Room (1 minute)

> "I know the question of switching providers has come up. I want to be direct: two of these four problems — Chile's regulation and the Brazilian bank policy change — would follow you to any provider. The retry issue is in your codebase. And the 3DS interaction, which is the biggest one, we can fix today. Switching resets the clock. Staying lets us fix this within days and actually come out stronger — with better routing, better retry logic, and payment methods like PIX that give you 95% approval rates regardless of what card issuers do."

---

## Close (1 minute)

> "So to summarize: four problems, four fixes, most of the revenue recovers in 48 hours. I'll send daily Slack updates until we're back above 80%, then twice-weekly reports. Can we align on getting Ricardo's team started on the 3DS revert and retry logic changes today?"

---

## Quick Reference Card (for TAM during the meeting)

### If Daniela asks "How did we miss this for two weeks?"
> "Two reasons: no automated alerting was in place, and the problems stacked gradually — the first drop on March 4th looked like normal volatility, then each new issue added on top before the previous one was investigated. One of our recommendations is building automated alerts so a 3-point drop in any country triggers an immediate review."

### If Ricardo pushes back on reverting the 3DS change
> "We're not saying 3DS at $30 is wrong — we're saying the current implementation method needs to change. Once we confirm dLocal's 3DS2 upgrade is stable and implement risk-based 3DS (which exempts trusted returning subscribers), we can protect even more transactions than a flat $30 threshold would. The revert is temporary while we build the smarter version."

### If the CFO asks about the Yuno contract
> "The total impact is ~$600K/month. Our plan recovers ~$470K within 48 hours, the full amount within 2 weeks, and positions HealthFlow to exceed pre-incident performance by $50K+/month through routing and APM optimization. A provider switch would take 3-6 months to implement and wouldn't resolve 3 of the 4 root causes."

### Key numbers to have ready
| Metric | Value |
|--------|-------|
| Current auth rate | 67.0% |
| Pre-incident auth rate | 82.0% |
| Total monthly revenue impact | ~$600K |
| 48-hour recovery target | 78-80% |
| 1-week recovery target | 82-83% |
| 1-month optimization target | 86-88% |
| Chile current auth rate | 21.4% |
| PIX auth rate (unaffected) | 95.0% |
| OXXO auth rate (unaffected) | 96.0% |
