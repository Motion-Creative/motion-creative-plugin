---
name: Analyze Ad
description: >
  Deep-dive analysis of a specific creative asset. Use when the user asks about a
  specific ad by name, ID, or URL — "analyze this ad", "why is this working?",
  "why is this not working?", "break down this creative", "[pastes ad name]",
  "look at this ad", "what's the angle in this ad?", "compare to account averages",
  or references a specific creative by name or identifier. Handles creative lookup,
  disambiguation when multiple matches exist, entity hierarchy mapping
  (creative → ads → adsets → campaigns), and metric-appropriate analysis.
  This is the most common interaction pattern — users often analyze multiple ads
  in sequence. Always use this skill, not creative-strategist, when the user
  references a specific creative.
allowed-tools:
  - Read
  - mcp__ruimotion__get_auth_context
  - mcp__ruimotion__get_creative_insights
  - mcp__ruimotion__get_creative_summary
  - mcp__ruimotion__get_creative_transcript
  - mcp__ruimotion__get_demographic_breakdown
  - mcp__ruimotion__get_workspace_brand
  - mcp__ruimotion__get_glossary_values
  - AskUserQuestion
model: opus
---

# Analyze Ad

Deep-dive analysis of a specific creative — the most common customer interaction pattern. Users paste ad names one after another, going 20+ messages deep. Optimize for this multi-turn flow.

## References

Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/evaluation-framework.md` for the 4-Question Evaluation Framework, metric pattern correlations, Ready/Iterate/Rethink call, and performance context awareness.

## Phase 0: Find the Creative

The user may reference a creative by:
- **Exact ad name** — match against `adName` in creative insights results
- **Approximate name** — partial/fuzzy match, present candidates
- **Ad ID** — match against `adIds` array
- **Creative asset ID** — use `creativeAssetIds` filter parameter
- **Description** — "the UGC video with the woman unboxing"

Steps:
1. Pull `get_creative_insights` with SPEND insightType, limit 15. Include `includeCreativeUrls: true` so the user can see the creatives. Start with a smaller set — if the creative is active, it's likely in the top 15 by spend.
   On the first pull, extract `goalMetric` and `spendThreshold`. If `goalMetric.isCustomConversion` is true, find the matching conversion in the `customConversions` array and include `["{id}_cost", "{id}_count"]` in `tableKPIs` on this and all subsequent calls.
2. Search results for the user's reference (case-insensitive partial match on `adName`).
3. **Multiple matches** → present all matches with ad name, spend, and top metric. Ask which one. Never guess.
4. **No match in top 15** → widen the search: re-pull with limit 50. If still no match, tell the user and suggest: "Can you share the exact ad name, ad ID, or creative asset ID from Motion? Sometimes names don't match exactly."
5. **Single clear match** → proceed to analysis.

**Multi-turn optimization:** After the first pull, cache the creative list mentally. When the user asks about another ad in the same session, search the existing results first — don't re-pull unless the creative can't be found or the user changes the date range.

## Phase 1: Entity Hierarchy

For the matched creative, present the full entity map:

- **Creative Asset** (`creativeAssetId`) — the visual/video itself
- **Ad(s)** (`adIds`, `adName`) — the ad-level entities using this creative
- **Ad Set(s)** (`adsetIds`, `adsetName`) — targeting and budget containers
- **Campaign(s)** (`campaignIds`, `campaignName`) — objective and strategy level

If a creative runs in multiple ads/adsets/campaigns:
- Surface ALL contexts
- Note which campaign context is performing best
- Flag if the creative is in both testing and scaling campaigns — this is important context ("Running in your Winning Ads Scaling CBO means the algorithm has already validated it")
- Infer campaign stage from name heuristics: "Testing", "Prospecting", "Scaling", "CBO", "ABO", "Retargeting", "Lookalike", "TOF", "MOF", "BOF"

## Phase 2: Metric-Appropriate Analysis

Apply metric intelligence from `${CLAUDE_SKILL_DIR}/../creative-strategist/references/performance-metrics.md` — select metrics by creative format (video vs. static) and campaign objective (awareness, traffic, conversions). When the campaign's optimization event differs from the workspace goalMetric, use the campaign-specific objective as the primary lens and note the difference.

**Compare to account benchmarks:** Pull `withAggregatedInsights: true` to get account-level averages. Present the creative's metrics as above/below/at benchmark.

**Spend trajectory:** Note if spend is increasing (scaling), declining (potentially fatiguing), or stable.

## Performance Context Awareness

Apply performance context checks from `evaluation-framework.md` — funnel stage, budget starvation, account stage, and test fairness. Surface these factors when they change the interpretation of metrics.

## Phase 3: The "Why"

Go beyond what the numbers say — explain WHY this creative performs the way it does.

**For video creatives:**
1. Pull transcript via `get_creative_transcript` — this is the most valuable data for explaining performance.
2. Break down the script arc: Hook (0-3s), Problem/Agitation, Solution, Social Proof, CTA.
3. Name the messaging angle: "Primary Angle: Fit Frustration → Product Solution" not just "UGC testimonial."
4. Connect to psychological mechanism: "The question-style hook filters for people already in consideration mode — everyone who clicks already has intent."

**For static creatives:**
1. Pull `get_creative_summary` for AI-detected format and messaging.
2. Analyze the visual format and copy structure.
3. Identify the value proposition angle.

**Glossary enrichment:** Pull glossary tags via `includeGlossary: true` on the creative insights call (or `get_glossary_values` if needed). Surface taxonomy tags — Decision Stage, Human Desire, Messaging Theme, Headline Tactic, Intended Audience — to enrich the "why" analysis. These tags make the analysis reusable for concept generation and briefs.

**Compare to account patterns:**
- "Your entire account runs UGC demos. This 3D format creates visual novelty — the pattern break itself earns attention."
- "This is the only creative using problem-agitation hooks — and it's your top ROAS performer."

**If the creative is underperforming**, diagnose using the 4-Question metric correlations from `evaluation-framework.md`:
- Low hook rate (thumbstop_ratio) → Q1 failing (doesn't make sense fast). Everything downstream is moot until this works.
- High hook rate + low hold rate (video_thruplay_ratio) → Q1 passes but middle loses people. Something after the opening creates friction or loses the thread. If thruplay is unavailable, check video retention percentiles (video_p25/p50/p75_watched_ratio).
- High hook rate + hold rate, low CTR → Q2 or Q3 failing. Viewers stay but don't act — CTA unclear or no compelling reason to click.
- High CTR + low conversion → Q4 failing. Ad promises something the landing page doesn't deliver.
- Strong engagement + low conversion → Q3 failing. Creative entertains but doesn't persuade.
- Declining spend + stable metrics → algorithm deprioritizing — audience saturation, needs refresh (not a Q1-Q4 issue).

## Phase 4: The Call

Conclude every analysis with a clear Ready / Iterate / Rethink call from `evaluation-framework.md`:

- **Ready** — Creative can keep running. Note what makes it work and why. Any observations are polish, not blockers.
- **Iterate** — Workable foundation, but specific problems need fixing. Name what to fix in priority order — conversion blockers first (Q4), attention failures second (Q1), trust gaps third (Q3). Be specific enough that the team can act without a follow-up conversation.
- **Rethink** — Something fundamental isn't working that surface fixes won't solve. Name exactly what's broken (the audience signal, core message, proof structure, or format) and why it can't be patched.

Don't soften Rethink to avoid disagreement. Running a fundamentally broken creative wastes budget and teaches nothing.

## Phase 5: Next Steps

Offer specific options based on what you found:

- **"Want me to analyze another ad?"** — the most common next step, keep the flow going
- **"Want me to find similar patterns?"** — search for creatives with matching taxonomy/format
- **"Want me to write hooks in this style?"** — if the hook is strong, generate variants
- **"Want me to brief an iteration?"** — if the creative is fatiguing but the angle works
- **"Want me to compare this to [competitor/other creative]?"** — if context suggests it

If the user has analyzed multiple ads in the session, offer: "You've analyzed [N] ads — want me to compare them or find patterns across what we've looked at?"

## Disambiguation & Error Handling

**Creative not found:** This happens frequently. Be helpful:
- "I couldn't find an ad matching '[name]' in your last 30 days of data. A few things to try: (1) Share the exact ad ID from Motion, (2) Check if the name matches exactly — sometimes there are version suffixes, (3) Try a different date range if this ad ran earlier."

**Multiple versions:** When a creative runs in different campaigns with different names, show ALL versions with their respective performance. Don't collapse them — the user needs to see each context.

**Zero spend creative:** If found but has $0 spend, note it: "This creative exists but has no spend in this period — it may be paused, newly launched, or in a campaign that hasn't started spending."

## Data Accuracy

Follow the creative-strategist Data Accuracy rules:
- Metrics are EXTRACTED (directly from the tool response).
- Benchmark comparisons and performance explanations are INFERRED — cite the data points.
- If a metric is missing or zero, flag it rather than omitting.
- Never fabricate transcript text, hook copy, or performance data.
