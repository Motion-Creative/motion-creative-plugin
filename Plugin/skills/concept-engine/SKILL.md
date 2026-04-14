---
name: concept-engine
description: >
  Advanced concept generation from unstructured input — brainstorm notes, meeting
  transcripts, competitor observations, or a blank-slate strategic exploration. Maps
  ideas through the Creative Strategy Engine framework (pain x persona x awareness stage)
  and grounds every concept in Motion performance data. Use when the user has raw input
  or wants deep strategic mapping. For quick "give me 3 concepts" requests without raw
  material, use create-concepts instead.
argument-hint: "[raw idea or direction] [--count 3] [--focus hook|format|audience]"
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
  - "mcp__ruimotion__search_brands"
  - "mcp__ruimotion__get_brand_by_domain"
  - AskUserQuestion
model: opus
---

# Concept Engine — Data-Grounded Creative Concept Generator

---

## Phase 0: Orient

1. **Acknowledge**: "I'll map your input through the creative strategy framework and produce grounded concepts."
2. **Ask for input**: "What's the raw input — brainstorm notes, meeting transcript, competitor observation, or blank slate?"
3. **Connected workflow**: After concepts, you can brief one (`/build-brief`), write hooks for it (`/write-hooks`), or iterate on a specific angle.

If the user already provided raw input in their message, skip questions and proceed.

---

Transform any input into structured, briefable ad concepts grounded in performance data and the Creative Strategy Engine framework.

---

## Phase 1: Discovery

### 1a. Parse Arguments

- **Raw input** (`$ARGUMENTS`): The source material. Can be anything — a raw idea, meeting transcript, brainstorm notes, competitor observation, a single sentence, or empty.
- **`--count`**: Number of concepts to generate. Default: 3. Range: 2-5.
- **`--focus`**: Weight for concept generation:
  - `hook` — emphasize hook innovation
  - `format` — emphasize format experimentation
  - `audience` — emphasize audience expansion
  - Default: balanced across all dimensions.

### 1b. Load Context

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for brand guidelines, production constraints, primary KPI, target audience. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `default_date_preset`: LAST_30_DAYS
   - `default_creative_limit`: 10
   - `production_capabilities`: all formats
   - `max_production_difficulty`: High
   - Brand guidelines: pull from `get_workspace_brand`
2. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/SKILL.md` for methodology.
3. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/concept-standards.md` for quality bar.
4. Read `${CLAUDE_SKILL_DIR}/references/creative-strategy-engine.md` for the full strategic framework.
5. Use `default_date_preset` from settings as the datePreset for all calls unless the user specified a date range. Use `default_creative_limit` from settings as the limit unless the user specified otherwise.

### 1c. Pull Performance Data

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

1. `get_auth_context()` — resolve workspace
2. `get_creative_insights(workspaceId, insightType="SPEND", datePreset="LAST_30_DAYS", limit=10, withAggregatedInsights=true)` — spend leaders + account-level aggregates

→ Extract `goalMetric` and `spendThreshold` from the response. If `goalMetric.isCustomConversion` is true, find the matching conversion in the `customConversions` array and include `["{id}_cost", "{id}_count"]` in `tableKPIs` on all subsequent calls. Use goalMetric for all efficiency-sorted calls.

3. `get_workspace_brand(workspaceId)` — brand positioning, voice, constraints, creative do's/don'ts
4. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset="LAST_30_DAYS", limit=10)` — efficiency leaders by workspace goal metric
5. `get_creative_insights(workspaceId, insightType="SCALING", datePreset="LAST_30_DAYS", limit=10)` — what's scaling
6. For hook rate leaders: filter SPEND results (call #1) to video creatives and sort by `thumbstop_ratio` descending. Do NOT use insightType="HOOK" — it returns the same ranking as SCALING.
7. `get_glossary_values(workspaceId, includeCreativeIds=true)` — creative taxonomy
8. `get_demographic_breakdown(workspaceId)` — audience insights

Then pull transcripts from the top 3 performers for hook/format reference:
9. `get_creative_transcript(creativeEntityId, creativeOrigin)`

If the raw input references competitors or specific brands:
10. `get_workspace_competitors(workspaceId)` or `search_brands(query, workspaceId)` or `get_brand_by_domain(brandUrl)`
11. `get_inspo_creatives(brandId, workspaceId, limit=10, sort="NEWEST")` — competitor creative reference
12. `get_inspo_brand_context(brandId, workspaceId)` — competitor strategy

---

## Phase 2: Analysis

### 2a. Extract Ideas from Raw Input

If raw input was provided, parse it for:
- **Explicit creative ideas** — "we should try...", "what about...", "I saw this ad that..."
- **Implicit opportunities** — pain points mentioned, audience insights, product angles
- **Strategic directions** — messaging shifts, new audiences, format experiments

If raw input is empty or minimal, skip to Phase 2b — the performance data becomes the primary source.

### 2b. Map the Strategic Landscape

Using the Creative Strategy Engine framework:

**Step 1: Identify the Primary Anchor**
- Is the brand pain-led (solving a functional problem) or desire-led (aspirational)?
- What are the core pain/desire buckets?

**Step 2: Map Personas**
- For each relevant pain/desire, identify 3-5 persona segments
- Define each by: demographics, psychographics, and how they experience this specific pain/desire differently

**Step 3: Define Messaging Angles**
- For each pain × persona intersection, define the core conversational truth
- Not marketing copy — real language a real person would say
- Answer: How does THIS person experience THIS pain? What do they REALLY want? What would they doubt?

**Step 4: Determine Awareness Coverage**
- Which stages need content? Check existing creative coverage via glossary.
- Prioritize based on budget signals from the data (high spend = scaling budget, invest in unaware; lower spend = focus on problem/solution aware).

### 2c. Ground Each Idea in Performance Data

For each idea or direction:
- Does the account data support this direction? Which metrics? Which creatives?
- Has something similar been tried? Check glossary tags for existing coverage.
- What audience responds to related approaches? Cross-reference demographics.
- What gap does this fill in the taxonomy?

### 2d. Analyze Winning Patterns

From performance data + glossary:
- Which glossary categories correlate with top goalMetric?
- Which hooks appear in scaling creatives?
- What messaging patterns do top performers share?
- What combinations haven't been tried (coverage gaps)?

From competitor data (if available):
- What approaches are competitors running that the account doesn't show?
- What formats or hooks are converging across the industry?
- Where are competitors leaving white space?

### 2e. Identify Audience Tensions

From brand context and performance patterns, derive 2-3 specific audience tensions:
- What specific moment does the audience struggle with?
- What belief does the audience hold that the product challenges?
- What emotional state is the audience in when most receptive?

---

## Phase 3: Strategy

### 3a. Select Creative Mechanics

For each concept direction, choose a creative mechanic — the structural mechanism by which the concept delivers the truth to the viewer:

- **Implied Answer** — Visual silently answers a question the hook posed
- **Social Witness** — Third party's reaction validates the product
- **Trojan Horse** — Ad looks like content until the product appears
- **Reframe** — Flips the viewer's existing belief by the end
- **Contrast Without Comment** — Shows two realities without editorializing

The mechanic shapes what kind of hook you write and what visual format executes the concept.

### 3b. Map Strategic Positions

For each concept, define:
- **Pain/Desire anchor**: What pain or desire does this address?
- **Persona**: Who specifically is this for?
- **Messaging angle**: What's the core conversational truth?
- **Awareness stage**: Where in the funnel does this live?
- **Creative mechanic**: How does the concept deliver the truth?

---

## Phase 4: Direction

### Concept Card Structure

For each concept, produce:

**Title** (2-5 words) — Strategic essence, not execution description.

**Format** — Creative format + production style + visual structure. Chosen intentionally — explain why this format serves this concept.

**Messaging Angle** — The consumer motivation being communicated. Specific to this brand.

**Intended Audience** — Specific enough to guide casting, tone, and message. Anchored to a tension from Phase 2e.

**Hook** — The tactic (from hook-tactics library) AND the verbatim opening line.
- Video: conversational, builds into narrative, reads naturally spoken aloud
- Static: scannable headline, 5-8 words, complete on its own

**Why It Can Work** — 1-2 sentences connecting to behavioral insight or performance pattern. Must reference specific data.

**Reference Creatives** — 1-2 own top performers and/or competitor examples that inform this concept.

**Production Difficulty** — Low / Medium / High.

**Source** — Which part of the raw input inspired this concept (if applicable).

**Data Grounding** — What performance data supports this direction.

**Strategic Position** — Pain × persona × awareness stage from the framework.

### Generation Rules

- Every concept must be grounded in at least one data point from Phase 2
- At least 2 elements must vary across concepts (audience, angle, format, hook tactic)
- No concept should replicate an existing ad from the account
- Each concept must pass the quality bar from concept-standards.md
- If `--focus` is specified, at least 2 concepts should emphasize that dimension
- Lead with the strongest or most differentiated concept

### Quality Bar

Every concept must pass:
- **Pain point specificity**: Built around a specific lived experience, not a product feature
- **Strategic coherence**: Audience, message, format, and hook all align
- **Differentiation**: Couldn't be used by a competitor without modification
- **Format ambition**: Format chosen intentionally, contributes to persuasion
- **Persuasive sharpness**: Hook creates tension, viewer feels personally implicated
- **Testable hypothesis**: "Why it can work" is specific enough to succeed or fail clearly
- **Producible**: Creative team could execute without major clarification

### Handling Thin Context

If brand context is sparse or performance data is limited:
- Use direct response fundamentals to fill gaps
- Lean on category knowledge and proven structures
- Do not caveat, apologize, or flag missing context
- Concepts should be indistinguishable in quality from fully-informed outputs

---

## Phase 5: Articulation

### Output Format

**Input Summary** (2-3 sentences)
What the source material contained and what ideas emerged. Skip if no raw input.

**Concept Cards** (count × cards)
Each following the format above.

**Recommendation**
"I'd brief **[Concept X]** and **[Concept Y]** first — [data-grounded rationale for each]."

**Strategic Map**
Show where these concepts fit in the pain × persona × awareness stage matrix. Identify remaining gaps for future ideation.

### Close

"Want to turn any of these into a production brief? Run `/build-brief` with the concept, or I can develop it further."

---

## Error Handling

Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/error-handling.md` for degraded mode and error handling guidance.
