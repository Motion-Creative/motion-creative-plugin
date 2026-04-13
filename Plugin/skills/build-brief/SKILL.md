---
name: build-brief
description: Produce a structured creative brief for an ad concept — ready to hand to a designer or creative team. Takes a concept description or builds from performance data.
argument-hint: "<concept description or concept number from create-concepts>"
allowed-tools:
  - Read
  - AskUserQuestion
  - "mcp__ruimotion__get_auth_context"
  - "mcp__ruimotion__get_workspace_brand"
  - "mcp__ruimotion__get_glossary_values"
  - "mcp__ruimotion__get_creative_insights"
  - "mcp__ruimotion__get_inspo_creatives"
  - "mcp__ruimotion__get_creative_transcript"
  - "mcp__ruimotion__get_workspace_competitors"
  - "mcp__ruimotion__search_brands"
model: opus
---

# Build Brief

Produce a production-ready creative brief from a concept — structured so a creative team can execute without follow-up questions. Uses the creative-strategist skill methodology and brief template standards.

**Data accuracy**: Every performance reference in the brief must cite a specific data point. If the brief references "top-performing hooks," cite which hooks and their metrics. Never include performance claims not grounded in tool data. If a data point is missing (e.g., no demographic data to inform targeting), flag it rather than guessing.

---

## Phase 1: Get the Concept

### 1a. Parse Input

The `$ARGUMENTS` should contain one of:
- **A concept description** — freeform text describing what they want to make (e.g., "UGC problem-agitation ad targeting busy moms, hook about the 3pm energy crash")
- **A reference to a concept from a previous `/create-concepts` run** — the user might say "brief concept 2" or paste a concept title
- **Empty** — ask the user what to brief

If empty, use `AskUserQuestion`:
> "What creative concept do you want to brief? You can describe the idea, paste a concept from `/create-concepts`, or tell me the audience/angle you want to target and I'll build the concept."

### 1b. Get Auth Context

Call `get_auth_context()` for workspaceId.

---

## Phase 2: Gather Strategic Inputs (Parallel)

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

1. `get_creative_insights(workspaceId, "SPEND", datePreset="LAST_30_DAYS", limit=5, withAggregatedInsights=true)` — spend leaders + account-level aggregates

→ Extract `goalMetric` and `spendThreshold` from the response. If `goalMetric.isCustomConversion` is true, find the matching conversion in the `customConversions` array and include `["{id}_cost", "{id}_count"]` in `tableKPIs` on all subsequent calls. Use goalMetric for all efficiency-sorted calls.

2. `get_workspace_brand(workspaceId)` — brand positioning, voice, visual identity, tone constraints
3. `get_glossary_values(workspaceId)` — creative taxonomy for alignment
4. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset="LAST_30_DAYS", limit=5)` — efficiency leaders by workspace goal metric for visual reference

If the concept references a competitor or specific brand:
5. `search_brands(query, workspaceId)` or get competitor creatives for reference examples

### 2a. Load Settings & Methodology

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for org-specific configuration. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `default_date_preset`: LAST_30_DAYS
   - `brief_format`: default
   - `production_capabilities`: all formats
   - `max_production_difficulty`: High
   - Brand guidelines: pull from `get_workspace_brand`
2. Read the creative-strategist skill and `references/brief-template.md` for the output structure and quality bar.
3. Use `default_date_preset` from settings as the datePreset for all calls unless the user specified a date range.

**Settings influence:**
- If `brief_format` is "custom" and `references/custom-brief-template.md` exists, use that template instead of the default.
- Use `production_capabilities` and `max_production_difficulty` to constrain Deliverables section.
- Use brand guidelines (voice, do's/don'ts, positioning) as hard constraints on Copy and Visual Approach.
- Use `primary_kpi` to set the Success Metrics section — what "good" looks like depends on what the team optimizes for.
- Use `target_demographics` to inform audience-specific copy choices.

---

## Phase 3: Build the Concept (if needed)

If the user provided a full concept (from `/create-concepts` or detailed description), use it directly.

If the user provided a partial concept (just an audience, or just a hook idea), build it out:

1. Use the performance data from Phase 2 to identify relevant winning patterns
2. Use brand context to set voice and positioning constraints
3. Use glossary values to align with the workspace's creative taxonomy
4. Flesh out the concept following `references/concept-standards.md`

The concept must be clear before writing the brief — audience, hook approach, messaging angle, and format all need to be defined.

---

## Phase 4: Pull Reference Creatives

### 4a. Own Account Reference

From the goalMetric top performers (Phase 2), select the creative that best matches the brief's intended format and visual approach. This becomes the Visual Example in the brief — a real ad from the brand's account that shows "make it look like this."

### 4b. Competitor Reference (if relevant)

If the concept was inspired by competitor activity, pull 1-2 competitor creatives as additional reference:
- `get_inspo_creatives(brandId, workspaceId, limit=5, sort="NEWEST", status="ACTIVE")`

### 4c. Pull Transcripts

For the 2-3 most relevant reference creatives (own + competitor), pull transcripts with `get_creative_transcript`. These inform the Copy section — what hooks, pacing, and CTAs work in similar creatives.

---

## Phase 5: Generate the Brief

Follow the structure in `references/brief-template.md` exactly.

### Pre-Brief Reasoning

Before the brief, write 3-5 casual bullets showing the key inputs:
- What performance insight shaped the direction
- Who the creative is for and what tension it taps into
- What reference creatives informed the approach
- Any brand constraints that influenced choices

### The Brief

```
BRIEF: [Title]

CONCEPT OVERVIEW
[What we're making and why. 1-2 sentences. No jargon.]

COPY
[For video: timestamped script with spoken copy + text overlays per beat.
 For static: headline, subheading, supporting text, CTA.
 Write exactly what appears on screen or is spoken.
 No placeholders. The hook must be a specific verbatim line.]

VISUAL APPROACH
[For video: opening beat, background, pacing, production style, tone.
 For static: composition, layout, key visual elements.
 Specific enough that two designers would make similar things.]

DELIVERABLES
[Number of assets, format, dimensions, duration. No rationale.]

VISUAL EXAMPLE
[Reference creative from own account that shows how this should look.
 Include creative ID if available. Note if no strong match exists.]

SUCCESS METRICS
[Which insightTypes to watch. What "good" looks like.
 When to evaluate. What to diagnose if underperforming.]
```

### Hook Requirement

Every brief must include a specific, verbatim hook — the actual opening line. Not "start with a question about X" but the exact words. The hook must follow the persuasive attribute standard: direct, pressured, specific.

### Constraint Application

- All copy must match brand voice from `get_workspace_brand` AND settings brand guidelines (settings may have stricter rules)
- Messaging must align with brand positioning
- Creative Don'ts from settings are **hard constraints** — never include anything on the don't list
- Creative Do's from settings should be incorporated where natural
- Concepts must not replicate existing ads from the account (cross-reference with performance data)
- Flag any compliance concerns if the hook or body makes strong claims
- If settings specify `max_production_difficulty`, don't exceed it in the brief

---

## Quality Gate

Before presenting the brief, verify: Can a creator read this and NOT ask "what do you want me to make?" Check: Is the hook verbatim (not "start with a question about X")? Is the visual obvious on first read? Is the awareness stage clear in every line? Does every section change what gets made? Revise any section that fails.

---

## Phase 6: Output

Present the pre-brief reasoning bullets, then the brief itself. Render the brief as-is — do not add strategy sections, next steps, or concept cards after it.

End with: "This brief is ready for your creative team. Want me to adjust anything, or brief another concept?"

---

## Quality Bar

Before delivering, check against `references/brief-template.md` quality bar:

- **Can it be executed without a meeting?** If not, add detail.
- **Is the visual obvious immediately?** If not, be more specific in Visual Approach.
- **Does every section change what the creator does?** If a section is generic, cut it or make it specific.
- **Is the hook a real line, not a placeholder?** If it says "something like" or "[insert]", rewrite it.
- **Could a creator ask "what do you want me to make?"** If yes, the brief is not done.

---

## Error Handling

- **No concept provided and user skips AskUserQuestion:** Generate a concept from the top-performing creative patterns in the account. Note that the concept was auto-generated.
- **Brand context unavailable:** Proceed without voice constraints. Note the gap — copy may need voice adjustment.
- **No reference creatives match:** Note in Visual Example section that no strong match exists. Proceed without.
- **Transcript calls fail:** Write the Copy section from concept and brand context alone. Note reduced reference detail.
- **Very sparse data overall:** Generate the brief with reduced confidence. Flag: "Limited performance data — this brief is based on brand context and creative fundamentals rather than proven account patterns."
