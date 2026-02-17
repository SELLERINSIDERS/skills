---
name: meta-ads-copy
description: Create and update Meta (Facebook/Instagram) ad copy for supplement advertorial funnels. Generates persona-based, compliance-checked copy aligned with advertorial landing pages. Handles flexible creatives (asset_feed_spec) with multiple headline/text variations.
---

# Meta Ads Copy — Advertorial Funnel Ad Creation

## Overview

This skill creates compliant Meta ad copy for supplement products using an advertorial funnel:

```
Ad (curiosity hook) → Advertorial (neutral comparison article) → Product Page (direct sale)
```

Ad copy must create curiosity for the comparison article — never sell the product directly.

## When to Use This Skill

- Creating new Meta ads that drive traffic to an advertorial/comparison article
- Updating existing ad copy for advertorial alignment
- Generating persona-based copy variations for A/B testing
- Checking ad copy for FDA, FTC, and Meta platform compliance

## The Advertorial Funnel — Copy Rules

### What Ad Copy MUST Do
1. **Mirror the advertorial's neutral editorial tone** — "I compared", "we reviewed", "our team tested"
2. **Tease the comparison**, not the winner — "see how they compared", "the results surprised me"
3. **Create open loops** (Zeigarnik Effect) — curiosity that can only be resolved by reading the article
4. **Reference comparison criteria** the advertorial covers (e.g., ingredient quality, dosing, testing, grogginess)
5. **Use persona-based angles** — each ad targets a different emotional entry point

### What Ad Copy MUST NOT Do
1. **Name any product or brand** — the reader discovers the "winner" in the article
2. **Mention pricing, offers, or guarantees** — these are product-page concerns
3. **Reveal specific ingredients or mg amounts** as the main hook — dosing is one of many criteria
4. **Attack specific supplement types** (e.g., melatonin, ashwagandha) — if these products appear in the comparison article, attacking them creates an expectation mismatch
5. **Hint at a predetermined winner** — frame everything as a fair comparison
6. **Use superiority claims** — "the best", "only one", "stood out", "checked every box", "nothing compares"

### Direct-to-PDP Ads (Different Rules)
Ads driving directly to a product page CAN mention: product name, pricing, guarantees, specific ingredients, doses, and direct benefits.

## Persona-Based Copy Angles

Each ad should target a different emotional entry point. Assign personas across ads so the algorithm can find which resonates best with different audiences.

| Persona | Hook Style | Example Angle |
|---------|-----------|---------------|
| **Comparison Shopper** | "I compared X..." | Ingredient quality differences between top sellers |
| **Foggy Morning Person** | Relatable experience | Grogginess as a comparison criterion (neutral, not an attack) |
| **Label Reader** | Transparency / skeptic | Proprietary blends, dose hiding, generic vs. patented forms |
| **Overwhelmed Searcher** | Simplification | Too many options, need a curated guide |
| **Pattern Recognizer** | Trend awareness | New research, growing ingredient interest |
| **Authority Reviewer** | Expert credibility | Team review, methodology-driven, data-focused |

## Flexible Creatives (asset_feed_spec)

Each ad should use Meta's flexible creative format with:
- **3 primary texts** (short / medium / long versions of the persona angle)
- **3 headlines** (each a different hook complementing the visual)
- **1 description** (summarizes the comparison)

Meta automatically tests all combinations and optimizes for the best-performing mix.

### API Pattern
```python
creative = meta_post(f"{AD_ACCOUNT_ID}/adcreatives", {
    "name": f"Ad Name",
    "asset_feed_spec": json.dumps({
        "images": [{"hash": image_hash}],
        "bodies": [
            {"text": "Primary text option 1 (short)"},
            {"text": "Primary text option 2 (medium)"},
            {"text": "Primary text option 3 (long)"},
        ],
        "titles": [
            {"text": "Headline option 1"},
            {"text": "Headline option 2"},
            {"text": "Headline option 3"},
        ],
        "descriptions": [{"text": "Link description"}],
        "ad_formats": ["SINGLE_IMAGE"],
        "call_to_action_types": ["LEARN_MORE"],
        "link_urls": [{"website_url": advertorial_url, "display_url": "yourdomain.com"}],
    }),
    "object_story_spec": json.dumps({"page_id": PAGE_ID}),
})
```

**IMPORTANT:** `asset_feed_spec` and full `object_story_spec` (with `link_data`) are MUTUALLY EXCLUSIVE. When using `asset_feed_spec`, `object_story_spec` must contain ONLY `{"page_id": PAGE_ID}`.

## Compliance Rules

### FDA — Supplement Claims
- **Structure/function language ONLY**: "supports", "promotes", "helps maintain"
- **No disease claims**: never "cures", "treats", "prevents", "diagnoses"
- **No disease names**: insomnia, depression, anxiety disorder, ADHD, etc.
- **Safe alternatives**: "occasional sleeplessness", "difficulty unwinding", "everyday stress"
- **FDA disclaimer required on landing page** (not in ad copy itself):
  `*These statements have not been evaluated by the FDA. This product is not intended to diagnose, treat, cure, or prevent any disease.`

### FTC — Substantiation
- **No absolute claims**: "guaranteed results", "works for everyone", "scientifically proven"
- **No unsubstantiated comparisons**: "#1 supplement", "most effective", "outperforms competitors"
- **No fabricated testimonials** or before/after imagery

### Meta Platform — Ad Policies
- **Personal Attributes Trap**: Never use YOU + NEGATIVE STATE pattern
  - REJECTED: "Are you stressed?", "Struggling with sleep?", "Your supplement is failing you"
  - SAFE: "Many people experience...", "75% of adults report...", "Not every formula..."
- **No superiority claims**: "the best", "only one", "stood out", "got it right", "checked every box"
- **Use comparison language**: "See how they compared", "The differences were significant"

### Ingredient-Specific Rules
- **Never attack a supplement type** that appears in your own comparison article
- **Never make specific mg amounts the main hook** — dosing is one criterion among many
- **Grogginess**: Can mention as a comparison criterion, but never as an attack on a specific product
- **Melatonin, ashwagandha, etc.**: Mention neutrally if at all — never position as "the enemy"

## Copy Update Process (Creatives Are Immutable)

Meta ad creatives cannot be edited after creation. To update copy:

1. **Create a NEW creative** with corrected copy (reuse the same image hash)
2. **Update the ad** to point to the new creative
3. **Save state** (new creative IDs) for tracking and resume capability

```python
# 1. Create new creative
new_creative = meta_post(f"{AD_ACCOUNT_ID}/adcreatives", {
    "name": f"Ad Name v2",
    "asset_feed_spec": json.dumps({...new copy...}),
    "object_story_spec": json.dumps({"page_id": PAGE_ID}),
})

# 2. Swap into existing ad
meta_post(f"{ad_id}", {
    "creative": json.dumps({"creative_id": new_creative["id"]}),
})
```

## Copy Generation Workflow

1. **Read the advertorial** — understand what users see after clicking
2. **Assign personas** — map each ad to a persona angle
3. **Write 3 primary texts per ad** — short (1-2 sentences), medium (3-4 sentences), long (4-6 sentences)
4. **Write 3 headlines per ad** — each a different hook, max ~40 chars
5. **Write 1 description per ad** — summarizes the comparison angle
6. **Run compliance check** — scan for banned claims, superiority patterns, Personal Attributes trap
7. **Create creatives via API** — use asset_feed_spec for flexible testing
8. **Save state** — track creative IDs for future updates

## Rate Limits

- Development access tier: **5-second delays** between API calls
- On error code 80004: wait 60+ seconds, retry up to 3 times
- Check `x-business-use-case-usage` response header for usage stats
