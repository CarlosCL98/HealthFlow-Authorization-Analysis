# HealthFlow Authorization Collapse — Technical Analysis & Action Plan

**Prepared by:** Carlos Medina, Technical Account Manager at Yuno
**Date:** March 18, 2025
**Status:** Active incident — recovery in progress

---

## Context

HealthFlow is a telehealth subscription platform operating across Mexico, Colombia, Brazil, Argentina, and Chile, processing ~$2.8M/month in recurring subscriptions and one-time consultation fees.

Between March 4-17, 2025, authorization rates dropped from **82% to 67%**, resulting in an estimated **~$600K/month in lost revenue**. This repository contains the full technical analysis, root cause diagnosis, action plan, and merchant communication materials.

---

## Root Causes Identified

| # | Root Cause | Impact | Timeline to Fix |
|---|-----------|--------|-----------------|
| 1 | 3DS threshold misconfiguration + dLocal 3DS2 upgrade interaction | ~$335K/month | 0-24 hours |
| 2 | Chile regulatory compliance failure (`chile_tax_id`) | ~$77K/month | 0-48 hours |
| 3 | Aggressive retry logic triggering issuer rate limits | ~$72K/month | 0-24 hours |
| 4 | Brazilian issuer policy change (Visa Brasil enhanced verification) | ~$64K/month | 1-2 weeks |

---

## Recovery Projection

| Timeframe | Expected Auth Rate | Revenue Recovered |
|-----------|-------------------|-------------------|
| After 48 hours | ~78-80% | ~$470K/month |
| After 1 week | ~82-83% | ~$535K/month |
| After 1 month | ~84-86% | ~$600K/month |
| Steady state (month 2+) | ~86-88% | ~$650K+/month |

---

## Repository Structure

| File | Description |
|------|-------------|
| [`HealthFlow_Authorization_Analysis_Template.md`](HealthFlow_Authorization_Analysis_Template.md) | Full consolidated analysis (all sections in one document) |
| [`HealthFlow_Meeting_Talk_Track.md`](HealthFlow_Meeting_Talk_Track.md) | Meeting summary, talk track, and stakeholder Q&A prep |
| [`Section_1_Root_Cause_Analysis.md`](Section_1_Root_Cause_Analysis.md) | Detailed root cause diagnosis, impact breakdowns by country/method/acquirer, and event timeline |
| [`Section_2_Technical_Action_Plan.md`](Section_2_Technical_Action_Plan.md) | Priority matrix, 9 action items (P0-P3), and recovery projection |
| [`Section_3_Merchant_Communication.md`](Section_3_Merchant_Communication.md) | Merchant-facing email to Head of Payments |
| [`Section_4_Proactive_Optimization.md`](Section_4_Proactive_Optimization.md) | 7 optimization opportunities: routing, APMs, tokenization, retry logic, regional strategy, cost |
| [`Section_5_Appendix.md`](Section_5_Appendix.md) | Data sources, methodology, glossary, and all supporting data tables |

---

## Key Metrics at a Glance

- **Current Auth Rate:** 67.0% (target: 86-88%)
- **Worst Affected Market:** Chile at 21.4% (regulatory issue)
- **Worst Affected Payment Method:** Amex at 33.8%
- **Worst Affected Acquirer:** dLocal at 62.7% (-18.7pp)
- **Unaffected Methods:** PIX 95.0%, OXXO 96.0%
