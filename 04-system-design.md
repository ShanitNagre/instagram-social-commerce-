# System Design: ShopReel AI Commerce Pipeline
**Author:** Shanit Nagre — AI Product Manager  
**Date:** May 2026  
**Type:** Product-level system design (PM perspective)

---

## Overview

This document describes the system architecture for ShopReel — Instagram's AI-powered in-Reel commerce feature. Written from a PM perspective: focused on data flows, key design decisions, failure modes, and trade-offs rather than implementation specifics.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CONTENT LAYER                            │
│   Creator uploads Reel → CDN → Video Processing Pipeline        │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    AI IDENTIFICATION LAYER                       │
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │ Frame        │    │ Object       │    │ Brand/Logo       │   │
│  │ Extraction   │───▶│ Detection    │───▶│ Recognition      │   │
│  │ (2s interval)│    │ (Llama Vision│    │ (Meta Brand DB)  │   │
│  └──────────────┘    └──────────────┘    └────────┬─────────┘   │
│                                                   │             │
│                      ┌────────────────────────────▼──────────┐  │
│                      │  Attribute Extraction                  │  │
│                      │  Color · Style · Category · Material   │  │
│                      └────────────────────────────┬──────────┘  │
└───────────────────────────────────────────────────┼─────────────┘
                                                    │
                                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MATCHING LAYER                                │
│                                                                  │
│  ┌─────────────────┐    ┌──────────────────────────────────┐    │
│  │ Meta Commerce   │    │ Vector Similarity Search          │    │
│  │ Catalog         │◀───│ (2B+ products indexed)            │    │
│  │ (2B+ products)  │    │ Confidence threshold: ≥ 88%       │    │
│  └────────┬────────┘    └──────────────────────────────────┘    │
│           │                                                      │
│           ▼                                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Price + Availability Check (real-time, <200ms)         │    │
│  │  Seller verification · Brand safety filter              │    │
│  └─────────────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────┬─────────────┘
                                                    │
                                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                 PERSONALIZATION LAYER                            │
│                                                                  │
│  ┌────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │ Purchase       │  │ Browse/Engage   │  │ Social Graph    │  │
│  │ History        │  │ History         │  │ (what friends   │  │
│  │                │  │                 │  │  bought)        │  │
│  └────────┬───────┘  └────────┬────────┘  └────────┬────────┘  │
│           └──────────────────┬┘                    │            │
│                              ▼                     │            │
│           ┌──────────────────────────────────────┐ │            │
│           │  Ranking Model                        │◀┘            │
│           │  → Price range fit                    │             │
│           │  → Brand affinity                     │             │
│           │  → Style match                        │             │
│           │  → Purchase probability score         │             │
│           └──────────────────────────────────────┘             │
└───────────────────────────────────────────────────┬─────────────┘
                                                    │
                                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                      UX DELIVERY LAYER                          │
│                                                                  │
│  Shop Dot Indicator → Product Shelf (half-sheet) → Meta Pay     │
│                                                                  │
│  Creator Dashboard → Commission Attribution → Weekly Payout     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Design Decisions

### Decision 1: Frame Extraction Frequency — 2 seconds

**Options considered:**
- Every frame (30fps): Too compute-intensive, 15× more inference calls
- Every 5 seconds: Might miss quick product appearances
- Every 2 seconds: Balances coverage with compute cost

**Trade-off:** A product visible for only 1 second won't be tagged. Acceptable — products need to be clearly visible to be purchasable.

**Compute implication:** 1B Reels/day × avg 30s = 15B frames at 2s intervals = 7.5B inference calls/day. Requires significant GPU cluster — batched processing during upload, not real-time.

---

### Decision 2: Confidence Threshold — 88%

**Why 88%, not higher?**
- At 95%: Too many products missed, creator value drops, GMV suffers
- At 80%: Too many wrong products surfaced, user trust damaged
- At 88%: ~8% false positive rate — acceptable for "Report wrong product" fallback

**Failure mode:** Wrong product surfaced → user taps → sees wrong item → reports → product removed from that Reel within 60 seconds via moderation queue.

**Monitoring:** Track false positive rate daily by category. If fashion drops below 90% accuracy, retrain on fashion-specific data.

---

### Decision 3: Processing Timing — Upload-time, not Stream-time

**Option A: Process at upload time**
- AI runs when creator uploads Reel
- Products pre-tagged before distribution
- Latency: 30–90 seconds delay before Reel goes live
- ✅ Better accuracy (can run multiple passes)
- ✅ No real-time inference cost at view time

**Option B: Process at view time (streaming)**
- AI runs as user watches
- Real-time product identification
- ✅ No upload delay
- ❌ Massive compute cost at scale (2B users × multiple Reels)
- ❌ Latency visible to user

**Decision: Upload-time processing** for recorded Reels. Real-time for Live Commerce (Phase 2, separate pipeline).

---

### Decision 4: Catalog Architecture — Federated, Not Centralised

**Problem:** 2B+ products across thousands of brands/sellers. Centralized catalog would take years to build and maintain.

**Solution:** Federated catalog using existing Meta Commerce infrastructure:
- Brands/sellers already list products via Meta Ads / Facebook Shops
- Product catalog automatically available to ShopReel matching engine
- New sellers onboard via existing Meta Commerce onboarding
- Third-party catalogs (Shopify, WooCommerce) via existing API integrations

**Key insight:** Instagram doesn't need to build a new catalog. It needs to connect its AI layer to the catalog it already has.

---

### Decision 5: Creator Attribution — Deterministic, Not Probabilistic

**The problem:** If a user watches a Reel, sees a product, doesn't buy immediately, then buys 3 days later — does the creator get commission?

**Options:**
- 0-hour window: Only immediate purchases attributed (simple, unfair to creators)
- 30-day window: Standard affiliate model (complex, potential for fraud)
- 7-day window: Balanced (industry standard for social commerce)

**Decision: 7-day attribution window** using click-based tracking. If user taps the product shelf in a creator's Reel and purchases within 7 days, creator gets commission.

**Fraud prevention:** Rate limiting on creator account taps, anomaly detection on commission spikes, minimum 3-day account age for commission eligibility.

---

## Data Flows

### Purchase Flow (Happy Path)
```
User watches Reel
      ↓
AI detects product (upload-time, pre-processed)
      ↓
Shop dot appears (bottom-left, after 2s of product visibility)
      ↓
User taps Shop dot
      ↓
Product shelf loads (<1.2s, cached product data)
      ↓
User taps "Buy Now"
      ↓
Meta Pay checkout (pre-filled for returning users)
      ↓
Order confirmed
      ↓
Creator commission recorded (7-day attribution)
      ↓
Seller notified → fulfillment
```

### Failure Flows

**Wrong product identified:**
```
User taps product → Sees wrong item
      ↓
Taps "Report" → Reason: "Wrong product"
      ↓
Product tag removed from Reel (within 60s via auto-moderation)
      ↓
Logged for model retraining queue
```

**Product out of stock:**
```
Product shelf loads → Real-time availability check
      ↓
Out of stock detected
      ↓
Show "Similar items" from same brand/category
      ↓
Log miss rate for catalog completeness reporting
```

**Payment failure:**
```
Meta Pay checkout fails
      ↓
Show alternative payment methods
      ↓
Offer "Save for later" → Wishlist
      ↓
Push notification when product is back in stock
```

---

## Scaling Considerations

### Compute
- Frame extraction: Distributed processing cluster, batched at upload
- Vector search: Approximate nearest neighbor (ANN) for <50ms matching at 2B product scale
- Product shelf: CDN-cached product data, refreshed every 15 minutes for pricing

### Storage
- Product identification metadata stored per-Reel (not per-frame)
- Creator commission ledger: append-only event log, reconciled weekly
- Purchase attribution: 7-day rolling window, purged after reconciliation

### Reliability
- Product shelf loads independently of Reel playback — shelf failure doesn't affect video
- Graceful degradation: if matching fails, no shop dot (silent failure, not visible to user)
- Commerce is additive — core Reels experience is never dependent on commerce working

---

## Privacy & Compliance

| Requirement | Approach |
|-------------|----------|
| GDPR (EU) | Purchase behavior used only with explicit commerce opt-in |
| CCPA (California) | "Do Not Share" toggle disables personalization |
| DPDP (India) | Consent management framework for behavioral data |
| Minors | No commerce features for users under 18 |
| Data retention | Purchase history retained 24 months, then anonymized |

---

## PM Metrics for System Health

These are the metrics I'd track daily as the PM owning this system:

| Metric | Alert Threshold | Action |
|--------|----------------|--------|
| Product identification accuracy | < 85% | Pause AI scanner for affected category |
| Shelf load time p95 | > 2s | Page CDN team |
| False positive rate | > 10% | Raise confidence threshold |
| Checkout success rate | < 88% | Page payments team |
| Creator commission error rate | > 0.1% | Freeze payouts, audit |
| Reel completion rate delta | > -5% | Reduce shop dot aggressiveness |

---

*Shanit Nagre · AI Product Manager · [shanitnagre.github.io](https://shanitnagre.github.io)*
