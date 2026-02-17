# Meta Ads — Campaign Playbook

## Campaign Architecture

### Why Single Campaign + Single Ad Set

For launch-phase products with limited budget ($50-200/day):

```
Campaign (OUTCOME_SALES, CBO)
└── Ad Set (Broad US, Purchase Optimization)
    └── All ads compete fairly
```

**Rationale:**
- OUTCOME_SALES objective optimizes for purchases (not clicks or traffic)
- CBO lets Meta allocate budget automatically across ads
- Single ad set = all budget concentrated → faster learning phase exit
- Broad US targeting lets Andromeda find buyers automatically
- Advantage+ placements → algorithm picks best platforms/positions

### Why NOT Multiple Ad Sets by Awareness Stage

Splitting into multiple ad sets (Unaware, Problem Aware, Solution Aware, etc.) is suboptimal because:
1. Budget fragmentation — $200/day ÷ 5 = $40/day per ad set → never exits learning phase (needs ~50 conversions/week)
2. Andromeda handles awareness-stage matching automatically via creative analysis
3. Audience overlap between ad sets causes self-competition in auctions
4. Budget gets stuck in underperforming ad sets even with CBO

### Key Settings Reference

| Setting | Value | Why |
|---------|-------|-----|
| Objective | OUTCOME_SALES | Optimizes for purchases |
| Budget | CBO at campaign level | Lets Meta allocate across ad sets |
| Bid Strategy | LOWEST_COST_WITHOUT_CAP | Best for early campaigns |
| Billing Event | IMPRESSIONS | Standard for most objectives |
| Optimization Goal | OFFSITE_CONVERSIONS | Purchase optimization |
| Promoted Object | pixel_id + PURCHASE | Pixel-based conversion tracking |
| Targeting | US only, no restrictions | Broad for Andromeda |
| Placements | Advantage+ (omit all) | Let algorithm optimize |
| Status | PAUSED | Always create paused, review first |
| Attribution | 7-day click, 1-day view | Default, maximizes signal |

## The Advertorial Funnel

```
Ad (Meta) → Advertorial (neutral comparison article) → Product Page (direct sale)
```

- **Advertorial-bound ads**: curiosity hooks, Learn More CTA, never mention product/price
- **PDP-bound ads**: direct sell, Shop Now CTA, can mention product name/price/guarantees

### Destination Routing

| Ad Type | Destination | CTA |
|---------|------------|-----|
| Advertorial-bound | Comparison article (e.g., trustedselections.org) | Learn More |
| PDP-bound (Most Aware) | Product page (e.g., evolance.com) | Shop Now |

## Wave Management

### How to Launch a New Wave

1. **Pause the old wave** — don't delete (preserves data for analysis)
2. **Add new ads to the SAME campaign/ad set** — preserves pixel learning
3. **Create all new ads as PAUSED** — review in Ads Manager before activating
4. **Activate new wave after review** — Meta needs 24-48h to start optimizing

### Why Always Add to Existing Campaign

Creating a new campaign resets algorithm learning entirely:
- Pixel data and conversion patterns are lost
- Learning phase restarts (3-5 days, ~50 conversions)
- Historical optimization data is wasted

**Create new campaign ONLY when:**
- Changing objective (e.g., Traffic → Sales)
- Targeting a fundamentally different audience
- Launching a different product

**Add to existing when:**
- Adding new creatives or copy variations
- Testing different faces/people (UGC variants)
- Swapping creative waves (pause old, activate new)
- Adjusting budget or optimization settings

## Scaling Structures

### $100-200/day — Keep Single Campaign
Same structure, just increase `daily_budget`. Scale 20-30% every 3-5 days.

### $200-500/day — Two-Campaign Structure
```
Campaign 1: Sales — Scaling (80% budget, CBO)
└── Ad Set: Broad US — Purchase
    └── Top 8-10 proven performers

Campaign 2: Sales — Testing (20% budget, ABO or CBO)
└── Ad Set: Broad US — Purchase
    └── 5-10 new creatives being tested weekly
```

### $500+/day — Add Advantage+ Shopping
```
Campaign 1: Advantage+ Sales Campaign (60-70% budget)
└── Auto-managed by Meta (broad, all placements)
    └── All best performers

Campaign 2: Manual Sales — Scaling (20-30% budget, CBO)
└── Ad Set: Broad US — Purchase
    └── Graduated winners from testing

Campaign 3: Manual Sales — Testing (10% budget)
└── Ad Set: Broad US — Purchase
    └── New creatives
```

## Creative Strategy

### Face/UGC Variant Testing
When a design has multiple variants (same layout, different people/photos), create each as a separate ad with a unique copy angle. This lets the algorithm test:
- Which face/person resonates with the audience
- Which copy angle drives the most clicks and conversions

### Flexible Creatives (asset_feed_spec)
Each ad should use Meta's flexible creative format with:
- **3 primary texts** (short / medium / long versions of the persona angle)
- **3 headlines** (each a different hook complementing the visual)
- **1 description** (summarizes the comparison)

Meta tests all text combinations automatically and optimizes for the best-performing mix.

### Persona Mapping Across Ads
Assign different persona angles across ads so the algorithm finds which emotional entry point resonates best:

| Persona | Hook Angle |
|---------|------------|
| Comparison Shopper | "I compared the top sellers" — ingredient quality differences |
| Foggy Morning Person | Grogginess as comparison criterion (neutral, no attacks) |
| Label Reader | Transparency, proprietary blends, dose hiding |
| Overwhelmed Searcher | Too many options, curated guide |
| Pattern Recognizer | New research trends, ingredient interest |
| Authority Reviewer | Team review, expert methodology |

## Performance Analysis Framework

### Key Metrics to Track
- **CTR** — top-of-funnel health (>5% is strong for supplements)
- **CPC** — cost efficiency (lower is better, but not if it doesn't convert)
- **Advertorial→PDP rate** — critical funnel metric (target >25%)
- **ATC rate** — product page effectiveness
- **Purchase rate** — bottom-line metric
- **ROAS** — return on ad spend

### Critical Insight: High CTR ≠ Conversions
The algorithm can optimize for clicks instead of purchases. A high-CTR ad with zero conversions will consume budget without generating revenue. Always evaluate ads on purchase metrics, not just CTR.

### Funnel Breakdown by Destination
Always analyze advertorial-bound and PDP-bound ads separately:
- Advertorial ads: measure Impressions → Clicks → LPV → ViewContent (adv→PDP) → ATC → Purchase
- PDP ads: measure Impressions → Clicks → LPV → ATC → Purchase

### When to Pause an Ad
- 10+ days running with 0 purchases and significant spend (>$50)
- High CTR but 0 conversions (algorithm optimizing for wrong metric)
- CPP (cost per purchase) consistently >3x the average

### When to Scale an Ad
- Consistent purchases over 7+ days
- ROAS above breakeven threshold
- Increasing or stable conversion rates

## Ad Naming Convention
`CADENCE #{NN} — {headline_truncated} [{stage}]`
Example: `CADENCE #04 — Looking for a different approa [Problem Aware]`

## Wave 1 Performance Learnings (Feb 8-16, 2026)

### Funnel
- **$771.66 spent, 1 purchase** ($771.66 CPA)
- 25,744 impressions → 2,996 clicks (11.78% CTR) → 2,264 LPV → 110 ViewContent (**4.9% adv→PDP**) → 6 ATC → 1 purchase
- Only conversion from Ad #14 (Most Aware, direct to PDP)

### Key Takeaways
1. **Top of funnel was strong** — 11.78% CTR, $0.26 CPC, 75.6% LPV rate
2. **Critical leak at Advertorial→PDP (4.9%)** — 95% of advertorial visitors never reached the product page
3. **Ad #05 consumed 37% of budget** ($282) with 14.84% CTR but 0 purchases — algorithm optimized for clicks
4. **Direct-to-PDP (Ad #14)** was the only converting path — higher CPC ($3.14) but actual purchases
5. **Copy mismatch** between Wave 1 ad tone and neutral advertorial tone was likely a factor
6. **Wave 2 fix**: persona-based, editorial-tone copy that mirrors the advertorial voice
