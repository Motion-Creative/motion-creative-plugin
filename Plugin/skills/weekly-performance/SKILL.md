---
name: weekly-performance
description: >
  Generate a week-over-week creative performance slide deck comparing the last 7 days
  to the previous 7 days. Detects the week type (breakout, fatigue, shift, steady, launch),
  then builds a narrative arc around the biggest creative insight. Outputs a self-contained
  HTML slide deck. Activate when users ask for "weekly report", "weekly performance",
  "week over week", "WoW analysis", "weekly creative report", "performance deck",
  "weekly slide deck", or "what changed this week".
argument-hint: "[--output html|notion|slack|pdf|auto]"
allowed-tools:
  - Read
  - Agent
  - "mcp__ruimotion__get_auth_context"
  - "mcp__ruimotion__get_workspace_brand"
  - "mcp__ruimotion__get_creative_insights"
  - "mcp__ruimotion__get_demographic_breakdown"
  - "mcp__ruimotion__get_glossary_values"
  - "mcp__ruimotion__get_creative_summary"
  - AskUserQuestion
model: opus
---

# Weekly Performance Slide Deck

Generate a week-over-week creative performance analysis as a presentation-ready slide deck. The deck tells a story — it's not a data dump.

---

## Phase 0: Orient

1. **Acknowledge**: "Building your week-over-week creative performance deck."
2. **Ask format**: "HTML slide deck, Notion, or Slack? Anything specific you want highlighted?"
3. **Connected workflow**: After the deck, you can dive into specific ads (`/analyze-ad`), find iterations on fatiguing creatives (`/find-iterations`), or generate concepts to fill gaps (`/create-concepts`).

If the user already specified the format, skip the question and build.

---

## Phase 1: Setup

### 1a. Parse Arguments

- **`--output`**: Where to deliver the report. Default: `auto` (ask the user).
  - `html` — Self-contained HTML with CSS scroll-snap
  - `notion` — Structured document with headings, tables, callout blocks
  - `slack` — Concise highlights post (max 300 words)
  - `pdf` — Print-optimized HTML with page breaks
  - `auto` — Ask the user before building

### 1b. Load Context

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for org-specific config. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `demographic_focus`: both
   - `primary_metrics` / `secondary_metrics` / `exclude_metrics`: auto-detect
   - Brand guidelines: pull from `get_workspace_brand`
2. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/SKILL.md` for methodology.
3. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/performance-metrics.md` for metric definitions.
4. If settings contain `primary_metrics`, ensure those metrics lead the WoW analysis. If `exclude_metrics`, omit from all comparisons.

### 1c. Resolve Date Windows

Calculate two non-overlapping 7-day windows ending **yesterday** (never include today — partial data distorts metrics):
- **This week**: start = 8 days ago, end = yesterday
- **Last week**: start = 15 days ago, end = 9 days ago

Use YYYY-MM-DD format.

---

## Phase 2: Data Collection

### 2a. Prerequisites (Strict Sequence)

Call `get_auth_context()` → resolve workspaceId.

The first `get_creative_insights` response includes `goalMetric` (the workspace's preferred efficiency metric) and `spendThreshold` (minimum spend for statistical significance). Extract both. The goalMetric is the primary KPI for the entire deck.

Defaults if unavailable: settings `primary_kpi` for metric, $100 for threshold.

### 2b. Parallel Data Pulls

Once you have workspaceId and goalMetric, dispatch all in parallel:

1. **Brand context**: `get_workspace_brand(workspaceId)` — brand voice, creative constraints
2. **This week's creatives**: `get_creative_insights(workspaceId, insightType="SPEND", datePreset="CUSTOM", startDate="{this_week_start}", endDate="{this_week_end}", limit=20, withAggregatedInsights=true)`
3. **Last week's creatives**: `get_creative_insights(workspaceId, insightType="SPEND", datePreset="CUSTOM", startDate="{last_week_start}", endDate="{last_week_end}", limit=20, withAggregatedInsights=true)`
4. **Demographics**: `get_demographic_breakdown(workspaceId, datePreset="LAST_14_DAYS")` — 14-day aggregate (tool doesn't support custom ranges). Note this limitation.
5. **Glossary taxonomy**: `get_glossary_values(workspaceId, includeCreativeIds=true)`

### 2c. After Insights Return

Call `get_creative_summary` for the top 10 creatives by spend (this week) to get verbatim hooks and headlines. Use `spokenHook` for video, `headline` for static. Prefer `spokenHook` > `textOverlayHook` > `headline`.

### 2d. Derive Trends

Compare `aggregatedInsights` from both weeks for account-level WoW deltas: spend, goalMetric, CTR, CPA, thumbstop, impressions.

### 2e. Identify Launches

From this week's creatives, filter any with `launchDate` within the current week window. These are newly launched tests.

### 2f. Validate Before Proceeding

- If either pull returns fewer than 5 creatives above the spend threshold, flag the thin dataset and adjust scope.
- If the preferred metric has no usable values, fall back to spend-ranked order and state that explicitly.

---

## Phase 3: Detect the Week Type

Before building slides, classify the week. This determines the narrative arc. Evaluate in order, pick the first that fits:

**Breakout** — A creative that wasn't in last week's top 20 appeared in this week's top 10 AND outperforms account averages on goalMetric. Story: what's new and why it's working.

**Fatigue** — 3+ of last week's top 10 dropped out or declined >20% on goalMetric, AND nothing new replaced them at comparable efficiency. Story: what's dying and what needs to replace it.

**Shift** — The composition of the top 20 changed meaningfully: a different format, messaging angle, or demographic segment now dominates. Story: the direction change.

**Launch** — 5+ new creatives (launched within current week) appear with meaningful spend. Story: early reads on fresh creative.

**Steady** — None of the above. Story: the hidden signal everyone else would miss — the quiet riser, the subtle pattern, or the fatigue that hasn't hit yet.

---

## Phase 4: Build the Analysis

### 4a. Creative-Level Diff

For every creative in either week's top 20:
- Present in last week's top 20? This week's? Both? New? Dropped?
- Delta on: spend, goalMetric, CTR, thumbstop (if video)
- Verbatim hook/headline from `get_creative_summary`
- Glossary tags — how does its profile compare to the rest of the top 20?

### 4b. Pattern Extraction

Across both weeks' top performers:
- Tag combinations over-represented this week vs last
- What disappeared from the top 20?
- Multi-attribute clusters in winners (e.g., UGC + problem-first hook + testimonial)?
- What's absent from top performers that exists in the portfolio? (untested directions)

### 4c. Behavioral Interpretation

Every finding must answer "so what?" — translate metrics into viewer behavior:

| Signal | What It Means |
|---|---|
| High thumbstop + high hold rate | Opening promises and delivers. Viewers feel rewarded. |
| High thumbstop + low hold rate | Hook grabs but middle doesn't pay off. Viewers feel baited. |
| High CTR + low conversion | Ad sells the click but landing page doesn't close. |
| Spend increasing | Algorithm found evidence it works. Winning auctions. |
| Spend decreasing | Creative exhausting its audience or losing auctions. |
| High goalMetric + low spend | Algorithm found a pocket that works but can't scale it. |
| High spend + declining goalMetric | Budget leak. More urgent than any low-spend underperformer. |

Lead with the human reaction, support with the number.

### 4d. Demographic Layer

From 14-day aggregate: which segments drive spend and efficiency? Flag segments with >15% spend share or >20% goalMetric deviation. Note the 14-day limitation.

### 4e. Apply Brand Context

Frame everything through brand context:
- Write slide headlines in brand voice
- Filter recommendations against creative constraints
- If a winning pattern conflicts with brand guidelines, flag the tension

---

## Phase 5: Build the Slide Deck

Output as self-contained HTML. Each slide is a full-viewport section with CSS scroll-snap.

### Design Principles

- **The creative IS the slide.** Ads shown large (40-50% of slide area). Use `creativeUrl` as image/video source.
- **One insight per slide.** If you need more than 20 words of body text, the slide is doing too much.
- **Metrics are badges, not headlines.** Small contextual labels showing WoW delta.
- **Quote actual hooks/headlines.** Large type near the creative card.
- **Color-code direction.** Green for improvement, red for decline, neutral for flat.
- **Speaker notes.** Collapsible footer bar, 2-3 sentences with the behavioral "why."

### Slide Arc

**Slide 1: This Week**
Account-level WoW snapshot. Grounds the entire deck.
- Brand name, date range, week type as subtitle
- WoW deltas: total spend, impressions, goalMetric, CPA, CTR, thumbstop
- Color-coded arrows
- One framing sentence
- Week type framing line:
  - Breakout: "A new creative broke into the top 5"
  - Fatigue: "Multiple top performers are declining"
  - Shift: "The creative mix changed meaningfully"
  - Launch: "[N] new creatives entered the dataset"
  - Steady: "Stable surface, but there's a signal underneath"

**Slides 2-3: Winners & Losers** (1-2 slides)
The creatives driving the week's story. Per creative (2-4 per slide):
- Actual hook/headline in large type
- One metric badge: the WoW delta
- One line: what the creative does to the viewer

Feature selection by week type:
- Breakout: new risers vs what they displaced
- Fatigue: declining creatives, what originally made them work
- Shift: old vs new top performers
- Launch: highest-spend new creatives by signal strength
- Steady: hidden gem vs "safe" top performers

**Slide (conditional): Launches**
Include if 1+ creatives launched this week. Skip otherwise.
- Each launched creative: thumbnail, hook text, early metrics
- Label all metrics as "early reads"
- Group: promising, neutral, concerning

**Slide: Patterns**
One slide connecting evidence into a creative thesis.
- Not: "Hook Tactic: Problem-First increased 34%"
- Instead: "The audience is responding to ads that lead with a flaw"
- Name multi-attribute recipes if emerging
- Include demographic signal if relevant

**Slide: What's Next**
Always the final slide. 2-3 specific creative directions for next week.
Each must:
- Name format, hook approach, messaging angle
- Trace back to a finding from earlier slides
- Be specific enough to brief
- Not: "Test more UGC"
- Instead: "Shoot 2-3 confessional UGC pieces where the creator admits a problem with the product before explaining why they kept it"

### Conditional Adjustments

- Both Winners & Losers show same pattern → collapse to 1 slide
- Patterns has no clear thesis → replace with "What to Watch"
- Demographics show no meaningful deviation → don't force demographic angle
- Launches has 5+ creatives → expand to 2 slides
- **Maximum 6 slides.** If you need more, you're saying too much.

---

## Phase 6: Output

### Delivery

If `--output` is `auto`, ask: "Where would you like this report? I can build it as a slide deck, Notion page, Slack post, or shareable link."

### Response

Brief intro (1-2 sentences: the week type and biggest insight), then present the artifact.

### Follow-Up

Tied to week type:
- Breakout: "Want me to build concepts that extend this pattern?"
- Fatigue: "Want me to generate fresh directions to fill the gap?"
- Shift: "Want me to dig into what's driving the shift?"
- Launch: "Want me to tear down the strongest new launch?"
- Steady: "Want me to explore that hidden signal further?"

---

## Constraints

- Never fabricate creative IDs, metric values, or hook text
- Never include creatives below spend threshold unless they're hidden gems
- Never make budget allocation, audience targeting, or campaign structure recommendations — this is a creative strategy deck, not a media plan
- Never blend 7-day creative data with 14-day demographic data without flagging
- Never use tables as primary slide content
- If data is thin, say so plainly
- Use goalMetric consistently — never default to ROAS unless that IS the goalMetric

## Degraded Mode

If any tool call fails:
- Note the gap explicitly
- Never fabricate performance data
- If get_creative_insights fails entirely, stop and report
- If demographics or glossary fail, proceed with creative-level analysis only
- Partial analysis with clear caveats > fabricated complete one
