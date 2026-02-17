---
name: meta-ads-copy
description: Complete Meta ads skill for supplement advertorial funnels. Covers ad copy creation, compliance, campaign strategy, API patterns, and performance analysis. Single source of truth — all agents reference this skill.
---

# Meta Ads — Complete Skill Reference

## Overview

This skill covers everything needed to create, manage, and optimize Meta (Facebook/Instagram) ads for supplement products using an advertorial funnel:

```
Ad (curiosity hook) → Advertorial (neutral comparison article) → Product Page (direct sale)
```

Ad copy must create curiosity for the comparison article — never sell the product directly.

## Skill Files

| File | Contents |
|------|----------|
| `SKILL.md` (this file) | Copy rules, personas, compliance, generation workflow |
| `reference.md` | Copy examples by persona, compliance quick-check tables |
| `campaign-playbook.md` | Campaign architecture, scaling, wave management, performance analysis |
| `api-patterns.md` | API code patterns for creating/updating ads, bulk ops, state management |

## When to Use This Skill

- Creating new Meta ads that drive traffic to an advertorial/comparison article
- Updating existing ad copy for advertorial alignment
- Generating persona-based copy variations for A/B testing
- Checking ad copy for FDA, FTC, and Meta platform compliance
- Structuring campaigns and planning scaling strategies (see `campaign-playbook.md`)
- Writing API code to create/update ads (see `api-patterns.md`)

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

**IMPORTANT:** `asset_feed_spec` and full `object_story_spec` (with `link_data`) are MUTUALLY EXCLUSIVE. When using `asset_feed_spec`, `object_story_spec` must contain ONLY `{"page_id": PAGE_ID}`.

For full API code patterns (simple + flexible creatives, updates, bulk ops), see `api-patterns.md`.

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

For API code patterns, see `api-patterns.md`.

## Copy Generation Workflow

1. **Read the advertorial** — understand what users see after clicking
2. **Assign personas** — map each ad to a persona angle
3. **Write 3 primary texts per ad** — short (1-2 sentences), medium (3-4 sentences), long (4-6 sentences)
4. **Write 3 headlines per ad** — each a different hook, max ~40 chars
5. **Write 1 description per ad** — summarizes the comparison angle
6. **Run compliance check** — scan for banned claims, superiority patterns, Personal Attributes trap
7. **Create creatives via API** — use asset_feed_spec for flexible testing
8. **Save state** — track creative IDs for future updates

## Performance Learnings (Wave 1: Feb 8-16, 2026)

Use these to inform copy decisions for future waves.

### Copy-Relevant Findings
- **Problem Aware angle** had highest CTR (14.84%) — curiosity hooks like "Exploring better options" resonate
- **High CTR ≠ conversions** — Ad #05 consumed 37% of budget with 0 purchases. Algorithm optimized for clicks, not purchases
- **Direct-to-PDP (Most Aware)** generated the only purchase — "Calm+Rest by Evolance" with Shop Now CTA
- **Advertorial→PDP conversion was 4.9%** (target >25%) — Wave 1 copy had tone mismatch with neutral advertorial
- **Ad #12** ("What people are choosing") generated 3 checkouts from only $9.40 — social proof angles convert

### Wave 2 Copy Fix (Applied Feb 16)
- Persona-based, editorial-tone copy that mirrors the advertorial's voice
- No product reveals, no pricing, no melatonin attacks
- Always validate new copy against the advertorial content before publishing

### Implications for Future Copy
- Advertorial-bound ads MUST match the editorial tone of the advertorial
- Consider creating more Most Aware / direct-to-PDP ads — they convert despite higher CPC ($3.14 vs $0.26)
- Don't judge copy purely on CTR — high CTR ≠ purchases
- Social proof angles ("what people are choosing") show promise

For full funnel metrics and campaign-level analysis, see `campaign-playbook.md`.
