---
name: create-concepts
description: >
  Generate 2-5 data-driven creative concepts quickly from performance data and
  competitive signals. Use for straightforward "give me concept ideas", "what should
  we make", "what should we test next", or "ad ideas" requests. For raw brainstorm
  input, meeting notes, or deep strategic mapping, use concept-engine instead.
argument-hint: "[--count 3] [--focus hook|format|audience]"
allowed-tools:
  - Read
  - AskUserQuestion
  - "mcp__motion__get_auth_context"
  - "mcp__motion__get_workspace_brand"
  - "mcp__motion__get_creative_insights"
  - "mcp__motion__get_glossary_values"
  - "mcp__motion__get_workspace_competitors"
  - "mcp__motion__get_inspo_creatives"
  - "mcp__motion__get_creative_transcript"
  - "mcp__motion__get_inspo_brand_context"
model: opus
---

# Create Concepts

Generate data-driven creative concepts that bridge analysis to action. Every concept is grounded in performance data, informed by competitor intelligence, and specific enough to brief. Uses the creative-strategist skill methodology and concept quality standards.

---

## Phase 0: Orient

1. **Acknowledge**: "I'll generate ad concepts grounded in what's actually performing in your account."
2. **Ask direction**: "Any specific focus — hook-focused, format-focused, audience-focused? Or should I find the gaps and fill them?"
3. **Connected workflow**: After concepts, you can brief one (`/build-brief`), write hooks for it (`/write-hooks`), or go deeper on a specific angle.

If the user already specified the direction (e.g., "give me 3 UGC concepts focused on hooks"), skip questions and deliver.

---

## Phase 1: Setup

### 1a. Parse Arguments

- `--count`: Number of concepts to generate. Default: `3`. Range: 2-5.
- `--focus`: Optional weight for concept generation:
  - `hook` — emphasize hook innovation (new opening approaches)
  - `format` — emphasize format experimentation (new visual structures)
  - `audience` — emphasize audience expansion (new segments to target)

If no focus is specified, balance across all dimensions.

### 1b. Load Settings & Auth

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for org-specific configuration. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `default_date_preset`: LAST_30_DAYS
   - `default_creative_limit`: 10
   - `production_capabilities`: all formats
   - `max_production_difficulty`: High
   - `priority_glossary_categories`: use all
   - Brand guidelines + competitors: pull from Motion API
2. Call `get_auth_context()` + `get_workspace_brand(workspaceId)` in parallel.
3. Use `default_date_preset` from settings as the datePreset for all calls unless the user specified a date range. Use `default_creative_limit` from settings as the limit unless the user specified otherwise.
4. If settings specify `priority_glossary_categories`, prioritize those categories in pattern extraction and gap analysis.

**Settings influence:**
- Use `production_capabilities` and `max_production_difficulty` to constrain concept formats — don't propose full video production if the team only does UGC and static.
- Use `team_size` to calibrate production difficulty ratings.
- Use brand guidelines (voice, positioning, do's/don'ts, target audience) as hard constraints on concept generation.
- Use `primary_kpi` to weight which performance patterns matter most when grounding concepts.
- Brand settings supplement (not replace) `get_workspace_brand` results. If both exist, merge them — settings often has more specific creative rules.

### 1c. Load Methodology

Read the creative-strategist skill, `references/concept-standards.md`, and `references/insight-quality.md`.

---

## Phase 2: Data Collection (Parallel)

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

**Own performance:**
1. `get_creative_insights(workspaceId, "SPEND", datePreset="LAST_30_DAYS", limit=10, withAggregatedInsights=true)` — spend leaders + account-level aggregates

→ Extract `goalMetric` and `spendThreshold` from the response. Use goalMetric for all efficiency-sorted calls.

2. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset="LAST_30_DAYS", limit=10)` — efficiency leaders by workspace goal metric
3. `get_creative_insights(workspaceId, "SCALING", datePreset="LAST_30_DAYS", limit=10)` — what's growing
4. For hook rate leaders: filter SPEND results (call #1) to video creatives and sort by `thumbstop_ratio` descending. Do NOT use insightType="HOOK" — it returns the same ranking as SCALING.
5. `get_glossary_values(workspaceId, includeCreativeIds=true)` — creative taxonomy with creative mappings

**Competitor reference:**
6. `get_workspace_competitors(workspaceId)` — tracked competitors

After competitors return, pick the top 3 most active and dispatch per-competitor calls:
7. `get_inspo_creatives(brandId, workspaceId, limit=5, sort="NEWEST", launchDate="LAST_30_DAYS", status="ACTIVE")` — for each competitor
8. `get_inspo_brand_context(brandId, workspaceId)` — for the most relevant competitor

---

## Phase 3: Deep Dive

### 3a. Pull Transcripts

For the most interesting creatives (max 5-8 total transcript pulls):
- Top 2-3 own performers (by goalMetric or SCALING) that are video
- Top 2-3 competitor creatives that represent different approaches

Use `get_creative_transcript(creativeEntityId, creativeOrigin)`. Prioritize variety — don't pull transcripts for similar-looking creatives.

### 3b. Analyze Winning Patterns

From own performance data + glossary:
- Which glossary categories (format, hook type, asset type) correlate with top goalMetric?
- Which hooks appear in scaling creatives? What do they have in common?
- What messaging patterns do top performers share?
- What combinations haven't been tried? (coverage gaps in the taxonomy)

From competitor data:
- What approaches are competitors running that your account doesn't show?
- What formats or hooks are converging across the industry?
- Where are competitors leaving white space?

### 3c. Identify Audience Tensions

From brand context and performance patterns, derive 2-3 specific audience tensions — human lived experiences, not product features:
- What specific moment does your audience struggle with?
- What belief or assumption does your audience hold that your product challenges?
- What emotional state is your audience in when they'd be most receptive?

These tensions anchor the concepts. If `--focus audience`, generate more tensions and weight concepts toward untested segments from demographic data.

---

## Phase 4: Generate Concepts

For each concept, follow the structure in `references/concept-standards.md`:

### Concept Card

**Title:** Short, memorable name (2-5 words). Strategic essence, not execution description.

**Format:** Asset type (UGC, studio, hybrid) + visual format (testimonial, demo, problem-agitation, before-and-after, POV, etc.). Chosen intentionally — explain why this format serves this concept.

**Messaging Angle:** The consumer motivation being communicated. Specific to this brand, not category-generic.

**Intended Audience:** Specific enough to guide casting, tone, and message. Anchored to a tension from Phase 3.

**Hook:** The tactic (question, confession, bold claim, warning, curiosity, contrast) AND the verbatim opening line. For video: conversational, spoken aloud naturally. For static: scannable headline, 5-8 words.

**Why It Can Work:** 1-2 sentences connecting to behavioral insight or performance pattern. Not a sales pitch — a testable hypothesis. Must reference specific data: "Problem-agitation hooks drove 40% higher thumbstop in your account" or "No competitor is running this angle, creating white space."

**Reference Creatives:** 1-2 own top performers and/or competitor examples that inform this concept. Note what to learn from each.

**Production Difficulty:** Low / Medium / High.

### Generation Rules

- Every concept must be grounded in at least one data point from Phase 2-3
- At least 2 elements must vary across concepts (audience, angle, format, or hook tactic)
- No concept should replicate an existing ad from the account — cross-reference against performance data
- Each concept must pass the quality bar in `references/concept-standards.md`
- If `--focus` is specified, at least 2 concepts should emphasize that dimension
- Lead with the strongest or most differentiated concept

---

## Quality Gate

Before presenting concepts, verify each against the quality bar in `../creative-strategist/references/concept-standards.md`. Every concept must pass: pain point specificity, strategic coherence, differentiation, format ambition, persuasive sharpness, testable hypothesis, producible. Revise or discard any concept that fails. Fewer strong concepts beat many weak ones.

---

## Phase 5: Output

Open by reflecting what inputs shaped the concepts: brand context, performance patterns used, competitor signals incorporated. 2-3 sentences, conversational.

Present each concept as a numbered card with all fields from Phase 4.

**Close with a recommendation:** Which 2-3 concepts to brief first, based on:
- Expected impact (grounded in performance data)
- Production effort (lower effort = faster to market)
- Portfolio diversity (don't brief 3 versions of the same thing)

Format: "I'd brief **[Concept 1]** and **[Concept 3]** first — [one sentence rationale for each]."

If the user wants to turn a concept into a production brief, they can run `/build-brief` with the concept.

---

## Error Handling

Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/error-handling.md` for degraded mode and error handling guidance.
