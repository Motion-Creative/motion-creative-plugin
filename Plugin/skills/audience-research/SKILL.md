---
name: audience-research
description: Define audiences from tensions, mine customer reviews for real language, or analyze competitive creative for strategic signals. Covers tension-based audience mapping, 5-bucket review extraction, and competitive inspo research.
argument-hint: "[--mode definition|reviews|competitive] [--datePreset LAST_30_DAYS]"
allowed-tools:
  - Read
  - AskUserQuestion
  - "mcp__motion__get_auth_context"
  - "mcp__motion__get_creative_insights"
  - "mcp__motion__get_demographic_breakdown"
  - "mcp__motion__get_glossary_values"
  - "mcp__motion__get_workspace_brand"
  - "mcp__motion__get_workspace_competitors"
  - "mcp__motion__get_inspo_brand_context"
  - "mcp__motion__search_brands"
model: opus
---

# Audience Research

Three types of audience intelligence that feed creative strategy: defining who you're talking to, understanding how they actually talk and think, and reading what competitors are betting on.

---

## Phase 0: Orient

1. **Acknowledge**: "I'll help you understand your audience -- who they are, how they talk, and what competitors are doing."
2. **Detect mode** from `$ARGUMENTS` and user message:
   - `definition` -- who to target (tension-based audience mapping)
   - `reviews` -- how they talk (5-bucket review mining)
   - `competitive` -- what competitors bet on (competitive audience analysis)
   - `full` -- all three, sequentially
   - If ambiguous, ask: "Do you want audience definition, review mining, competitive analysis, or the full picture?"
3. **Connected workflows**: After audience work, the user can flow into `/create-concepts`, `/write-hooks`, or `/concept-engine` with the audience context established here.

If the user already specified a mode or their intent is clear, skip questions and proceed.

---

## Phase 1: Setup

### 1a. Parse Arguments

- `--mode`: One of `definition`, `reviews`, `competitive`, `full`. Default: detect from user message, or ask.
- `--datePreset`: Controls performance data window. Default: `LAST_30_DAYS`. Options: `LAST_7_DAYS`, `LAST_14_DAYS`, `LAST_30_DAYS`, `LAST_90_DAYS`.

### 1b. Load Settings & Auth

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for org-specific configuration. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `default_date_preset`: LAST_30_DAYS
   - Brand guidelines + competitors: pull from Motion API
2. Call `get_auth_context()` to resolve workspaceId.
3. Call `get_workspace_brand(workspaceId)` for positioning context -- brand voice, target audience, product details.
4. Use `default_date_preset` from settings as the datePreset for all calls unless the user provided a `--datePreset` argument.

**Settings influence:**
- Target audience from settings seeds the tension-mapping process -- start from what the brand already believes, then challenge and deepen it with data.
- Brand positioning shapes how tensions are framed (pain-led vs. desire-led).
- Primary KPI determines which performance metric anchors the "who's actually converting" analysis.

---

## Phase 2a: Audience Definition (if mode = definition or full)

### Methodology: Start with Tensions, Not Demographics

Don't start with demographics. Start with tensions.

A tension is the specific friction point that creates purchase intent for this product. It's not "busy parents" -- it's "parents who've stopped cooking dinner because they're out of ideas by 6pm and too tired to think." The more specifically you can name what someone is experiencing, the more useful the audience definition.

For each potential audience, ask:
- What specific situation puts them in the market for this?
- What have they already tried that didn't work?
- What would make them skeptical of an ad?
- Why would this product feel meaningfully different from what they've tried?

If you can't answer these for a given segment, the segment isn't specific enough.

### Existing vs. Growth Audiences

**Existing audiences** already believe the category works -- they just need a reason to choose this brand. Creative for existing audiences leads with proof, benefits, and differentiation. It doesn't need to convince them the problem is real.

**Growth audiences** don't yet see themselves as a customer. Creative for growth audiences starts in their world -- with their identity or situation -- before introducing the product. It earns the right to make a product claim by first demonstrating it understands them.

When mapping multiple audiences, flag which are existing and which are growth. The distinction changes what the creative needs to accomplish.

### Multiple Segments

When producing multiple audience definitions, make the differences meaningful. Each should represent a different tension, not a variation on the same one.

The test: could each segment respond to the same creative and for the same reason? If yes, they're the same audience.

Make the contrast between segments explicit. If one is an existing buyer and another is a growth audience, call that out -- the creative implications are completely different.

### Quality Bar

Every definition must:
- Name a specific situation, not a demographic group
- State the tension in the person's own terms, not the brand's terms
- Be specific enough to guide casting, tone, and message selection
- Include enough skepticism context that downstream creative execution knows what to avoid

A definition that could apply to most brands in the category with minor word swaps is too generic.

### MCP Data Grounding

Ground tension-based segments in actual performance data:

1. `get_creative_insights(workspaceId, insightType="SPEND", datePreset, limit=10, withAggregatedInsights=true)` -- spend leaders + account-level aggregates

→ Extract `goalMetric` and `spendThreshold` from the response. Use goalMetric for all efficiency-sorted calls.

2. `get_demographic_breakdown(workspaceId, datePreset)` -- which age/gender segments drive best performance? Use this to validate or challenge tension hypotheses.
3. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset, limit=10)` -- which creatives convert most efficiently by workspace goal metric? What audiences do their names, adset names, and campaign names suggest?
4. `get_glossary_values(workspaceId)` -- which audience tags already exist in the creative taxonomy? What's been tested vs. what's a gap?

Synthesis pattern: "Your data shows 25-34 women drive your best goalMetric -- here's the tension that explains why..." Connect the quantitative signal to the qualitative tension.

---

## Phase 2b: Review Mining (if mode = reviews or full)

Customer reviews are one of the fastest ways to understand how real people actually think and talk. They show you the real problem in their own words, what almost stopped them, what finally convinced them, what changed after they bought, and how they describe themselves in relation to the product.

Reviews don't replace performance data. They make your creative feel true.

### Getting Review Data

Reviews are not available through Motion MCP. The user must supply them:
- Pasted directly into the conversation
- Provided as a file path
- Referenced by URL (if URL, ask the user to paste the content -- we can't fetch external URLs)

If the user hasn't provided review data and this mode is active, ask: "I'll need customer reviews to mine. Can you paste them, share a file, or point me to a source?"

### The 5 Buckets

Organize extraction into five buckets. Run each separately -- don't mix them.

**1. Pain Points** -- What problem were they experiencing before? Look for how long they'd had it, what they'd tried before, how it affected their life, and the emotional weight of living with it.

**2. Trigger Moments** -- What finally made them buy? The specific moment, event, or realization that pushed them over the edge. Life events (wedding, diagnosis), recommendations, hitting a breaking point, running out of patience with other solutions. These are often the most powerful hook material.

**3. Objections Before Purchasing** -- What almost stopped them? In positive reviews, objections almost always appear in past tense: "I was skeptical but..." or "I almost didn't try it because..." These are gold for objection-handling copy.

**4. Transformations** -- What changed after? The specific result, the emotional shift (confidence, relief, freedom), and how they describe it in their own words. The more visceral and specific, the more useful.

**5. Standout Language** -- Exact phrases worth stealing for ads. Pull the lines that made you stop while reading: unusually specific descriptions, before/after language that is visceral and concrete, phrases that could work as a hook with zero editing. Keep all verbatim quotes here rather than scattered throughout the other buckets -- this is where the raw material lives.

When the review set contains multiple products, run all five buckets separately per product.

### Separating Signal From Noise

- Repeated ideas across many reviews = strong signal
- The same idea across multiple platforms = stronger
- Old reviews = less reliable
- Extreme comments = directional, not definitive

A single brilliant, funny, or painfully honest line can still be worth using -- just don't build your entire strategy on one anecdote.

### Guardrails

- Don't treat reviews as performance proof. They show what people say and feel.
- Don't over-generalize from one loud voice.
- Don't clean up the language so much that it loses personality.
- Don't group themes unless each group clearly implies creative direction.
- Don't fabricate or rewrite quotes in a way that changes their meaning.

### Downstream Handoff

Each bucket feeds specific downstream skills:
- **Pain Points** --> Creative Strategy Engine (pain/desire anchor mapping)
- **Trigger Moments** --> `/write-hooks` (emotional readiness = powerful hook material)
- **Objections** --> `/write-hooks` (Risk Reversal, Contrarian, objection-handling tactics)
- **Transformations** --> `/write-hooks` (aspirational and social proof hooks)
- **Standout Language** --> `/write-hooks` (verbatim ad copy, phrases that work as hooks with zero editing)

---

## Phase 2c: Competitive Audience Analysis (if mode = competitive or full)

### Data Collection

1. `get_workspace_competitors(workspaceId)` -- get tracked competitors. Take the top 3 by relevance.
2. Per competitor: `get_inspo_brand_context(brandId, workspaceId)` -- their positioning, who they target, their brand voice and strategy.

If no competitors are tracked, tell the user: "You don't have competitors set up in Motion. Add them in Motion's Inspo tab, or tell me a brand name/domain and I'll search for it." Fall back to `search_brands(query, workspaceId)` if the user provides a name.

### Analysis

For each competitor:
- What audiences are they targeting? Infer from their positioning, creative themes, and brand context.
- What tensions are they naming? What problems or desires does their messaging address?
- What's absent? What audiences or tensions are they ignoring?

### Cross-Reference With Own Data

Pull `get_demographic_breakdown(workspaceId, datePreset)` for the user's own account. Compare:
- Segments competitors target that the user doesn't -- potential growth audiences
- Segments the user owns that competitors don't -- potential defensible positions
- Segments everyone targets -- crowded territory, need sharper differentiation

### Pattern Reading

**Convergence** -- Multiple competitors targeting the same audience = strong category signal. The audience is real and responsive. Differentiation must come from message, not audience selection.

**Gaps** -- Audiences nobody is targeting = whitespace opportunity OR proven dead-end. Don't call a gap an opportunity without acknowledging both possibilities. If you can explain why the audience would respond to this product, it leans toward opportunity. If you can't, it leans toward dead-end.

**Critical caveat:** Competitor creatives have no performance data. You're reading bets, not results. Frame competitive audience insights as "they're targeting this" not "this is working for them."

---

## Phase 3: Synthesis

### Map to Creative Strategy Matrix

Using the Creative Strategy Engine framework (pain/desire x persona x awareness stage):

1. Place each identified audience segment on the matrix.
2. For each cell, note whether it's backed by data (performance data, review evidence, competitive signal) or is a hypothesis.
3. Flag which segments are **existing audiences** (already converting in the account data) vs. **growth audiences** (identified through tension mapping or competitive gaps but not yet proven in performance).

### Identify Strategic Priorities

- Which audience segments have the strongest data support? (These should be the first creative bets.)
- Which segments are hypotheses worth testing? (Frame as experiments, not recommendations.)
- Where do review insights reinforce or contradict performance data? (Alignment = high confidence. Contradiction = worth investigating.)

---

## Phase 4: Output

### Audience Definition Cards (one per segment, if Phase 2a ran)

For each segment:
- **Tension**: The specific friction point, in the person's terms
- **Persona**: What their day looks like, what they've tried, what they believe
- **Type**: Existing (already converting) or Growth (untapped)
- **Skepticism Context**: What would make them distrust an ad? What should creative avoid?
- **Awareness Stage**: Where they sit on the awareness spectrum (unaware through most-aware)
- **Data Grounding**: What performance data supports this segment (demographic performance, top creative signals, glossary tags)
- **Creative Implications**: What the creative needs to accomplish for this audience -- lead with proof (existing) or start in their world (growth)

### Review Mining Summary (if Phase 2b ran)

Present the 5 buckets with:
- 3-5 key findings per bucket, organized by strength of signal
- Verbatim quotes preserved in Standout Language
- A "Downstream Actions" section mapping the strongest findings to specific skills: which pain points feed `/concept-engine`, which trigger moments feed `/write-hooks`, which objections inform hook tactics

### Competitive Audience Map (if Phase 2c ran)

- Per-competitor summary: who they target, what tensions they name, what's absent
- Convergence zones (everyone targeting) with differentiation implications
- Gap zones (nobody targeting) with opportunity/dead-end assessment
- Cross-reference with own demographic data

### Recommended Next Steps

Be specific based on findings:
- "Your strongest audience tension is [X]. Run `/create-concepts --focus audience` to generate concepts for this segment."
- "The review mining surfaced 4 hook-ready phrases. Run `/write-hooks` and feed in these trigger moments."
- "Competitors are all targeting [X] but nobody is addressing [Y]. Run `/concept-engine` to explore that whitespace."

---

## Error Handling

- **Auth fails**: Stop and tell the user to check their Motion connection. Don't proceed without workspace context.
- **Demographic data unavailable**: Note the gap explicitly. Proceed with qualitative tension analysis but flag reduced confidence in audience sizing.
- **No review data provided for review mining**: Ask the user to paste reviews or provide a source. Don't fabricate or assume review content.
- **No competitors tracked**: Suggest adding competitors in Motion's Inspo tab. Offer to search by brand name/domain as fallback.
- **Few competitors (1-2)**: Proceed but note limited competitive sample. Don't over-interpret patterns from thin data.
- **Glossary empty**: Note that no audience taxonomy exists yet -- this analysis will help establish one. Proceed without taxonomy grounding.
- **All tools fail**: Say so clearly. Offer to do qualitative tension analysis from brand context alone, but flag that no data grounding is available.

## Degraded Mode

If any MCP tool call fails:
- Note the gap explicitly in the output
- Never fabricate performance data or fill gaps with assumptions
- Proceed with whatever data is available
- If all tools fail, conduct qualitative analysis from brand context and user-provided information only
