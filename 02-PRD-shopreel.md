# PRD: Instagram AI Social Commerce — ShopReel
**Author:** Shanit Nagre — AI Product Manager  
**Date:** May 2026  
**Status:** Concept Spec  
**Version:** 1.0

---

## 📌 One-Line Pitch
*Turn every Reel into a shoppable moment — AI identifies products in real-time, surfaces purchase options natively, and completes checkout without leaving Instagram.*

---

## 🎯 Problem Statement

### The Broken Purchase Journey
Instagram drives more purchase intent than any other social platform. But the path from "I want that" to "I bought that" is broken:

**Current flow (7 steps, ~3 minutes):**
Pause Reel → Go to profile → Find link in bio → Open external browser → Search product → Add to cart → Checkout

**Result:** 70% product discovery rate → 13% purchase conversion. 57% of purchase intent is lost to friction.

**TikTok's answer:** In-video shopping with one-tap checkout. $20B GMV in year 2.
**Instagram's answer:** A Shop tab nobody visits.

### Why Now
1. TikTok Shop is accelerating — projected $50B GMV by 2026
2. Meta's Llama vision models can identify products in video frames in real-time
3. Instagram has payment infrastructure (Meta Pay) but hasn't integrated it into Reels
4. Creator monetization is a top platform priority — commerce solves it directly
5. YouTube Shopping launched 2024 — the window to lead is closing

---

## 👥 Users

### Primary: Impulse Buyer (18–34, fashion/beauty/lifestyle)
- Watches 45+ min Reels/day
- Buys 2–4 products/month based on social discovery
- **Pain:** "By the time I find the product, I've lost the feeling"
- **Goal:** Buy the thing I just saw, immediately
- **Success:** "I bought it without leaving the video"

### Secondary: Creator (10K–1M followers)
- Earns through brand deals, affiliate links
- **Pain:** Link in bio converts poorly, brand deals are inconsistent income
- **Goal:** Earn directly from content without complex affiliate setups
- **Success:** "Every Reel I post is a storefront"

### Tertiary: Brand / Merchant
- Sells via Instagram but struggles to attribute sales to specific content
- **Pain:** No direct commerce integration, poor attribution
- **Goal:** Convert Instagram viewers into buyers at scale
- **Success:** "I can see exactly which Reel drove which sale"

---

## ✅ Goals & Success Metrics

### North Star
**Instagram Commerce GMV** — total transaction value processed through ShopReel

### Primary Metrics

| Metric | Baseline | Target (12 months) |
|--------|----------|-------------------|
| Reel-to-purchase conversion rate | 1.2% | ≥ 5% |
| Commerce GMV | ~$4B/yr | $15B/yr |
| Creator commerce revenue | — | $2B/yr |
| Avg purchase value | — | $48 |
| ShopReel-enabled Reels % | 0% | 40% of all Reels |

### Secondary Metrics
| Metric | Target |
|--------|--------|
| Time from product view to purchase | < 45 seconds |
| Cart abandonment rate | < 35% |
| Creator opt-in rate | ≥ 60% |
| Product identification accuracy | ≥ 92% |

### Guardrail Metrics
- Reel completion rate: must not drop > 5%
- User-reported ad fatigue: must not increase
- Creator satisfaction NPS: must stay ≥ current baseline

---

## 🔧 Feature Breakdown

### Feature 1: AI Product Scanner (Core)
**What:** Computer vision model runs on every Reel, identifies visible products in real-time, matches them to catalog.

**How it works:**
- Meta's vision AI (Llama Vision) scans video frames every 2 seconds
- Identifies product category, brand, color, style attributes
- Matches against Meta Commerce catalog (2B+ products)
- Confidence threshold: only surface match if ≥ 88% confident

**UX:** Subtle "Shop" dot appears bottom-left of Reel when products detected. Doesn't autoplay, doesn't interrupt. Tap to expand.

---

### Feature 2: Native Product Shelf
**What:** Swipe-up shelf within the Reel showing identified products — without leaving the video.

**Design:**
- Half-sheet modal, Reel continues playing behind it
- Shows product image, name, price, brand, rating
- "Buy Now" → Meta Pay checkout (2-tap for saved cards)
- "Save" → adds to wishlist
- "See More" → full product page

**Key principle:** The Reel never stops. Pause on tap is opt-in, not forced.

---

### Feature 3: Creator Commerce Dashboard
**What:** One-tap for creators to tag products in their Reels and earn commission on sales.

**Creator flow:**
1. Upload Reel as normal
2. AI auto-suggests products it detected → creator confirms/edits
3. Creator optionally adds affiliate products manually
4. Dashboard shows: views → product taps → purchases → earnings

**Commission structure:**
- Physical goods: 8–15% of sale
- Beauty/fashion: 12–18%
- Creator gets paid weekly via Meta Pay

**Why creators will use it:** Current affiliate link earnings average $0.02/view. ShopReel targets $0.12/view for commerce-enabled Reels.

---

### Feature 4: AI Purchase Personalization
**What:** Same product, different price points and variants shown to different users based on purchase history and behavior.

**Examples:**
- User who buys Nike → shown Nike version of detected sneaker
- User who bought in $30–50 range → shown products in that range
- User who abandoned cart on similar item → shown with "Still available" nudge

**Powered by:** Meta's existing purchase behavior graph + real-time Reels engagement signals

---

### Feature 5: Live Commerce (Phase 2)
**What:** During Instagram Live, AI identifies products in real-time and surfaces purchase buttons to viewers.

**Use case:** Creator does a "Get Ready With Me" live — each product they pick up gets a buy button for viewers. QVC model with social trust layer.

**Revenue potential:** Live commerce is 10–20× higher conversion than recorded content in China (Douyin/TaoBao Live model).

---

## 📐 Technical Requirements

### AI Product Identification Pipeline
```
Reel Video (frames @ 2s intervals)
        ↓
Meta Vision AI (Llama Vision)
  → Object detection + classification
  → Brand/logo recognition
  → Color + style attribute extraction
        ↓
Product Matching Engine
  → Vector similarity search against Meta Commerce catalog
  → Confidence scoring (threshold: 88%)
  → Deduplication (same product, multiple frames)
        ↓
Commerce Layer
  → Price + availability check (real-time)
  → Personalization filter (user purchase history)
  → Affiliate attribution (creator commission)
        ↓
UX Delivery
  → Shop dot indicator
  → Product shelf modal
  → Checkout (Meta Pay)
```

### Performance Requirements
| Requirement | Target |
|-------------|--------|
| Product identification latency | < 800ms per frame |
| Shelf load time | < 1.2s |
| Checkout completion | < 45s end-to-end |
| False positive rate (wrong product) | < 8% |
| Catalog coverage | 2B+ products |

---

## 🗺️ Phased Rollout

### Phase 1 — Creator Beta (Month 1–3)
- 10,000 opted-in creators in US + India
- Manual product tagging only (no AI scanner yet)
- Measure: creator opt-in rate, purchase conversion, GMV
- **Go/No-Go:** Purchase conversion ≥ 3%, creator NPS ≥ 40

### Phase 2 — AI Scanner Launch (Month 4–6)
- AI product identification enabled for opted-in creators
- 5% of Reels feed gets ShopReel treatment
- Measure: AI accuracy, Reel completion rate impact, GMV lift
- **Go/No-Go:** AI accuracy ≥ 88%, completion rate drop < 5%

### Phase 3 — Full Rollout (Month 7–9)
- All creators eligible, all Reels scannable
- Personalization engine live
- **Go/No-Go:** GMV run-rate ≥ $10B annualized

### Phase 4 — Live Commerce (Month 10–12)
- Real-time product identification during Instagram Live
- Creator Live Commerce dashboard
- Target: $15B annualized GMV

---

## ⚠️ Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| AI misidentifies products → wrong purchase | High | High | 88% confidence threshold + "Report wrong product" → immediate removal |
| Reels feel too commercial → user drop-off | High | High | Shop dot is subtle, opt-in expansion, never auto-interrupts |
| Creators don't opt in | Medium | High | Clear earnings dashboard, $0.12/view target vs current $0.02 |
| Brand catalog gaps — product not found | High | Medium | "Find similar" fallback when exact match unavailable |
| Privacy concerns — behavior tracking | Medium | High | Clear opt-out, GDPR/DPDP compliance, no cross-app tracking disclosure |
| TikTok Shop responds aggressively | High | Medium | Speed to market — 6 month window before TikTok closes gap |

---

## 🚫 Not Building (v1)

- **AR try-on** — powerful but adds 6+ months; v2
- **Group shopping / social cart** — interesting but unproven; v2  
- **B2B / wholesale** — different buyer journey; separate product
- **Subscription products** — complex recurring commerce; v2

---

## 💬 Design Principles

1. **The Reel is primary.** Commerce is secondary. Never interrupt the viewing experience.
2. **One tap to interested, two taps to purchased.** If it takes more, we've failed.
3. **Earn creator trust first.** If creators don't opt in, the product doesn't exist.
4. **Surface, don't push.** Show the shop dot. Don't autoplay the shelf.

---

*Shanit Nagre · AI Product Manager · [shanitnagre.github.io](https://shanitnagre.github.io)*
