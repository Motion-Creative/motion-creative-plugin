---
name: customize
description: >
  Set up motion-creative for your brand — configure KPIs, brand guidelines, competitors,
  production constraints, and brief preferences. Run this before using other commands
  for the best results. Use when: customize creative plugin, set up creative analysis,
  configure brand, update brand guidelines, change KPI, update competitors.
allowed-tools:
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - "mcp__ruimotion__get_auth_context"
  - "mcp__ruimotion__get_workspace_brand"
  - "mcp__ruimotion__get_workspace_competitors"
  - "mcp__ruimotion__get_glossary_values"
model: opus
---

# Customize Motion Creative

Set up this plugin for your brand so every analysis, concept, and brief reflects how your team actually works.

## How This Works

This command configures your `motion-creative.config.md` file — the source of truth that all other commands read from. This file is yours: plugin updates never touch it. You can re-run `/customize` anytime to update settings as your strategy evolves.

## Step 1: Detect Current State

Try to read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md`.

- **If the file exists:** This is a re-configuration. Read it, show the user their current settings, and ask what they want to update. Only touch those sections.
- **If the file does not exist:** This is a first-time setup. Walk through all sections.

## Step 2: Gather Context from Motion

Pull real data from the Motion workspace to pre-fill as much as possible:

```
Run in parallel:
1. get_auth_context → workspace ID and name
2. get_workspace_brand → brand positioning, voice
3. get_workspace_competitors → competitor list
4. get_glossary_values → available taxonomy categories
```

Use these results to pre-fill settings rather than asking the user for information Motion already has.

## Step 3: Walk Through Configuration Sections

Work through each section, using Motion data where available and asking only when needed.

### 3a. Workspace & KPIs

If `get_auth_context` returned workspace info, apply it directly. Then ask:

**Primary KPI:** "What's the one metric your team optimizes for above all else?"
- Options: ROAS, CPA, CPL, CPC, Purchases, Purchase Value
- Default suggestion based on workspace type if available

**Secondary KPIs:** "What 2-3 metrics do you watch alongside [primary KPI]?"
- Options from: HOOK, CPA, ROAS, SPEND, SCALING, CTR_ALL, CPC, PURCHASES

### 3b. Competitors

If `get_workspace_competitors` returned results, show them: "Motion already tracks these competitors: [list]. Want to keep this list, or add/remove any?"

If no competitors configured: "Which competitor brands should we track? Use domains (e.g., competitor.com)"

### 3c. Brand Guidelines

If `get_workspace_brand` returned brand info, use it to pre-fill. Then ask the user to review and refine:

**Brand Voice:** "How should your ad copy sound? (e.g., casual and witty, authoritative, empathetic)"

**Brand Positioning:** "In one sentence, what makes your brand different?"

**Creative Do's and Don'ts:** "Any rules your creative team follows? Things you always or never do in ads?"

**Target Audience:** "Who's your primary audience? What tension does your product resolve for them?"

### 3d. Production Constraints

**Team Size:** "How big is your creative team?"
- Options: Small (1-3), Medium (4-8), Large (9+)

**Production Capabilities:** "What formats can your team produce?"
- Options (multi-select): UGC, Static images, Motion graphics, Full video production, Animation, Photography

**Max Difficulty:** "What's the most complex production your team should take on?"
- Options: Low (static, simple UGC), Medium (edited UGC, motion graphics), High (full production, multi-scene)

### 3e. Brief Preferences

"Do you have a custom brief template your team uses, or should we use the default?"
- If custom: Ask the user to describe their format or paste it. Save to `references/custom-brief-template.md`
- If default: Keep the built-in template

### 3f. Reporting Defaults

"What time range do you typically analyze?" (default: LAST_30_DAYS)

"How many creatives per metric is useful?" (default: 10)

### 3g. Metric Preferences

"Are there specific metrics you always want to see first, or any you want excluded from analysis?"
- **Primary metrics**: Always show these first (e.g., ROAS, CPA)
- **Secondary metrics**: Show when going deeper (e.g., hook rate, hold rate, CTR_ALL)
- **Exclude metrics**: Never show these (e.g., irrelevant conversion types)

If the user doesn't have preferences, leave these empty — skills will auto-detect based on creative format and campaign objective.

### 3h. Glossary Categories

If `get_glossary_values` returned categories, show them: "These taxonomy categories are available: [list]. Which ones matter most for pattern analysis? (or keep all)"

### 3i. Slack Channel (if Slack MCP available)

"Where should morning briefings be posted?" Search Slack for creative-related channels to suggest options.

## Step 4: Apply Settings

Write all gathered values to `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` using this structure:

```markdown
---
# motion-creative configuration
# Created by /customize. Plugin updates will not touch this file.

# ── Motion Workspace ──
workspace_id: "the-workspace-id"
workspace_name: "The Workspace Name"

# ── Primary KPI ──
primary_kpi: "ROAS"

# ── Secondary KPIs ──
secondary_kpis:
  - "hook_rate"  # Uses thumbstop_ratio from API
  - "CPA"

# ── Competitors ──
competitors:
  - "competitor1.com"
  - "competitor2.com"

# ── Reporting Defaults ──
default_date_preset: "LAST_30_DAYS"
default_creative_limit: 10
morning_briefing_lookback: "LAST_7_DAYS"

# ── Demographic Focus ──
demographic_focus: "both"
target_demographics:
  - "25-34"
  - "35-44"

# ── Production Constraints ──
team_size: "small"
production_capabilities:
  - "UGC"
  - "static"
  - "motion graphics"
max_production_difficulty: "Medium"

# ── Brief Preferences ──
brief_format: "default"

# ── Metric Preferences ──
primary_metrics: []
secondary_metrics: []
exclude_metrics: []

# ── Glossary Priority Categories ──
priority_glossary_categories: []

# ── Notification Channel ──
slack_channel: ""
---

# Brand Guidelines

## Brand Voice
[Populated from user answers and get_workspace_brand]

## Brand Positioning
[Populated from user answers and get_workspace_brand]

## Creative Do's
[Populated from user answers]

## Creative Don'ts
[Populated from user answers]

## Target Audience
[Populated from user answers]

## Current Campaign Themes
[Populated from user answers]
```

Replace all bracketed placeholders with real values from the user's answers and Motion API data. Every field should have an actual value — no placeholders, no empty descriptions.

## Step 5: Summary

Present what was configured:

```markdown
## Your Motion Creative Setup

**Workspace:** [name]
**Primary KPI:** [metric] | **Secondary:** [metrics]
**Competitors:** [list]
**Team:** [size] team with [capabilities]
**Brief format:** [default/custom]

### Brand Summary
- Voice: [summary]
- Positioning: [summary]
- Key rules: [do's/don'ts highlights]

### What's Next
Your settings are saved in `motion-creative.config.md`. Plugin updates won't touch this file.

Run these commands to start getting value:
- `/performance-analysis` — see what's working right now
- `/morning-briefing` — get tomorrow's daily delta
- `/industry-trends` — see what competitors are doing
```

## Important Rules

- **Never assume industry defaults.** If Motion data doesn't provide the answer and the user hasn't said, ask.
- **Skip what's already configured.** On re-runs, only touch what the user asked to change.
- **Use Motion data first.** Don't ask the user for things the workspace already knows.
- **Plain language only.** Never mention "frontmatter", "YAML", or "config file internals" to the user.
- **Don't rename anything.** Only replace values and update content.
