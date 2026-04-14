---
name: industry-trends
description: Analyze competitor ad libraries — surface creative trends, formats, hooks, and messaging strategies across the industry.
argument-hint: "[--brand domain.com] [--days 30]"
allowed-tools:
  - Read
  - "mcp__motion__get_auth_context"
  - "mcp__motion__get_workspace_competitors"
  - "mcp__motion__get_inspo_creatives"
  - "mcp__motion__get_creative_transcript"
  - "mcp__motion__get_inspo_brand_context"
  - "mcp__motion__search_brands"
  - "mcp__motion__get_brand_by_domain"
  - "mcp__motion__get_workspace_brand"
model: opus
---

# Industry Trends Analysis

Analyze competitor ad libraries to surface creative trends — what formats, hooks, and messaging strategies are being used across the industry. Uses the creative-strategist skill's competitive intelligence framework.

**Critical caveat to carry through the entire analysis:** Competitor creatives have no performance data. You're reading bets, not results. Frame everything as "they're testing this" not "this is working for them."

---

## Phase 1: Setup

### 1a. Parse Arguments

- `--brand`: Optional. A domain (e.g., `nike.com`) or brand name to focus on a single competitor. If omitted, analyze all tracked competitors.
- `--days`: Controls the launch date filter. Default: `30`. Maps to: 7 → `LAST_7_DAYS`, 14 → `LAST_14_DAYS`, 30 → `LAST_30_DAYS`, 90 → `LAST_90_DAYS`.

### 1b. Load Settings & Auth

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for org-specific configuration. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `competitors`: pull from `get_workspace_competitors`
   - Brand guidelines: pull from `get_workspace_brand`
2. Call `get_auth_context()` to resolve workspaceId.

**Settings influence:**
- If no `--brand` specified and settings has a `competitors` list, use those as fallback if workspace competitors are empty.
- Use brand guidelines (voice, positioning, do's/don'ts) to sharpen the "Compare Against Own Approach" analysis.
- Use target audience from settings to evaluate which competitor approaches are relevant to your segments.

### 1c. Load Methodology

Read the creative-strategist skill for the competitive intelligence framework.

---

## Phase 2: Data Collection

### 2a. Get Competitors

**If `--brand` is provided:**
- Try `get_brand_by_domain(brandUrl)` first
- If that fails, try `search_brands(query, workspaceId)` with the brand name
- If no match, tell the user the brand wasn't found and suggest checking the domain

**If no `--brand`:**
- Call `get_workspace_competitors(workspaceId)` to get all tracked competitors
- If the list is empty, tell the user they haven't set up competitor tracking in Motion

### 2b. Pull Competitor Data (Parallel)

For each competitor, dispatch in parallel:

1. `get_inspo_creatives(brandId, workspaceId, limit=500, sort="NEWEST", launchDate=mapped_preset, status="ACTIVE")` — recent active ads. **Always set limit=500.** The API defaults to ~20 if omitted, which produces incomplete and misleading analysis.
2. `get_inspo_brand_context(brandId, workspaceId)` — brand positioning, voice, strategy

### 2c. Pull Transcripts

For video creatives from each competitor, pull transcripts for the **3-5 most recent** per competitor using `get_creative_transcript(creativeEntityId, creativeOrigin)`. Budget transcript calls — they're expensive. Prioritize:
1. Most recently launched
2. Video format (transcripts only work for video)
3. Variety of creative approaches (don't pull 5 similar-looking ads)

### 2d. Get Own Brand Context

Call `get_workspace_brand(workspaceId)` for comparison context — your positioning, voice, strategy.

---

## Phase 3: Analysis

### 3a. Format Trends

Across all competitor creatives:
- What percentage is video vs. static vs. carousel?
- For videos: what's the duration distribution?
- What production styles dominate (UGC, studio, hybrid)?
- Are there emerging formats that multiple competitors are converging on?

### 3b. Hook & Messaging Patterns

From transcripts:
- Classify first-3-second hooks: question, problem-agitation, testimonial, statistic, pattern-interrupt, bold claim, confession, warning
- What messaging themes appear across competitors? (e.g., convenience, trust, transformation, urgency)
- What CTAs are being used?
- What emotional triggers dominate?

Quote actual hooks and headlines from the data — don't generalize.

### 3c. Brand Context Analysis

From inspo brand context:
- Where are competitors positioning themselves?
- What tone/voice are they using?
- Where are gaps between their stated positioning and actual creative execution?
- What audiences are they targeting?

### 3d. Activity & Freshness

- Which competitors are most active (most new launches)?
- Which competitors have the highest active-to-inactive ratio (more conviction in current creative)?
- Are there competitors who have slowed down or stopped launching?

### 3e. Compare Against Own Approach

Using workspace brand data:
- What are competitors doing that your brand isn't?
- What is your brand doing that nobody else is? (potential differentiator)
- Where are there crowded angles that everyone is running? (opportunity to differentiate)
- Where are there white spaces nobody is filling?

---

## Phase 4: Output

Open by reflecting what data was gathered: how many competitors, how many creatives analyzed, what time window, whether transcripts were available.

### Report Structure

**If You Read Nothing Else**
3-5 trend bullets. What the industry is converging on, what's shifting, what nobody's doing.

**Industry Creative Landscape**
Format mix across competitors. Activity levels. Production style trends. The macro view.

**Hook & Messaging Patterns**
Specific patterns from transcripts. Quote actual hooks. Categorize by type. Note which patterns appear across multiple competitors (convergence) vs. unique approaches.

**Competitor Deep Dives**
Per-competitor summary for the most notable competitors:
- Brand context: their positioning, audience, voice
- Creative approach: what they're running, how it connects to their strategy
- Notable creatives: 2-3 specific ads worth paying attention to (with descriptions)

Skip this section if `--brand` is specified — the whole analysis is already a deep dive.

**Gaps & Opportunities**
What nobody's doing. What your data says works but competitors haven't caught onto. Angles that are crowded (avoid) vs. underexplored (test). Frame as specific creative opportunities, not abstract observations.

**Implications for Your Creative Strategy**
How these trends should (or shouldn't) influence your next creative bets. Be specific: "Test a [format] with a [hook type] targeting [audience]" not "Consider diversifying your creative mix."

---

## Error Handling

- **No competitors tracked:** Tell the user to set up competitor tracking in Motion, or use `--brand` to analyze a specific brand
- **Brand not found:** Suggest checking the domain or trying a different search term
- **Inspo brand context unavailable:** Note it and proceed — analyze creatives without strategic context
- **No video creatives (no transcripts):** Analyze visual formats and positioning only — note reduced messaging insight
- **Single competitor with few ads:** Adjust scope — don't over-interpret a thin sample. Suggest expanding the time window with `--days 90`
