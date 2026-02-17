# Meta Ads — API Patterns & Operations

## API Basics

- **API version**: v21.0
- **Base URL**: `https://graph.facebook.com/v21.0`
- **Rate limits**: Development access tier — **5-second delays** between API calls
- **Error 80004**: Wait 60+ seconds, retry up to 3 times
- **Usage header**: Check `x-business-use-case-usage` for rate limit stats

## Creating Ads

### Step 1: Upload Image
```python
image_hash = meta_upload_image(Path("images/ad_XX_feed.png"))
```

### Step 2: Create the Creative

**Option A — Simple creative (single headline + single text):**
```python
creative = meta_post(f"{AD_ACCOUNT_ID}/adcreatives", {
    "name": f"CADENCE #XX — {headline}",
    "object_story_spec": json.dumps({
        "page_id": PAGE_ID,
        "link_data": {
            "image_hash": image_hash,
            "link": destination_url,
            "message": primary_text,
            "name": headline,
            "caption": "trustedselections.org",  # display link
            "call_to_action": {
                "type": "LEARN_MORE",
                "value": {"link": destination_url},
            },
        },
    }),
})
```

**Option B — Flexible creative (multiple headlines + multiple texts):**
Use `asset_feed_spec` with minimal `object_story_spec` (page_id only).
Meta tests all combinations automatically. Up to 5 of each.
```python
creative = meta_post(f"{AD_ACCOUNT_ID}/adcreatives", {
    "name": f"CADENCE #XX — {headline}",
    "asset_feed_spec": json.dumps({
        "images": [{"hash": image_hash}],
        "bodies": [
            {"text": "Primary text option 1"},
            {"text": "Primary text option 2"},
            {"text": "Primary text option 3"},
        ],
        "titles": [
            {"text": "Headline option 1"},
            {"text": "Headline option 2"},
            {"text": "Headline option 3"},
        ],
        "descriptions": [{"text": "Link description"}],
        "ad_formats": ["SINGLE_IMAGE"],
        "call_to_action_types": ["LEARN_MORE"],
        "link_urls": [{"website_url": destination_url, "display_url": "trustedselections.org"}],
    }),
    "object_story_spec": json.dumps({"page_id": PAGE_ID}),
})
```

**CRITICAL:** `asset_feed_spec` and full `object_story_spec` (with `link_data`) are **MUTUALLY EXCLUSIVE**. When using `asset_feed_spec`, `object_story_spec` must contain ONLY `{"page_id": PAGE_ID}`.

### Step 3: Create the Ad
```python
ad = meta_post(f"{AD_ACCOUNT_ID}/ads", {
    "name": f"CADENCE #XX — {headline} [{stage}]",
    "adset_id": ADSET_ID,
    "creative": json.dumps({"creative_id": creative["id"]}),
    "status": "PAUSED",
})
```

## Updating Ads

### Update Copy (Creatives Are Immutable)
Meta ad creatives cannot be edited after creation. To update copy:
1. Create a NEW creative with corrected copy (reuse same image hash)
2. Update the ad to point to the new creative
3. Save new creative IDs to state file for tracking

```python
# 1. Create new creative with fixed copy (flexible)
new_creative = meta_post(f"{AD_ACCOUNT_ID}/adcreatives", {
    "name": f"CADENCE #XXv2 — {headline}",
    "asset_feed_spec": json.dumps({
        "images": [{"hash": image_hash}],
        "bodies": [{"text": pt1}, {"text": pt2}, {"text": pt3}],
        "titles": [{"text": hl1}, {"text": hl2}, {"text": hl3}],
        "descriptions": [{"text": description}],
        "ad_formats": ["SINGLE_IMAGE"],
        "call_to_action_types": ["LEARN_MORE"],
        "link_urls": [{"website_url": url, "display_url": "trustedselections.org"}],
    }),
    "object_story_spec": json.dumps({"page_id": PAGE_ID}),
})

# 2. Swap it into the ad
meta_post(f"{ad_id}", {
    "creative": json.dumps({"creative_id": new_creative["id"]}),
})
```

### Update Image
```python
new_hash = meta_upload_image(Path("images/ad_XX_feed_v2.png"))
new_creative = meta_post(f"{AD_ACCOUNT_ID}/adcreatives", {
    "name": "CADENCE #XX — New Image v2",
    "object_story_spec": json.dumps({
        "page_id": PAGE_ID,
        "link_data": {"image_hash": new_hash, "link": url, ...},
    }),
})
meta_post(f"{ad_id}", {
    "creative": json.dumps({"creative_id": new_creative["id"]}),
})
```

### Pause/Activate
```python
meta_post(f"{ad_id}", {"status": "PAUSED"})  # or "ACTIVE"
```

### Delete
```python
requests.delete(f"{META_API_BASE}/{ad_id}", params={"access_token": TOKEN})
```

## Bulk Operations

### Pause All Ads
```python
for ad_id in all_ad_ids:
    meta_post(f"{ad_id}", {"status": "PAUSED"})
    time.sleep(5)
```

### Get Performance Data
```python
resp = requests.get(
    f"{META_API_BASE}/{adset_id}/ads",
    params={
        "access_token": TOKEN,
        "fields": "name,status,insights{impressions,clicks,spend,actions}",
    },
)
```

## Display Link (Caption)
To show a clean URL instead of the full URL:
- In `object_story_spec` → `link_data`: add `"caption": "trustedselections.org"`
- In `asset_feed_spec` → `link_urls`: add `"display_url": "trustedselections.org"`
- Do NOT include `https://` or `www.` — just the bare domain

## Instagram Account
Set the Instagram account at the **ad set level** (applies to ALL ads):
```python
meta_post(f"{adset_id}", {"instagram_actor_id": "INSTAGRAM_ACCOUNT_ID"})
```

Note: Setting IG at the creative level uses `instagram_user_id` in `object_story_spec`, but may require `pages_read_engagement` permission. Ad set level is simpler and ensures consistency.

## Site Links
Additional links shown below the ad:
- **Ads Manager UI** (easiest): Ad level → "Site links" toggle
- **API**: through `asset_feed_spec.sitelinks` (requires specific permissions)
- Currently recommended to manage manually in Ads Manager

## Face/UGC Variant Testing
When a design has multiple variants (same layout, different people):
1. Create each variant as a separate ad with a unique copy angle
2. Algorithm tests which face + copy combination performs best

**Naming convention:**
- Images: `ad_17v1_feed.png`, `ad_17v2_feed.png`, etc.
- Ad numbers: sequential (17, 18, 19... for variants)
- Track variant index in ad data for mapping

## State Management
Always save state after each successful API call for resume capability:
- `campaign_id` — the campaign
- `adset_id` — the single ad set
- `creative_ids` — {ad_num: creative_id}
- `ads_created` — {ad_num: ad_id}
- `image_hashes` — {ad_num: image_hash}

Use separate state files for different operations (e.g., initial creation vs. copy updates vs. new wave additions).
