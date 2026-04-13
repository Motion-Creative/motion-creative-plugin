---
name: competitor-watch
description: Weekly competitive intelligence scan — tracks competitor ad strategies, messaging shifts, and creative patterns. Compares against last week's baseline to surface what changed. Use for "competitor scan", "what are competitors doing", "competitive intel", "competitor watch", or "what changed in the market".
argument-hint: "[--competitor domain.com] [--baseline-only]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Agent
  - AskUserQuestion
  - ToolSearch
model: opus
---

# Competitor Watch — Weekly Competitive Intelligence

A self-contained weekly competitive scan that any Motion customer can run. Tracks competitor ad strategies, messaging shifts, creative patterns, and new launches — then compares against last week's baseline to surface what actually changed.

**No external context required.** This skill uses only the Motion MCP and stores its own baselines. No repo, no strategy docs, no internal knowledge base needed.

**Core principle:** The value is in the delta, not the state. "Foreplay launched 6 new Lens-focused ads this week" is intelligence. "Foreplay has 41 active ads" is a fact sheet. Every finding answers: **"What should I pay attention to this week?"**

**Baseline model:** First run establishes a baseline. Every subsequent run compares current state to the previous week's baseline, reports what changed, and updates the baseline. Baselines are stored locally as markdown files.

---

## Phase 0: Discover Motion MCP Connector

The Motion MCP connector has a different tool prefix for each user. It follows the pattern `mcp__<uuid>__<tool_name>` where the UUID varies by installation (e.g., `mcp__7fa9f3c0-457b-4a74-a671-2a3f0c40b6fe__get_auth_context`).

**Before doing anything else, discover the connector:**

1. Use ToolSearch to find the Motion MCP tools: search for `get_auth_context` — this is a Motion-specific tool name
2. From the result, extract the full tool prefix (everything before `get_auth_context`). This is your connector prefix for this user's session.
3. Store the prefix and use it for ALL subsequent Motion MCP calls in this run.

**The Motion MCP tools you need (append each to the discovered prefix):**
- `get_auth_context` — resolve workspace
- `get_workspace_competitors` — list tracked competitors
- `get_inspo_creatives` — pull ad library data
- `get_creative_transcript` — get video ad transcripts
- `get_inspo_brand_context` — brand positioning data
- `search_brands` — find brands by name
- `get_brand_by_domain` — find brands by domain
- `get_workspace_brand` — get own brand info

**If no Motion MCP tools are found:** Stop and tell the user: "I couldn't find a Motion MCP connection. Connect your Motion workspace at https://motionapp.com and make sure the MCP connector is enabled, then try again."

---

## Phase 1: Setup

### 1a. Parse Arguments

- `--competitor`: Optional. A domain (e.g., `foreplay.co`) to scan a single competitor. If omitted, scan all saved competitors.
- `--baseline-only`: Establish baselines without producing a delta report. Use for first run.

### 1b. Resolve Workspace

Using the discovered connector prefix, call `get_auth_context()`.

If multiple workspaces exist, ask the user which one to scan. Store the workspaceId for all subsequent calls.

### 1c. Load or Create Competitor Watchlist

Check for a saved watchlist at `~/.claude/competitor-watch/watchlist.md`.

**If watchlist exists:** Read it. It contains brand names, domains, and resolved brandIds from a previous session. Use these directly — no need to ask again.

**If no watchlist exists (first run):** Ask the user to set up their competitor watchlist:

> "This is your first competitor watch. Who are the 3-5 brands you want to track? Give me their names or website domains and I'll set everything up."

Use AskUserQuestion to collect this. Accept brand names, domains, or both.

Then resolve each brand:
1. Try `get_brand_by_domain(brandUrl)` if a domain was provided
2. If that fails or a name was given, try `search_brands(query)` with the brand name
3. If multiple results, show the matches and ask which one
4. If no match, tell the user the brand wasn't found in Motion's ad library and skip it

Once all brands are resolved, save the watchlist to `~/.claude/competitor-watch/watchlist.md`:

```markdown
---
created: [YYYY-MM-DD]
updated: [YYYY-MM-DD]
workspace_id: [workspaceId]
---

# Competitor Watchlist

| Brand | Domain | Brand ID | Added |
|-------|--------|----------|-------|
| [Name] | [domain] | [brandId] | [YYYY-MM-DD] |
| [Name] | [domain] | [brandId] | [YYYY-MM-DD] |
```

**Updating the watchlist:** If the user says "add [brand]" or "remove [brand]" or "change my competitors", update the watchlist file accordingly. The watchlist is the persistent config — it survives across sessions.

### 1d. Get Own Brand Context

```
get_workspace_brand(workspaceId)
```

Used for comparison framing — "how does this compare to our own ad strategy?"

### 1e. Load Previous Baselines

Look for baseline files at `~/.claude/competitor-watch/{competitor-slug}.md` for each brand on the watchlist. If no baseline exists for a brand, this is a first run for that brand — establish the baseline, don't compute deltas.

---

## Phase 2: Ad Intelligence — Full Active Portfolio + Recently Killed

For each competitor, dispatch in parallel:

### 2a. Pull Ad Data (Queries per Competitor)

**No limits.** Pull everything — don't sample. Paginate if responses hit the API ceiling.

**Query 1: Full Active Portfolio (ALL active ads, no limit)**
```
get_inspo_creatives(brandId, workspaceId, sort="NEWEST", status="ACTIVE")
```
All currently running ads regardless of launch date. This IS their strategy right now. Do NOT set a limit — get every active ad. If the response is capped by the API, paginate with additional calls (e.g., sort="OLDEST" to get the other end) until you have the full set.

**Query 2: Recently Killed (inactive, paused in last 7 days, no limit)**
```
get_inspo_creatives(brandId, workspaceId, sort="NEWEST", launchDate="LAST_30_DAYS", status="INACTIVE")
```
Pull all inactive ads from the last 30 days, then filter client-side to only those with `pauseDate` within the last 7 days. These are the ads that just got turned off — the freshest kill signals. Ads killed more than 7 days ago are in the baseline already; this week's kills are the delta.

**Query 3: Brand Context**
```
get_inspo_brand_context(brandId, workspaceId)
```
Positioning, voice, strategy signals.

**Pagination strategy:** The API may cap results (typically 50 per call). If you hit the cap:
1. First call: `sort="NEWEST"` — gets the most recent ads
2. Second call: `sort="OLDEST"` — gets the oldest active ads
3. Merge and deduplicate by creative ID
4. Repeat until you have the full set or diminishing returns (same ads appearing)

### 2b. Full Active Portfolio Analysis

Analyze the complete active portfolio as a body of work — this reveals the competitor's live-in-market strategy:

**Format & Production:**
- Video / static / carousel split
- Production styles (UGC, screen recording, studio, talking head, motion graphics)
- Production quality spectrum

**Messaging Themes:**
- What hooks appear most frequently? Group by type: problem-agitation, testimonial, question, bold claim, social proof, demo walkthrough
- What value propositions are being pushed? Rank by frequency (= conviction level)
- What emotional registers dominate? (fear, aspiration, frustration, curiosity, urgency)
- Quote actual hooks verbatim — don't summarize

**Feature & Product Emphasis:**
- What product features are mentioned across active ads? Rank by frequency.
- Any features being promoted that your brand doesn't have? (competitive gap signal)
- Any features you have that they're NOT promoting? (potential differentiation)

**Landing Page Strategy:**
- Where are ads driving traffic? Group by destination.
- Which products/features get the most ad-driven traffic?

### 2c. Survival Analysis — Weekly Cohorts

Cohort every ad by age and compute survival:

**Long-runners (60+ days, still ACTIVE)**
These are evergreen winners. They've survived multiple creative refresh cycles. Pay attention to what makes them durable.

**Survivors (launched 15-60 days ago, still ACTIVE)**
Proven creative. If it's been running 2+ weeks, they've seen enough data to know it works. This is the **strongest signal** of what's actually performing.
- What do survivors have in common? (format, hook, messaging)
- Survival rate for this cohort

**Testing Zone (launched 7-14 days ago, still ACTIVE)**
Past the first cut but could still be in evaluation. **Moderate signal.**
- Which are iterations on survivors? Which are new directions?
- Any killed from this cohort? (fast kills = strong negative signal)

**Fresh Tests (launched < 7 days ago)**
Newest bets. Reveal **strategic intent** — what they're thinking about right now.
- New messaging themes being tested?
- Events, promos, product launches?
- Iterations on existing winners or entirely new directions?

**Important: "Inactive" does not mean "failed."** An ad can stop running because:
- It underperformed (the obvious read, but not the only one)
- It was time-bound — a promo, event, or seasonal campaign with a planned end date
- Budget shifted to a hotter creative
- It was replaced by a new iteration
- It completed its audience/impression goal

Always reason over the *likely* reason. Look at content clues (mentions a date/event/sale?), runtime (2+ weeks before killing = probably adequate), and siblings (newer ad with similar messaging = iteration replacement). Quick kills (< 5 days, no promo content) are more likely genuine underperformance.

### 2d. Deep-Dive Prioritization

You can't pull transcripts for every ad. Use this priority:

**Always deep-dive (transcript + full analysis) — 5-8 per competitor:**
- Long-running survivors (15+ days active) — what messaging is actually working?
- Ads representing NEW messaging directions not seen in last week's baseline
- Ads mentioning or positioning against your brand or other competitors

**Summary only (read copy + metadata):**
- 7-14 day survivors, ads in clusters where another ad already got a deep-dive

**Metadata only (format, CTA, landing page, launch date):**
- Fresh tests (< 7 days), clear duplicates, minor copy variations

For deep-dives, dispatch Agent sub-agents in parallel:
```
For each ad: pull get_creative_transcript, extract opening hook (verbatim),
core claim, proof structure, emotional register, CTA, competitive positioning,
feature depth. Return a structured brief card.
```

### 2e. Strategic Clustering

Group ads into clusters that reveal the creative strategy:

- **Test clusters:** Ads with same messaging angle but different hooks, formats, or creators. Name the bet: "They're testing whether UGC outperforms screen recordings for their analytics feature."
- **Promotional intel:** Events, free trials, discounts, webinars, partnerships, product launches?
- **Sub-product detection:** If a competitor's ads drive to multiple domains or products (e.g., superside.com + superads.ai), analyze each product's messaging separately. A company running two businesses through one ad account is a different signal than a single-product company.

### 2f. Competitive Evaluation

For the full portfolio, answer: **If a prospect evaluating your brand saw these ads, how would it make them think about you?**

- What claims are they making that you can't match today?
- What claims are they making that you do better but aren't advertising?
- What audience overlap exists?
- What social proof might sway a prospect?
- Is their creative quality higher, lower, or comparable to yours?
- What would your sales team need to know to handle this in a competitive deal?

---

## Phase 3: Own-Brand Self-Benchmark

Run the same analysis on your own brand's ads. This creates the comparison frame.

### 3a. Pull Own-Brand Data

Use the brandId from `get_workspace_brand`. Pull the same queries as competitors:
- ALL active ads (no limit, paginate with NEWEST + OLDEST if needed)
- Inactive ads from last 30 days, filtered to pauseDate within last 7 days

### 3b. Analyze the Same Way

Apply the same framework: format distribution, messaging themes, survival analysis, test clusters.

### 3c. Comparison Output

For every competitor finding, provide the own-brand comparison:
- "They run 73% video vs your 45%"
- "Their oldest survivor is 27 days vs your 12"
- "They lead with AI agent messaging; you lead with analytics depth"

Don't just compare numbers — interpret what the gap means. A format gap might be intentional or a blind spot worth testing.

---

## Phase 4: Delta Computation (vs Last Week's Baseline)

If a baseline exists from the previous scan, compute what changed:

**Volume & Velocity:**
- Total active ads now vs last week (net change)
- New launches since last scan
- Ads that went inactive since last scan
- Weekly launch cadence trend

**Strategic Shifts:**
- New messaging territories they weren't running last week
- Format mix changes
- Features promoted that weren't in last week's baseline
- Tone or positioning shifts in brand context

**What They Stopped Doing:**
- Messaging angles from last week's baseline that are gone
- Formats abandoned
- Claims stopped

**What Survived:**
- Ads that were active last week AND still active this week = confirmed winners
- Anything that was in the "Testing Zone" last week and survived to "Proven" this week

**Signal Strength:**

| Signal | Strength | Why |
|--------|----------|-----|
| A survivor from 15+ days with a new messaging angle | Very High | Proven message they're scaling |
| 3+ fresh tests all exploring the same new theme | High | Coordinated strategic bet |
| A feature promotion for something you don't have | High | Competitive gap |
| Format mix shift (e.g., UGC ratio doubled) | Medium | Production strategy change |
| Volume spike or drop vs last week | Medium | Resource allocation signal |
| Single ad with an unusual approach | Low-Medium | Could be a one-off test |
| Minor copy variations on existing themes | Low | Skip unless pattern emerges |

---

## Phase 5: Output

### Report Structure

```markdown
# Competitor Watch — [Date]
**Workspace:** [name] | **Competitors:** [N scanned] | **Baseline:** [first run / delta from YYYY-MM-DD]

---

## What Changed This Week
[3-5 bullets MAX. Only findings that matter. Each: what shifted + why it matters + what to consider.
If nothing significant: "Stable week — no major competitive shifts detected."]

---

## Competitor Intel

### [Competitor Name] — [one-line: quiet / active / shifting / launching]

**Active Portfolio:** [N] total | Video [X%] / Static [Y%]
**30-Day Launches:** [N] launched | [N] survived | [N] killed | [X%] survival rate
**Weekly Cadence:** [accelerating / steady / slowing / paused]

**Live Strategy:**
- [Top messaging themes, ranked by frequency]
- [Dominant hooks and emotional registers — quote the actual hooks]
- [Format & production strategy]
- [Features being promoted]

**Survivors (15+ days, still running):**
- [What they have in common — the pattern that's working]

**Fresh Tests (this week):**
- [New launches — what they're testing, what it signals]

**Test Clusters:**
- [Cluster 1: name the bet, N ads, what they're varying]

**Competitive Eval:**
- [If a prospect saw these ads, how would it affect your deals?]

**Delta from Last Week:**
- [What's new, what's gone, what shifted. Skip if first run.]

[Repeat per competitor. Skip sections with no content.]

---

## Your Brand Benchmark

**Your Active Portfolio:** [N] total | Video [X%] / Static [Y%]
**30-Day Survival Rate:** [X%]

**How You Compare:**

| Dimension | You | [Comp 1] | [Comp 2] | [Comp 3] | Read |
|-----------|-----|----------|----------|----------|------|
| Active ads | [N] | [N] | [N] | [N] | [who's investing more?] |
| Oldest active ad | [days] | [days] | [days] | [days] | [portfolio maturity] |
| Video/static mix | [%] | [%] | [%] | [%] | [format strategy] |
| Visual format diversity | [N types] | [N] | [N] | [N] | [creative range] |
| Customer proof in ads | [Y/N] | [Y/N] | [Y/N] | [Y/N] | [credibility gap?] |
| Biggest bet | [what] | [what] | [what] | [what] | [strategic direction] |
| Key feature promoted | [what] | [what] | [what] | [what] | [positioning overlap] |
| Comment-gate CTAs | [Y/N] | [Y/N] | [Y/N] | [Y/N] | [engagement tactic] |

**Gaps worth testing:**
- [Things competitors do that you don't — worth a test?]
- [Things you do that nobody else does — double down?]

---

## Recommended Actions
[2-3 specific, grounded-in-findings recommendations.
Each connects to a specific finding above.
Not "diversify your creative mix" — instead: "Test a UGC creator ad using
the solopreneur burnout angle that's been running 27 days for [competitor]."]
```

### Output Rules

- **Lead with "What Changed."** If someone reads only the first section, they get the important stuff.
- **Delta-first on repeat runs.** Don't report state — report change. "No changes" = skip or one line.
- **Quote actual hooks.** Don't summarize when you can quote.
- **Carry the competitor data caveat.** You're reading bets, not results. Frame as "they're testing this" not "this is working for them." The only proven patterns come from your own performance data.
- **Recommendations must be specific.** "Test a confessional UGC ad where a creator admits a frustration with analytics tools before showing your product" not "consider UGC."
- **Length:** 5-minute read max. If the market is quiet, make it 2 minutes.

---

## Phase 6: Update Baseline

After producing the report, write the new baseline for each scanned competitor.

### Baseline File Format

Write to `~/.claude/competitor-watch/{competitor-slug}.md`:

```markdown
---
competitor: [Name]
brand_id: [brandId]
last_scanned: [YYYY-MM-DD]
---

# [Competitor Name] — Weekly Baseline

## Portfolio Snapshot
**Active Ad Count:** [N]
**Format Mix:** [video X% / static Y%]
**Production Styles:** [UGC, screen recording, etc.]

## Active Ad Inventory
[For each active ad: ID, format, hook (verbatim), CTA, landing page, launch date, days active]

## Messaging Themes
- [theme 1 — frequency, example hook]
- [theme 2 — frequency, example hook]

## Survival Cohorts
**60+ day long-runners:** [list with IDs]
**15-60 day survivors:** [list with IDs]
**7-14 day zone:** [list]
**< 7 day fresh tests:** [list]

## Test Clusters
- [Cluster name: ad IDs, what they're varying]

## Features Promoted
- [ranked list]

## Brand Context Summary
[positioning, voice, audience from get_inspo_brand_context if available]
```

### Baseline Rules

- Overwrite the previous baseline on each scan — this is a weekly snapshot, not an append log
- Store the full active ad inventory (IDs + hooks) so next week's scan can diff: "these 3 ads are new, these 2 are gone"
- If a section couldn't be populated (e.g., brand context unavailable), note the gap
- Baseline files live at `~/.claude/competitor-watch/` — outside any repo, purely local

---

## Error Handling

- **No watchlist and no `--competitor` flag:** Trigger first-run onboarding — ask for 3-5 brands to track
- **Brand not found:** Suggest checking the domain or trying a different search term
- **Brand context unavailable:** Note it, proceed with ad data only
- **Few/no ads returned:** Note thin data, don't over-interpret. Suggest expanding to LAST_90_DAYS
- **No previous baseline:** Treat as first run, produce a full landscape report, label as "Baseline Established"
- **Transcript quality poor:** Some transcripts return garbage (single words, unrelated text). If a transcript doesn't make sense, skip it and note the gap. Don't quote bad transcripts.

---

## Connected Workflows

After the scan:
- **Deep dive on one competitor's ads:** `/industry-trends --brand [domain]`
- **Generate concepts that counter competitor moves:** `/create-concepts` with competitive context
- **Analyze your own performance in detail:** `/performance-analysis`
- **Build a brief inspired by a competitor's approach:** `/build-brief`
