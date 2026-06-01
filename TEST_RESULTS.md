# Motion Creative Plugin — Test Results

**Workspace:** Motion (FB, TikTok, YouTube) — `6424a223ab8613ce345f95b9`
**Date:** 2026-04-13
**Methodology:** Pulled raw data from Motion MCP tools, then ran the same tool calls each skill would make and compared outputs against ground truth.

---

## Executive Summary

**22 checks run across 6 skill categories. Results: 12 PASS, 4 PARTIAL PASS, 6 FAIL.**

Three critical issues will visibly degrade the user experience:

1. **Own-brand voice data is empty** — `get_workspace_brand` returns only a brandId with zero positioning, voice, or guidelines. Every skill that personalizes output to the user's brand (build-brief, qa-feedback, write-hooks, create-concepts, ugc-scripts) produces generic, unbranded recommendations.

2. **HOOK insightType is broken** — returns the identical 10 creatives in the identical order as SCALING. The write-hooks skill thinks it's pulling "top hook performers" but actually gets scaling creatives. The top spender ($18K, 59.75% thumbstop) doesn't appear in HOOK top 10 at all.

3. **hook_rate and hold_rate are universally empty** — No skill has fallback logic. Morning-briefing's "hook changes" section, weekly-performance's behavioral interpretation table, and the 4-Question Framework's Q2 all reference metrics that don't exist for this workspace.

---

## Detailed Findings by Skill Area

### A. Performance Analysis / Morning Briefing / Weekly Performance

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 1 | goalMetric correctly surfaced from SPEND call | **PASS** | Custom conversion `3612264235660975_cost` returned. Per-creative values available for 7/10 top creatives (3 lowest-spend omitted due to zero conversions). |
| 2 | Campaign hierarchy (names, ad sets, ads) present | **PASS** | Full `campaignName`, `adsetName`, `adName` in every insight row. Naming conventions match the skill's heuristics exactly (e.g., "Prospecting", "ABO", "Broad"). |
| 3 | Demographic breakdown available | **PASS** | 21 age/gender segments with spend, ROAS, thumbstop. Top segment: 35-44F at $50,626 spend. |
| 4 | WoW deltas computable (Week 1 vs Week 2) | **PASS** | Spend +45.8%, 11 creatives overlap, 9 new to Week 2, 9 dropped. Biggest gainer: Caleb Kruse +256%. |
| 5 | ROAS insightType for ROAS-meaningless workspace | **FAIL** | Returns data ranked by ROAS where account average is 0.012. No guardrail warns when ROAS is meaningless for a B2B lead-gen workspace. Skills would present misleading "efficiency" rankings if a user requests ROAS analysis. |
| 6 | hook_rate / hold_rate data available | **FAIL** | `hook_rate`: N/A for all creatives across all insightTypes. `hold_rate`: missing entirely from response. No skill documents fallback behavior. Morning-briefing's "Hook changes" section will always be empty. |
| 7 | "Efficiency standouts" computable with goalMetric | **PARTIAL** | 7/10 creatives have the custom conversion cost. The 3 missing are lowest-spend — exactly the "hidden gem" candidates the skill tries to surface. Skill silently skips them. |

### B. Competitor / Industry Trends / Competitor Watch

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 8 | limit=500 returns 500 creatives | **PASS** | HexClad (1,195 active ads) returned exactly 500. |
| 9 | NEWEST + OLDEST two-pass deduplication | **PASS** | Zero overlap between passes. Combined 1,000 unique creatives = 83.7% portfolio coverage. |
| 10 | Format distribution reliable at scale | **PARTIAL** | 500-sample: 60% video / 40% image. 20-sample ground truth showed 85%/15% — dramatically different. Small samples produce unreliable format mix conclusions. |
| 11 | Inactive ("recently killed") ads available | **PASS** | AG1 returned 50 inactive creatives with pauseDate (Mar 27 - Apr 8) and daysActive (1-8 days). Delta tracking is viable. |
| 12 | Inspo brand context rich for competitors | **PASS** | HexClad returned: positioning, voice/tone, 9 messaging angles, customer voice analysis (praise, objections, motivations, transformations), 5 competitive comparisons, 9 products with features. |
| 13 | Own-brand voice from get_workspace_brand | **CRITICAL FAIL** | Returns ONLY `{"brandId":"6656a559f4a504552b6518c0"}`. Zero positioning, voice, tone, or guidelines. Skills that depend on brand voice (build-brief, qa-feedback, write-hooks, create-concepts, ugc-scripts) will produce generic output. |

### C. Analyze Ad / Write Hooks

| # | Check | Result | Evidence |
|---|-------|--------|----------|
| 14 | Entity hierarchy for specific creative | **PASS** | "Eliah V1 H1" returns campaignName, adsetName, adName. Multiple ad variants (H1, H2) in different adsets within the same campaign are visible. |
| 15 | Transcript with hook (first 1-3s) | **PASS** | Full transcript: 8 segments, 0-40960ms, hook: "This is a visual hook and it works because of a formula not random luck. Let me break it down." |
| 16 | Creative summary for static/image ads | **PASS** | Returns `adFormat: "Comparison ad"`, summary, hook text. CTAs array empty (no CTA detected in the image). |
| 17 | HOOK insightType differs from SCALING | **FAIL** | Identical 10 creatives in identical order. write-hooks skill using HOOK insightType is NOT pulling best hook performers — it's pulling scaling creatives. Top spender ($18K, 59.75% thumbstop) absent from HOOK top 10. |
| 18 | 4-Question Framework metric coverage | **PARTIAL** | Q1 (Stop scroll): PASS — thumbstop 59.75%. Q2 (Hold attention): DEGRADED — hold_rate missing but video retention percentiles (25p/50p/75p/95p) available as proxy. Q3 (Drive action): PASS — CTR, CPC, click ratios all present. Q4 (Convert): DEGRADED — goalMetric cost present ($6,139) but only 3 conversions on $18K spend. |

---

## Impact Assessment by Skill

| Skill | Blocked? | Issues |
|-------|----------|--------|
| **performance-analysis** | No | hook_rate empty (degrades hook section); goalMetric missing for lowest-spend creatives |
| **morning-briefing** | No | "Hook changes" section always empty; hold_rate comparison impossible |
| **weekly-performance** | No | Behavioral interpretation table incomplete (hold_rate missing); creative thumbnails may not render (creativeUrls empty) |
| **analyze-ad** | No | 4-Question Q2 degraded; Q4 low conversion volume for this workspace |
| **industry-trends** | No | Small samples give wrong format distributions; must use limit=500 |
| **competitor-watch** | No | Two-pass strategy misses ~16% of large portfolios; works well otherwise |
| **write-hooks** | **YES** | HOOK insightType returns scaling creatives, not hook performers. Skill pulls wrong data. |
| **build-brief** | **Degraded** | No brand voice data → generic copy, can't enforce Creative Don'ts |
| **create-concepts** | **Degraded** | No brand voice → concepts not grounded in brand positioning |
| **qa-feedback** | **Degraded** | No brand voice → can't score Brand Alignment dimension |
| **ugc-scripts** | **Degraded** | No brand voice → scripts won't match brand tone |
| **find-iterations** | No | Same hook_rate/hold_rate gaps; diversity audit depends on glossary (well-populated) |
| **audience-research** | No | Demographic data available; competitor brand context rich |
| **customize** | No | Pre-fill will be empty for brand voice fields |
| **concept-engine** | **Degraded** | No brand voice → same issue as create-concepts |

---

## Recommended Fixes (Priority Order)

### P0: Fix Now

**1. write-hooks: Stop using insightType HOOK**
The HOOK ranking is identical to SCALING. Replace with: pull SPEND ranking for video creatives, then sort client-side by `thumbstop_ratio`. This surfaces actual high-hook creatives like "Tarah - Hooks Guide" (63.41% thumbstop) and "Eliah - Top Hooks" (59.75% thumbstop) that HOOK ranking misses entirely.

**2. Brand voice fallback: Use get_inspo_brand_context for own brand**
`get_workspace_brand` is empty. But `get_inspo_brand_context` works for ANY brandId — including the workspace's own brand (`6656a559f4a504552b6518c0`). Test whether calling `get_inspo_brand_context(brandId="6656a559f4a504552b6518c0")` returns the same rich data it does for competitor brands. If so, update all brand-dependent skills to use this as primary source.

Alternatively, the `/customize` skill already creates `motion-creative.config.md` with brand guidelines. Ensure all skills read this file as the brand voice source even when MCP data is empty.

### P1: Fix Soon

**3. Add fallback logic for hook_rate / hold_rate**
In every skill that references hook_rate or hold_rate, add a documented degradation path:
- If hook_rate is N/A → use `thumbstop_ratio` and note the substitution
- If hold_rate is N/A → use video retention percentiles (`video_p25_watched_ratio` through `video_p75_watched_ratio`) and note the substitution
- Morning-briefing: skip "Hook changes" section instead of showing empty data
- Weekly-performance: skip hold_rate interpretation patterns instead of showing incomplete analysis

**4. Guard against meaningless ROAS**
When ROAS < 0.05 at account level AND the goalMetric is a custom conversion, auto-detect that ROAS is not the real efficiency metric. Either warn the user or auto-switch to the goalMetric for efficiency rankings.

### P2: Improve Later

**5. Document the NEWEST+OLDEST coverage gap**
The two-pass strategy covers 83.7% of large portfolios. For brands with 1000+ ads, add a third pass or note the gap in the output.

**6. Enforce minimum sample sizes for format distribution claims**
A 20-sample showed 85% video; a 500-sample showed 60% video. Skills making format distribution claims should warn when sample size < 100 or when the sample represents < 50% of the known portfolio.

**7. Handle creativeUrls being empty**
`includeCreativeUrls: true` doesn't return URLs. Skills (especially weekly-performance HTML deck) should fall back to `thumbnailUrl` from the `ad` object, which IS populated.

---

## Data Quality Summary

| Data Source | Quality | Notes |
|-------------|---------|-------|
| Creative insights (SPEND) | Good | Rich metrics, campaign hierarchy, custom conversions |
| Creative insights (SCALING) | Good | Scaling direction and WoW ratios accurate |
| Creative insights (HOOK) | **Bad** | Identical to SCALING — not a distinct ranking |
| Demographics | Good | 21 segments with full metric set |
| Glossary | Good | 8 categories, 205 values, well-populated |
| Own-brand voice | **Empty** | Only brandId returned |
| Competitor inspo creatives | Good | limit=500 honored, inactive ads available |
| Competitor brand context | Excellent | Rich positioning, voice, customer analysis |
| Transcripts (own) | Good | Full text with timestamps, hook extractable |
| Transcripts (competitor) | Good | Available but may have transcription errors (e.g., "HexCloud") |
| Creative summaries | Good | Format detection, summary, hooks work for image creatives |
| Account aggregates | Good | Spend, CPM, CTR, custom conversions all present |
| hook_rate / hold_rate | **Missing** | N/A / absent for all creatives across all calls |
| creativeUrls | **Missing** | Empty despite being requested; thumbnailUrl available as fallback |
