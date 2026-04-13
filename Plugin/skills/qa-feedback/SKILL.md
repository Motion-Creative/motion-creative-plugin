---
name: qa-feedback
description: >
  Review a creative asset against brand guidelines, performance patterns, concept standards,
  and strategic framework. Provides structured, actionable feedback with a scorecard.
  Activate when users ask to "review this ad", "QA this creative", "give feedback on",
  "review this asset", "check this against brand guidelines", "creative review",
  "is this ready to launch", or share a creative for review.
argument-hint: "<asset URL, screenshot, or creative ID> [--type video|image|static|carousel] [--brief original brief text]"
allowed-tools:
  - Read
  - "mcp__motion__get_auth_context"
  - "mcp__motion__get_workspace_brand"
  - "mcp__motion__get_glossary_values"
  - "mcp__motion__get_creative_insights"
  - "mcp__motion__get_creative_transcript"
model: opus
---

# Creative QA & Feedback

Review a creative asset across 6 dimensions: brand alignment, hook quality, strategic positioning, concept integrity, production quality, and brief adherence. Every piece of feedback connects to data, brand guidelines, or the strategic framework.

---

## Phase 1: Get the Asset

### 1a. Parse Arguments

- **Asset** (`$ARGUMENTS`): The creative to review. Can be:
  - A URL to the ad
  - A screenshot description
  - A creative entity ID from Motion
  - A description of the asset
- **`--type`**: video, image, static, carousel, or unknown. Default: unknown (infer from context).
- **`--brief`**: The original brief this asset was built from. Optional but enables brief adherence check.

If no asset is provided, ask the user what they want reviewed.

### 1b. Load Context

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for brand voice, creative do's/don'ts, target audience. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `default_date_preset`: LAST_30_DAYS
   - Brand guidelines: pull from `get_workspace_brand`
2. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/SKILL.md` for methodology.
3. Use `default_date_preset` from settings as the datePreset for all calls unless the user specified a date range.
4. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/concept-standards.md` for quality bar.
5. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/insight-quality.md` for insight quality standards.
6. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/evaluation-framework.md` for the 4-Question Evaluation Framework, metric pattern correlations, Ready/Iterate/Rethink call, and multi-persona feedback methodology.

---

## Phase 2: Pull Comparison Data

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

1. `get_auth_context()` — resolve workspace
2. `get_creative_insights(workspaceId, insightType="SPEND", datePreset="LAST_30_DAYS", limit=5, withAggregatedInsights=true)` — spend leaders + account-level aggregates

→ Extract `goalMetric` and `spendThreshold` from the response. Use goalMetric for all efficiency-sorted calls.

3. `get_workspace_brand(workspaceId)` — brand guidelines, voice, positioning, do's/don'ts
4. `get_glossary_values(workspaceId)` — creative taxonomy categories
5. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset="LAST_30_DAYS", limit=5)` — efficiency leaders by workspace goal metric for comparison
6. `get_creative_insights(workspaceId, insightType="HOOK", datePreset="LAST_30_DAYS", limit=5)` — best hooks for comparison

If the asset is a video with a creative entity ID:
7. `get_creative_transcript(creativeEntityId, creativeOrigin)` — pull the transcript for detailed review

---

## Phase 3: Review Framework

### Step 1: 4-Question Evaluation (Primary Lens)

Before scoring individual dimensions, run the asset through the 4-Question Evaluation Framework from `evaluation-framework.md`. Answer each question in order — failing an earlier question overrides strengths in later ones:

1. **Does this make sense fast?** — Within the first 1-3 seconds, can the viewer understand what this is about?
2. **Will the right person feel like it's for them?** — Does the creative signal who it's for?
3. **Will they believe it?** — Are claims credible and supported, not just stated?
4. **Will they take the intended action?** — Is the next step obvious and compelling?

Use the metric pattern correlations from `evaluation-framework.md` to ground each answer in data when performance data is available from the comparison pull (Phase 2). If no performance data exists, evaluate based on creative execution alone and say so.

This establishes the strategic foundation. The detailed scoring in Step 2 adds granularity.

### Step 2: Detailed Scoring (6 Dimensions)

Evaluate the asset across 6 dimensions:

### Dimension 1: Brand Alignment (Pass / Needs Work / Fail)

- Does the copy match brand voice? (Check against `get_workspace_brand` + settings)
- Does messaging align with brand positioning?
- Are Creative Don'ts violated? **These are hard fails.**
- Are Creative Do's incorporated where natural?
- Is the visual style consistent with the brand?

### Dimension 2: Hook Quality (Pass / Needs Work / Fail)

- Does the first 3 seconds (video) or headline (static) create tension, curiosity, or recognition?
- Is it direct, pressured, specific? (Not generic or vague)
- Would this stop a scroll? Compare against account's top-performing hooks.
- Does the hook match the messaging angle's core truth?
- Classify the hook by tactic and trigger — is the right combination for this awareness stage?

### Dimension 3: Strategic Positioning (Pass / Needs Work / Fail)

Using the Creative Strategy Engine:
- Does this asset target a clear pain × persona intersection?
- What awareness stage is it aimed at? Is the approach appropriate for that stage?
- Does the messaging angle come through clearly?
- Is there a specific human tension driving the creative, or is it generic?

### Dimension 4: Concept Integrity (Pass / Needs Work / Fail)

Against Concept Standards:
- **Pain point specificity**: Built around a specific lived experience?
- **Strategic coherence**: Audience, message, format, and hook all align?
- **Differentiation**: Could a competitor use this without modification?
- **Persuasive sharpness**: Does it make the viewer feel personally implicated?

### Dimension 5: Production Quality (Pass / Needs Work / Fail)

- **Video**: Pacing, cuts, text overlay readability, audio quality, hook timing
- **Static**: Layout, typography hierarchy, visual clarity, CTA visibility
- **Both**: Professional enough for paid media?

### Dimension 6: Brief Adherence (Pass / Needs Work / Fail / N/A)

Only if `--brief` was provided:
- Does the asset match the brief's concept overview?
- Is the hook the one specified in the brief?
- Does the visual approach match?
- Are deliverable specs met (format, duration, dimensions)?

If no brief provided, mark as N/A.

### Step 3: Make the Call

Synthesize the 4-Question evaluation and the 6-dimension scoring into a single call using the Ready / Iterate / Rethink framework from `evaluation-framework.md`:

- **Ready** — The creative can run as-is. Any notes are polish, not blockers. Lead with what's strong.
- **Iterate** — There's a workable foundation but specific problems need fixing. Name what to fix in priority order — specific enough that the team can act without a follow-up conversation.
- **Rethink** — Something fundamental isn't working. Surface fixes won't save it. Name exactly what's broken and why it can't be patched.

The difference between Iterate and Rethink: Iterate means you can fix what's there. Rethink means the angle, the brief, or the approach needs to change — not just the execution. Don't soften Rethink to avoid disagreement.

---

## Phase 4: Output

### Overall Assessment

Map the call from Step 3 directly:
- **Ready** maps to **Ready to Launch**
- **Iterate** maps to **Needs Revisions**
- **Rethink** maps to **Rethink Approach**

### Score Card

| Dimension | Rating | Key Finding |
|---|---|---|
| Brand Alignment | Pass/Needs Work/Fail | [One-line summary] |
| Hook Quality | Pass/Needs Work/Fail | [One-line summary] |
| Strategic Positioning | Pass/Needs Work/Fail | [One-line summary] |
| Concept Integrity | Pass/Needs Work/Fail | [One-line summary] |
| Production Quality | Pass/Needs Work/Fail | [One-line summary] |
| Brief Adherence | Pass/Needs Work/Fail/N/A | [One-line summary] |

### Detailed Feedback

**Prioritization:** Lead with what will hurt performance most. Conversion blockers first (Q4). Attention failures second (Q1). Trust gaps third (Q3). Craft issues the viewer wouldn't notice in a feed scroll don't belong next to real problems. The order of feedback IS the prioritization — what leads is what the team should do first.

For each dimension rated "Needs Work" or "Fail":
- What specifically needs to change
- Why it matters (connect to performance data or brand guidelines)
- Concrete suggestion for how to fix it

**Keep feedback actionable.** "The opening line is unclear — try leading with the problem instead of the product" is actionable. "The hook needs work" is not. Every piece of feedback should tell the team what to change, not just what's wrong.

### Strengths

What's working well — be specific. If the hook is strong, say why. If the visual approach is fresh, explain what makes it different.

### Recommended Next Steps

1-3 specific, prioritized actions. Frame as "do this to improve [metric]" not just "fix this."

---

## Key Principles

- Be specific and constructive, not vague or harsh
- Connect every piece of feedback to data, brand guidelines, or the strategic framework
- A "Fail" requires a clear fix suggestion
- Compare against what's working in the account (top performers) not against abstract ideals
- If the asset is strong, say so clearly — don't find problems for thoroughness' sake
- Brand Don'ts violations are always hard fails regardless of creative quality

---

## Multi-Persona Feedback (Optional Mode)

If the user requests multi-persona review, or if stakeholders have conflicting views about a creative, use the multi-persona approach from `evaluation-framework.md`.

Each persona evaluates the creative against their stated priorities — not the same observations reworded in a different tone. Where personas agree, the signal is strong. Where they disagree, there's a tradeoff the team needs to decide on.

If the user asks for multi-persona feedback without defining personas, ask them to describe 2-4 personas with their priorities and concerns before proceeding.

---

## Error Handling

- **Brand guidelines unavailable**: Evaluate against general direct-response principles, note the gap.
- **Transcript fails for video**: Evaluate hook quality from description/URL only, note reduced depth.
- **Performance data sparse**: Compare against general best practices.
- **Asset is ambiguous**: Ask clarifying questions before reviewing.

## Degraded Mode

If any tool call fails:
- Note the gap explicitly
- Never fabricate comparisons
- Proceed with available data
- A review with clear caveats > no review
