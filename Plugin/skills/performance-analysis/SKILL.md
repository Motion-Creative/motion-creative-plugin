---
name: performance-analysis
description: Analyze creative performance — what's working, what's scaling, what's dying. Multi-metric analysis with demographic breakdown and actionable recommendations.
argument-hint: "[--datePreset LAST_30_DAYS] [--limit 10] [--metric goalMetric]"
allowed-tools:
  - Read
  - Agent
  - "mcp__ruimotion__get_auth_context"
  - "mcp__ruimotion__get_creative_insights"
  - "mcp__ruimotion__get_demographic_breakdown"
  - "mcp__ruimotion__get_glossary_values"
  - "mcp__ruimotion__get_workspace_brand"
  - AskUserQuestion
model: opus
---

# Creative Performance Analysis

Analyze creative performance using the creative-strategist skill methodology — multi-metric landscape, demographic overlay, and creative taxonomy. Produce an actionable report of what's working, what's scaling, and what's dying.

---

## Phase 0: Orient

Before pulling data:

1. **Acknowledge**: "I'll analyze your creative performance to find what's working, scaling, and dying."
2. **Detect complexity**: Is this a quick question ("how's ROAS?") or a full deep dive ("what's working?")? Quick questions get 1-2 tool calls and a direct answer. Full analysis gets the multi-metric sweep.
3. **Ask if ambiguous**: "Quick top-line or full deep dive? Any specific metric or time period you care about?"
4. **Connected workflow**: After analysis, you can create concepts (`/create-concepts`), find iterations (`/find-iterations`), or dive into a specific ad (`/analyze-ad`).

If the user provides clear, specific intent (e.g., "full performance analysis for last 30 days"), skip questions and deliver.

---

## Phase 1: Setup

### 1a. Parse Arguments

- `--datePreset`: Time window for analysis. Default: `LAST_30_DAYS`. Options: TODAY, YESTERDAY, THIS_MONTH, LAST_MONTH, LAST_7_DAYS, LAST_14_DAYS, LAST_30_DAYS, LAST_90_DAYS.
- `--limit`: Max creatives per metric query. Default: `10`.
- `--metric`: Optional focus metric (SPEND, SCALING, HOOK, CPC, CTR_ALL, PURCHASES, PURCHASE_VALUE, or the workspace's goalMetric). If provided, lead the analysis with this metric. If not, use the standard multi-metric approach.

### 1b. Load Settings & Auth

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for org-specific configuration. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `default_date_preset`: LAST_30_DAYS
   - `default_creative_limit`: 10
   - `demographic_focus`: both
   - `primary_metrics` / `secondary_metrics` / `exclude_metrics`: auto-detect
   - `priority_glossary_categories`: use all
   - Brand guidelines: pull from `get_workspace_brand`
2. Call `get_auth_context()` to resolve workspaceId (use settings workspace_id as fallback context).
3. If settings contain a `primary_kpi` and no `--metric` was specified, use the primary KPI to lead the analysis.
4. If settings contain `target_demographics`, weight demographic analysis toward those segments.
5. If settings contain `primary_metrics`, ensure those metrics lead the analysis. If `secondary_metrics`, include after primary. If `exclude_metrics`, omit those from all queries and output.
6. Use `default_date_preset` from settings as the datePreset for all calls unless the user provided a `--datePreset` argument. Use `default_creative_limit` from settings as the limit unless the user provided a `--limit` argument.

If auth fails, stop and tell the user to check their Motion workspace connection.

### 1c. Load Methodology

Read the creative-strategist skill and `references/performance-metrics.md` for metric definitions and interpretation patterns.

---

## Phase 2: Data Collection

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

1. `get_creative_insights(workspaceId, insightType="SPEND", datePreset, limit, withAggregatedInsights=true)` — spend leaders + account-level aggregates

→ Extract `goalMetric` and `spendThreshold` from the response. If `goalMetric.isCustomConversion` is true, find the matching conversion in the `customConversions` array and include `["{id}_cost", "{id}_count"]` in `tableKPIs` on all subsequent calls. Use goalMetric for all efficiency-sorted calls.

2. `get_creative_insights(workspaceId, insightType="SCALING", datePreset, limit)` — what's gaining/losing allocation
3. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset, limit)` — efficiency leaders by workspace goal metric
4. `get_creative_insights(workspaceId, insightType="HOOK", datePreset, limit)` — thumbstop rate leaders
5. `get_demographic_breakdown(workspaceId, datePreset)` — age/gender performance
6. `get_glossary_values(workspaceId)` — creative taxonomy categories

If `--metric` is specified, ensure that metric is included and prioritized in the analysis.

---

## Phase 3: Analysis

### 3a. Build Creative-Level View

For each creative appearing across the results, collect:
- Its rank/position in each metric query it appeared in
- Key performance values (spend, goalMetric, hook rate, and other relevant metrics)
- Whether it's scaling, stable, or declining
- **Its campaign and ad set context** — `campaignName`, `campaignIds`, `adsetName`, `adsetIds`

Cross-reference to identify:
- **All-rounders:** Creatives that appear in top results across multiple metrics
- **Specialists:** High on one metric but absent from others (e.g., high HOOK but not in goalMetric top — hooks but doesn't convert)
- **Hidden gems:** Strong goalMetric but low spend — underexploited

### 3a-ii. Map Campaign & Ad Set Structure

This step is critical — **campaign and ad set context changes the meaning of every metric.**

1. **Group creatives by campaign**, then by ad set within each campaign
2. Apply campaign tier classification and structural issue detection from `references/campaign-context.md`

Consult `references/campaign-context.md` for campaign tier naming heuristics, ad set strategy signals, and structural issue patterns.

### 3b. Pattern Extraction

Use glossary values to tag each creative with its taxonomy categories. Then analyze:
- Which categories (format, hook type, asset type, messaging angle) are over/under-represented in top performers?
- Which categories are scaling vs. declining?
- What combinations appear in winners but not losers?
- What hasn't been tried? (categories with zero or minimal coverage)
- **Which campaign tiers favor which creative patterns?** (e.g., UGC dominates testing but studio images dominate scaling — or vice versa)

Consult `references/performance-metrics.md` for metric combination stories.

### 3c. Demographic Analysis

From the demographic breakdown:
- Which age/gender segments drive the best performance?
- Are there underserved segments where creative investment could expand?
- Do certain creatives skew strongly toward one demographic?

### 3d. Apply Brand Context from Settings

If config contains brand guidelines (Brand Voice, Creative Do's/Don'ts, Target Audience):
- Frame recommendations through brand constraints — don't suggest approaches that violate creative don'ts
- Weight audience insights toward the brand's stated target audience
- Use brand voice to inform how insights are framed

If settings contain `priority_glossary_categories`, prioritize those categories in pattern extraction (3b).

### 3e. Apply Insight Quality Bar

Before writing findings, check each insight against `references/insight-quality.md`. Every finding must explain the "why" — what's happening in the viewer's mind — not just state a metric.

---

## Phase 4: Output

Open by reflecting what data you pulled: time window, number of creatives analyzed, any filters applied. Keep it conversational — one sentence, not a formatted log.

### Report Structure

**If You Read Nothing Else**
3-5 bullets. The most actionable findings. Lead with what changed or what's surprising. Every bullet should make the reader want to do something.

**Top Performers**
Creatives with the best combination of goal metric efficiency + meaningful spend. Not just highest efficiency (which could be low-spend noise). Show the creative, its key metrics, **which campaign/ad set it runs in**, and WHY it works — the behavioral insight. Contextualize efficiency against the campaign tier (e.g., below-breakeven on cold prospecting may be acceptable if LTV justifies it; below-breakeven on scaling is a problem).

**Scaling Winners**
Creatives gaining spend allocation with stable or improving efficiency. These are the ones the algorithm is betting on. Note what they have in common. **Flag whether they're scaling in an uncapped environment (strong signal) or a cost-cap environment (inflated signal).**

**Efficiency Standouts**
Strong goalMetric but not yet scaling — hidden gems that deserve more budget. Explain why they might be underexploited. **Note their campaign context — a gem in a cost-cap ad set needs uncapped validation before you declare it a winner.** Recommend the graduation path: which campaign/ad set to move it to next.

**Campaign Structure Issues**
Structural problems in how campaigns and ad sets are organized that work against performance. This section surfaces:
- Testing creatives competing with proven winners inside the same CBO (budget starvation)
- "Top performers" ad sets containing fatigued creatives (budget traps)
- Creatives ready to graduate from Testing → Scaling
- CBO allocation conflicts where budget flows to high-spend losers over low-spend winners
- Cost-cap artifacts inflating efficiency metrics

**Declining Creatives**
Losing share or performance dropping. Don't just flag them — recommend what to do: refresh, pause, test new hooks on the same body, etc. **If a declining creative sits in a "Top performers" ad set, explicitly recommend removal — it's dragging down the ad set.**

**Demographic Sweet Spots**
Where performance concentrates by age/gender. Which segments are strongest, which are underserved, and what that means for creative strategy.

**Recommendations**
3-5 specific next actions. Each recommendation must be:
- Specific enough to act on today
- Grounded in a finding from the analysis
- Framed as "do this because the data shows X"
- **Campaign-aware** — recommendations should specify where creatives should move (e.g., "graduate BRoll TO from Testing 1 to Top performers ad set"), not just "increase budget"

---

## Error Handling

- **Auth fails:** Stop and tell user to check Motion workspace connection
- **One metric query fails:** Note the gap, continue with remaining data
- **All queries fail:** Report the failure, suggest checking the Motion MCP connection
- **Empty results:** Note that no creatives match the criteria — suggest broadening the datePreset or checking spend thresholds
- **Limited data (few creatives):** Adjust analysis depth — don't over-interpret small samples
