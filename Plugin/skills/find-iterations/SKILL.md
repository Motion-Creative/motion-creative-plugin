---
name: find-iterations
description: >
  Identify top performers and fatiguing creatives, then suggest specific iterations —
  new hooks, format variations, audience pivots, and refreshes. Produces structured
  iteration cards ready for briefing. Activate when users ask to "find iterations",
  "what should we iterate on", "refresh ideas", "creative refresh", "what's fatiguing",
  "extend our winners", "what to test next", or need iteration opportunities from
  existing creative performance.
argument-hint: "[--datePreset LAST_30_DAYS] [--limit 10]"
allowed-tools:
  - Read
  - Agent
  - "mcp__ruimotion__get_auth_context"
  - "mcp__ruimotion__get_workspace_brand"
  - "mcp__ruimotion__get_creative_insights"
  - "mcp__ruimotion__get_glossary_values"
  - "mcp__ruimotion__get_demographic_breakdown"
  - "mcp__ruimotion__get_workspace_competitors"
  - "mcp__ruimotion__get_inspo_creatives"
  - "mcp__ruimotion__get_inspo_brand_context"
  - "mcp__ruimotion__get_creative_transcript"
  - AskUserQuestion
model: opus
---

# Find Iteration Opportunities

Identify what's working, what's dying, and what specific iterations to test next. Every iteration card is specific enough to brief immediately.

---

## Phase 0: Orient

1. **Acknowledge**: "I'll identify what's fatiguing and suggest specific iterations to test."
2. **Ask if needed**: "Any specific creatives you want to focus on, or should I scan everything and find the opportunities?"
3. **Connected workflow**: After finding iterations, you can brief one (`/build-brief`), write hooks for it (`/write-hooks`), or analyze the original ad (`/analyze-ad`).

If the user already specified what they want to iterate on, skip questions.

---

## Phase 1: Setup

### 1a. Parse Arguments

- **`--datePreset`**: Analysis window. Default: `LAST_30_DAYS`.
- **`--limit`**: Max creatives per metric. Default: `10`.

### 1b. Load Context

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for primary KPI, brand guidelines, production constraints, competitors. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `default_date_preset`: LAST_30_DAYS
   - `default_creative_limit`: 10
   - `priority_glossary_categories`: use all
   - Brand guidelines + competitors: pull from Motion API
2. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/SKILL.md` for methodology.
3. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/performance-metrics.md` for metric interpretation.
4. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/concept-standards.md` for quality bar.
5. Read `${CLAUDE_SKILL_DIR}/references/creative-diversity.md` for diversity audit methodology, exploitation vs exploration framework, and "What to Build Next" prioritization.
6. Use `default_date_preset` from settings as the datePreset for all calls unless the user provided a `--datePreset` argument. Use `default_creative_limit` from settings as the limit unless the user provided a `--limit` argument.
7. If settings specify `priority_glossary_categories`, prioritize those categories in taxonomy analysis and gap detection.

---

## Phase 2: Data Collection

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

1. `get_auth_context()` — resolve workspace
2. `get_creative_insights(workspaceId, insightType="SPEND", datePreset, limit, withAggregatedInsights=true)` — spend allocation

→ Extract `goalMetric` and `spendThreshold` from the response. Use goalMetric for all efficiency-sorted calls.

3. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset, limit)` — efficiency leaders by workspace goal metric
4. `get_creative_insights(workspaceId, insightType="SCALING", datePreset, limit)` — momentum
5. `get_creative_insights(workspaceId, insightType="HOOK", datePreset, limit)` — attention capture
6. `get_glossary_values(workspaceId, includeCreativeIds=true)` — creative taxonomy
7. `get_demographic_breakdown(workspaceId, datePreset)` — audience breakdown
8. `get_workspace_brand(workspaceId)` — brand context
9. `get_workspace_competitors(workspaceId)` — for competitor reference

Then for top 2-3 competitors:
10. `get_inspo_creatives(brandId, workspaceId, limit=5, sort="NEWEST", launchDate="LAST_30_DAYS")` + `get_inspo_brand_context(brandId, workspaceId)`

---

## Phase 3: Analysis

### 3a. Identify Candidates

**Top performers** — Strong goalMetric + meaningful spend. Proven creatives worth iterating to extend their life.

**Fatiguing creatives** — Were scaling but now declining, or high spend with dropping goalMetric. Need a refresh before they burn out.

**Hidden gems** — Strong goalMetric but low spend. Underexploited — iterate to help them scale.

**High-hook/low-convert** — Great thumbstop but poor goalMetric. The hook works, the body doesn't.

### 3b. Diagnose What's Working (Top Performers)

Pull transcripts for the top 3-5 performers (max 5-8 total calls). Prioritize variety and highest goalMetric.

Cross-reference with glossary tags:
- What hook approach are they using?
- What format/visual structure?
- What messaging angle?
- Which demographic segment responds best?

### 3c. Diagnose What's Fatiguing

For declining creatives, determine the cause:

- **Hook fatigue?** HOOK rate dropping → same audience has seen the opening too many times
- **Audience saturation?** SCALING declining → exhausted the target segment
- **Format staleness?** All top performers share a format competitors have also adopted
- **Messaging wear-out?** Same angle repeated across multiple creatives

### 3d. Map the Iteration Matrix

Using the Creative Strategy Engine framework and glossary taxonomy:
- Which pain × persona × awareness stage combinations are working?
- Which haven't been tried?
- What tag combinations exist in winners but not in the broader catalog?
- Where can you take a winning element and combine it with something untested?

### 3e. Diversity Audit

Reference `references/creative-diversity.md` for the full methodology. Assess the portfolio across five dimensions:

- **Coverage**: How many distinct pain/desire buckets, personas, awareness stages, and messaging angles are represented?
- **Concentration**: What % of spend or top performers fall into a single matrix cell? A single pain/desire bucket? A single awareness stage? Multiple winners in the same cell = proven but unexpanded. All winners at Stage 4-5 = harvesting, not building demand.
- **Format spread**: How many distinct asset types and visual formats? Is one format carrying disproportionate weight?
- **Hook diversity**: Across top performers, how many distinct hook tactics and psychological triggers? If all winners use the same tactic, that is a signal — either untested alternatives or genuine dominance.
- **Stage balance**: TOFU (Stage 1-2) vs MOFU (Stage 3) vs BOFU (Stage 4-5) coverage. All bottom-funnel = cannot scale.

Map existing creative back onto the matrix. Look for concentration signals (everything that works is in one region) and gap signals (entire pain/desire buckets, personas, or stages with no representation).

### 3f. Exploitation vs Exploration Assessment

Assess whether the portfolio is balanced between deepening proven territory and expanding into new territory.

**Exploitation** (iterating on proven): Lower risk, predictable returns, but limited growth ceiling.
**Exploration** (testing new matrix cells): Higher variance, but required for growth.

**Warning signs of too much exploitation:**
- All new creative is a variation of the same concept
- The winning angle hasn't changed in months
- Top performers are all in the same matrix cell
- Account looks efficient but isn't growing

**Warning signs of too much exploration:**
- No pattern has emerged across tests
- Many one-off concepts, no consistent winners
- Account is learning but not compounding

A healthy portfolio does both — most volume runs on proven angles while a consistent portion tests new territory. Call out which direction the current portfolio skews and what the implication is.

---

## Phase 4: Generate Iteration Cards

For each opportunity, produce a structured card:

### Iteration Card Format

**Source Creative**: Name/ID of the creative being iterated on
**What's Working**: Specific element to preserve — hook, format, angle, audience
**Iteration Type**: One of:
- **Hook Refresh** — New opening on same body/format
- **Format Swap** — Same message in a different creative format
- **Audience Pivot** — Same creative approach for a different demographic or persona
- **Messaging Angle Shift** — New angle on the same pain/desire
- **Full Refresh** — New concept inspired by what made the original work

**Specific Change**: Exactly what to change and what to keep
**Target Audience**: Who this iteration targets (same or new segment)
**New Hook** (if applicable): Verbatim opening line
**Why This Iteration**: Data-grounded rationale — connect to a performance finding or taxonomy gap
**Expected Impact**: What metric should improve and why
**Production Difficulty**: Low / Medium / High

### Generation Rules

- Generate 5-8 iteration cards total
- Each must be specific enough to brief immediately
- Mix of iteration types (don't generate 5 hook refreshes)
- At least 1 card for a fatiguing creative, 1 for a hidden gem, 1 for a top performer
- Reference competitor approaches where relevant (learn, don't copy)

### What to Build Next — Prioritization

Use the diversity audit and iteration matrix to prioritize cards in this order:

1. **Proven cells with no recent creative** — Build more here first. You know it works; you are leaving demand uncaptured.
2. **Adjacent cells to winners** — Similar persona or pain/desire bucket to a winner but a slightly different intersection. Likely responsive, lower risk to test.
3. **Completely untested cells** — Highest uncertainty, but required for expansion. Worth running smaller tests to qualify before scaling.
4. **Missing funnel stages** — If Stage 1-2 is absent and budget allows, this is a structural gap worth closing. Top-of-funnel coverage is what makes scaling possible.

Prioritize by: known signal first, adjacent hypotheses second, net-new exploration third. Every iteration card should cite which bucket it falls into.

---

## Phase 5: Output

### Report Structure

**If You Read Nothing Else** (3-5 bullets)
The highest-leverage iterations. What to test first.

**Top Performers to Iterate**
For each: what makes it work, 1-2 specific iterations with rationale.

**Fatiguing Creatives to Refresh**
For each: diagnosis of what's declining and why, specific refresh approach.

**Diversity Audit**
Portfolio health snapshot: coverage breadth, concentration risk, format spread, hook diversity, funnel stage balance. Call out the biggest concentration risk and the highest-value gap.

**Exploitation vs Exploration Balance**
Which direction the portfolio skews, what that means for growth, and the recommended shift.

**Untested Combinations**
Gaps in the glossary/taxonomy matrix — formats, angles, or audience segments that haven't been tried with proven elements.

**Iteration Priority**
Ranked list of all iteration cards by expected impact. Each card tagged with its prioritization bucket (proven cell, adjacent, untested, missing stage). Include production difficulty.

### Close

"Want me to develop any of these into full briefs? Pick a card and I'll build it out with `/build-brief`."

---

## Error Handling

Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/error-handling.md` for degraded mode and error handling guidance.
