---
name: morning-briefing
description: Daily creative delta briefing — what changed overnight in performance and competitor activity. Designed for team standup. Shows big things the team should know about.
allowed-tools:
  - Read
  - Agent
  - "mcp__motion__get_auth_context"
  - "mcp__motion__get_creative_insights"
  - "mcp__motion__get_demographic_breakdown"
  - "mcp__motion__get_workspace_competitors"
  - "mcp__motion__get_inspo_creatives"
model: opus
---

# Morning Briefing

Daily creative delta briefing — what changed day-over-day that the team should know about. This is a standup tool, not a deep analysis. Compact, specific, action-oriented.

**Core principle:** The briefing is the delta, not the state. "Hook rate dropped 15%" is useful. "Hook rate is 3.2%" is a fact sheet. Every bullet should answer "what's different from yesterday?"

---

## Phase 1: Get Context

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for org-specific configuration. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `morning_briefing_lookback`: LAST_7_DAYS
   - `competitors`: pull from `get_workspace_competitors`
   - `primary_metrics` / `exclude_metrics`: auto-detect
2. Call `get_auth_context()` for workspaceId.
3. Read the creative-strategist skill for methodology.

**Settings influence:**
- Use `primary_kpi` to determine which metric delta leads the "Big Things" section. If primary KPI is CPA, a CPA swing matters more than a ROAS swing.
- Use `morning_briefing_lookback` as the comparison window if set (default: LAST_7_DAYS).
- Use `competitors` list from settings if workspace competitors aren't configured in Motion.
- If settings contain `primary_metrics`, ensure those metrics lead the delta analysis. If `exclude_metrics`, omit those from the report.

---

## Phase 2: Pull Performance Data (Parallel)

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

**Spend signals (first):**
1. `get_creative_insights(workspaceId, "SPEND", datePreset="YESTERDAY", limit=10, withAggregatedInsights=true)`

→ Extract `goalMetric` and `spendThreshold` from the response. Use goalMetric for all efficiency-sorted calls.

2. `get_creative_insights(workspaceId, "SPEND", datePreset="LAST_7_DAYS", limit=10)`

**Scaling signals:**
3. `get_creative_insights(workspaceId, "SCALING", datePreset="YESTERDAY", limit=10)`
4. `get_creative_insights(workspaceId, "SCALING", datePreset="LAST_7_DAYS", limit=10)`

**Efficiency signals:**
5. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset="YESTERDAY", limit=10)` — efficiency leaders by workspace goal metric
6. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset="LAST_7_DAYS", limit=10)`

**Hook signals:**
7. `get_creative_insights(workspaceId, "HOOK", datePreset="YESTERDAY", limit=10)`

**Note:** If YESTERDAY returns empty or very limited data (common early in the day), fall back to comparing LAST_7_DAYS against LAST_30_DAYS. Note the adjusted window in the briefing header.

---

## Phase 3: Pull Competitor Activity

1. `get_workspace_competitors(workspaceId)` — get tracked competitors
2. For each competitor (parallel): `get_inspo_creatives(brandId, workspaceId, limit=5, sort="NEWEST", launchDate="LAST_7_DAYS")`

If competitor tracking isn't set up, skip this section and note it.

---

## Phase 4: Compute Deltas

Compare yesterday vs. 7-day data to identify what changed:

### Performance Deltas
- **Newly scaling:** Creatives appearing in yesterday's scaling list but not in the 7-day top — they're just starting to gain traction
- **Declining:** Creatives in the 7-day top performers but absent from or dropping in yesterday's data — they're losing steam
- **Spend shifts:** Where budget moved — which creatives gained share, which lost it
- **Hook changes:** Significant thumbstop rate changes — new creatives hooking well, or established ones losing grip

### Competitor Deltas
- **New launches:** Competitor creatives launched in the last 7 days
- **Volume signals:** Competitors launching more/fewer creatives than usual

### What Counts as "Big"
Not everything that changed is worth reporting. Focus on:
- Creatives that shifted from scaling to declining (or vice versa) — a reversal
- Top-spending creatives with notable goalMetric movement — budget at risk
- New creatives that immediately started scaling — a possible winner
- Competitor launches that represent a new approach (not just more of the same)
- Any creative with significant hook rate change — the first signal of fatigue or resonance

If nothing significant changed, say so: "Stable day — no major movements. Current scaling winners are holding." A quiet briefing is better than manufactured urgency.

---

## Phase 5: Output

### Format

Keep it tight. This is for a standup, not a strategy session.

```
# Creative Morning Briefing — [Date]
**Workspace:** [name] | **Window:** Yesterday vs. 7-day trend

## Big Things That Changed
- [Bullet 1 — the single most important delta]
- [Bullet 2 — second most important]
- [Bullet 3 — if warranted, otherwise skip]

## Performance Deltas

| Creative | Metric | Yesterday | 7-Day | Direction |
|----------|--------|-----------|-------|-----------|
| [name]   | goalMetric | [val]     | [val] | ↑ / ↓ / → |

## Scaling Moves
[1-2 sentences: what's gaining budget, what's losing it, and why it matters]

## Competitor Watch
[New launches from tracked competitors. Format, hook approach, what's notable. Skip if nothing new.]

## Suggested Focus for Today
- [Action 1 — specific, grounded in the data above]
- [Action 2 — if warranted]
```

### Rules

- **"Big Things That Changed" is mandatory.** Even if the answer is "nothing." This is the section people read.
- **3 bullets max** in Big Things. If there are more than 3 significant changes, pick the top 3 and move the rest to the sections below.
- **Performance Deltas table** is optional — include it when there are specific numbers worth comparing. Skip if the story is better told in prose.
- **Competitor Watch** is optional — skip if no new launches.
- **Suggested Focus** should be 1-2 actions, not a to-do list. Each action should connect to a specific finding.
- **Total length:** Aim for something that can be read in under 2 minutes.
- If someone wants depth, they run `/performance-analysis`.

---

## Error Handling

- **YESTERDAY returns empty:** Fall back to LAST_7_DAYS vs. LAST_30_DAYS comparison. Note the adjusted window.
- **Competitor tools fail:** Skip Competitor Watch section, note the gap.
- **All queries fail:** Report: "Motion data unavailable this morning. Check your workspace connection."
- **Very few creatives in results:** Note limited data, don't over-interpret. "Low volume day — directional signals only."
