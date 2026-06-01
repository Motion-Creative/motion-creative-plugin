# Motion Creative Plugin — Test Plan

Goal: Verify every skill (1) pulls data that matches what users see in the Motion app, and (2) produces analysis that's genuinely valuable — not surface-level observations anyone could make by glancing at the dashboard.

---

## How to Use This Plan

Each skill has two test tracks:

- **Data Fidelity** — Does the output match what you'd see in Motion's UI for the same workspace, date range, and filters? Run the skill, then cross-reference specific numbers against the app.
- **Insight Quality** — Would a creative strategist who already uses Motion learn something new? Or is the output just restating metrics in sentence form?

For every test, use a real workspace with:
- 20+ active creatives (enough variety to expose ranking/filtering bugs)
- At least 2 tracked competitors with active ads
- Glossary taxonomy with tags applied to creatives
- Mix of video + static formats
- Creatives across multiple campaigns/ad sets

---

## 1. Performance Analysis (`/performance-analysis`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Top performers by SPEND match app | Open Motion > Top Ads > sort by Spend (Last 30 Days). Compare top 10 names + spend values | Same creatives, spend within 5% |
| goalMetric value is correct | Check workspace's configured goal metric in Motion settings | Skill names the right metric (e.g., "ROAS" not "CPA" if workspace uses ROAS) |
| spendThreshold filtering works | Find a creative just below the threshold in Motion. Confirm it's excluded from "top performer" claims | No low-spend creatives presented as reliable performers |
| Scaling detection matches app | Open Motion > Top Ads > sort by Scaling. Compare which creatives show scaling up vs. down | Same scaling direction for top 5 creatives |
| Demographic breakdown matches | Open Motion > Demographics. Compare age/gender split with skill output | Same top-performing segments, percentages within 5% |
| Account-level aggregates correct | Check Motion dashboard totals (total spend, avg CPA/ROAS, etc.) for the date range | Aggregates match within 5% |
| Date range respected | Run with LAST_7_DAYS. Confirm no creative data from outside that window appears | All cited creatives have activity in the window |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| "Why" behind performance | Does it explain WHY top performers work (hook structure, messaging angle, audience fit) — not just that they have good numbers? | "Creative X has the best ROAS at 4.2x" with no explanation |
| Campaign structure insights | Does it identify structural issues (budget starvation, creative competition within ad sets, fatigue patterns)? | Only listing creatives by metric rank with no structural observation |
| Non-obvious pattern detection | Does it find a pattern you didn't already know from the dashboard? (e.g., "Your UGC creatives outperform polished video by 2x on hook rate but underperform on conversion — the hook works but the sell doesn't land") | "Video creatives tend to have higher hook rates" (obvious) |
| Actionable recommendations | Are the 3-5 recommendations specific enough to act on without asking follow-up questions? | "Consider testing new creatives" or "Optimize your top performers" |
| Declining creative diagnosis | Does it diagnose WHY a creative is declining (fatigue, seasonal, audience saturation) vs. just flagging the decline? | "Creative Y spend dropped 30% WoW" with no hypothesis |

---

## 2. Morning Briefing (`/morning-briefing`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Yesterday vs. 7-day delta accuracy | Pull yesterday's top creatives in Motion and compare to 7-day view. Verify the deltas the skill reports | Direction (up/down) correct; magnitude within 10% |
| Scaling moves match | Check which creatives gained/lost budget allocation yesterday vs. prior days | Same creatives flagged as scaling up/down |
| Competitor launches are real | Open Meta Ad Library for the competitor. Confirm the "new launches" actually exist and launched this week | All cited competitor ads exist and are recent |
| No stale data | Run on Monday morning. Confirm it's not showing Friday's data as "yesterday" | Weekend data handled correctly |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Delta-focused, not state-focused | Does it lead with what CHANGED, not what IS? | "Your top creative is X with Y spend" (that's state, not delta) |
| Under 2 min read time | Is it actually concise enough for standup? | 800+ word output with tables and deep dives |
| Suggested focus is specific | "Pause Creative X and test a new hook variant targeting 25-34F" vs. generic | "Keep monitoring performance" |
| Signal-to-noise ratio | Does every bullet matter, or is there filler? | Reporting metrics that didn't meaningfully change |

---

## 3. Weekly Performance (`/weekly-performance`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| WoW spend delta correct | Calculate manually: (this week total spend - last week total spend) / last week total spend | Within 3% of skill's reported delta |
| Week type detection valid | Look at the data yourself. Does the detected week type (Breakout/Fatigue/Shift/Launch/Steady) match reality? | Type is defensible given the data |
| Creative rankings match both weeks | Check Motion for both CUSTOM date ranges. Verify the creatives called "winners" and "losers" are correctly categorized | No creative miscategorized (e.g., called a "winner" when it actually declined) |
| Hooks/headlines are verbatim | Compare quoted hooks against actual creative content in Motion or ad account | Exact match, no paraphrasing or hallucination |
| HTML deck renders correctly | Open the output HTML file in a browser | All slides render, no broken layouts, creative thumbnails display |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Narrative arc, not data dump | Does the deck tell a story connecting the slides, or is each slide independent? | Slide 1: metrics. Slide 2: different metrics. No thread connecting them |
| Behavioral "why" in speaker notes | Do speaker notes explain the human behavior driving the numbers? | Speaker notes just restate the metric: "CTR increased 15%" |
| "What's Next" grounded in evidence | Are the next steps derived from THIS week's data, or generic best practices? | "Test more video formats" with no connection to this week's findings |
| Would you actually present this? | Could you show this deck to your team without editing it? | Needs heavy editing to be presentable |

---

## 4. Analyze Ad (`/analyze-ad`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Entity hierarchy correct | Check Motion: which campaigns/ad sets does this creative run in? | All listed campaigns and ad sets match |
| Metrics vs. account benchmark accurate | Compare creative's metrics and the account averages in Motion | Both the creative's numbers AND the benchmarks match |
| Transcript accuracy (video) | Play the actual video. Compare transcript to what skill outputs | Hook words match verbatim; no fabricated dialogue |
| Demographic overlay matches | Check this creative's demographic performance in Motion (if available) | Same top segments identified |
| Glossary tags correct | Check the creative's tags in Motion's glossary view | Tags match; no invented categories |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| 4-Question Framework diagnosis is specific | Does it map metric patterns to specific Q1-Q4 failures? (e.g., "High hook rate but low conversion = Q3 failure: the creative attracts but doesn't persuade") | Generic: "This creative could be improved" |
| Ready/Iterate/Rethink call is justified | Is the verdict backed by the evidence presented? Could you argue it to a colleague? | "Iterate" with no clear reason why it's not "Ready" or "Rethink" |
| Recommendations are creative-specific | Are the suggestions tailored to THIS ad, or could they apply to any ad? | "Try a stronger hook" (applies to everything) |
| Multi-turn continuity | Analyze 3 ads in sequence. Does it maintain context and start identifying patterns across them? | Treats each analysis as completely isolated |

---

## 5. Industry Trends (`/industry-trends`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Competitor ad counts match | Open Meta Ad Library for each competitor. Count active ads. Compare to skill's portfolio counts | Within 10% (some lag is expected) |
| Cited competitor ads exist | For each specific ad referenced, verify it exists in the ad library | All cited ads are real |
| Format mix percentages plausible | Eyeball the competitor's ad library. Does the reported video/static/carousel split look right? | Roughly correct proportions |
| Transcript quotes are real | For quoted hooks, verify against the actual ad content | Verbatim match |
| "Recently killed" ads accurate | Check if cited inactive ads were actually recently active | Ads existed and were recently paused/removed |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Convergence vs. differentiation | Does it distinguish what EVERYONE is doing (table stakes) from what's UNIQUE to specific competitors? | Treating every competitor's approach as equally noteworthy |
| Gaps are actually gaps | Are the "opportunities" genuinely underexplored, or just things competitors haven't tried because they don't work? | "Nobody is running print ads" (not a gap, wrong channel) |
| Competitive signals framed as bets, not results | Does it correctly caveat that ad library data shows STRATEGY, not PERFORMANCE? | "Competitor X's video ads are performing well" (we can't know that) |
| Implications are brand-specific | Do the "implications for your strategy" account for YOUR brand's positioning, audience, and constraints? | Generic: "Consider testing video ads" |

---

## 6. Competitor Watch (`/competitor-watch`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| All tracked competitors included | Check workspace's competitor list in Motion. Compare to report | No competitors missing |
| limit=500 captures full portfolio | For a competitor with 100+ active ads, verify the count matches ad library | Count matches or is very close |
| Baseline delta detection works | Run twice (1 week apart). Second run should show deltas vs. first run | Deltas are accurate (new ads flagged, removed ads flagged) |
| Slack canvas created correctly | Check the Slack channel. Is the canvas well-formatted and complete? | Canvas exists, readable, contains full report |
| Local baseline files written | Check `~/.claude/competitor-watch/` for baseline files | Files exist per competitor with expected content |

### Insight Quality

| Check | How to Verify | Fail Example |
|-------|-----------------|--------------|
| Delta-focused (not full portfolio rehash) | Second run should lead with what CHANGED, not repeat the full portfolio analysis | Second run looks identical to first run |
| Survival rate analysis meaningful | Does "30-day launch survival rate" reveal something about the competitor's testing velocity? | Just reporting a number without interpretation |
| "Your Brand Benchmark" is comparative | Does the comparison table reveal actual strategic gaps, not just format counts? | "You have 15 video ads, they have 20" with no strategic takeaway |
| Recommendations are prioritized | Are the 2-3 recommended actions ranked by impact and feasibility? | List of things to "consider" with no priority |

---

## 7. Write Hooks (`/write-hooks`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Top HOOK performers match app | Open Motion > sort by Hook Rate. Compare top performers cited | Same creatives referenced |
| Extracted hook language is verbatim | Compare "proven hooks from your account" to actual creative content | Exact match |
| Awareness stage mapping is reasonable | For each hook, does the assigned awareness stage match the hook's intent? | "Problem-aware" hook that actually targets "most-aware" audience |
| Tactic classification accurate | Compare hook to the tactic definition in hook-tactics.md | Tactic label matches what the hook actually does |

### Insight Quality

| Check | How to Verify | Fail Example |
|-------|-----------------|--------------|
| Hooks pass the scroll-stop test | Read each hook. Would YOU stop scrolling? | Generic: "Are you tired of [problem]?" |
| Psychological triggers are genuine | Does the trigger create real cognitive friction, or is it labeled but absent? | Labeled "curiosity gap" but the hook reveals everything upfront |
| Hooks are diverse | Do they span multiple awareness stages and triggers, or cluster? | 8 hooks that are all "problem-aware" with "social proof" trigger |
| Voice matches brand | Do hooks sound like YOUR brand, not generic ad copy? | Hooks could be for any brand in any industry |
| Coverage gap identification | Does it flag which awareness stages or triggers are UNTESTED in your account? | Only generates hooks without noting what's missing |

---

## 8. Build Brief (`/build-brief`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Reference creatives are real and relevant | Verify cited "Visual Example" creatives exist in Motion and match the concept | Real creatives, relevant format/angle |
| Success metrics use correct goalMetric | Verify the brief's success criteria use the workspace's actual goal metric | Matches workspace configuration |
| Brand voice constraints respected | Compare brief copy to brand do's/don'ts in config | No violations of Creative Don'ts |

### Insight Quality

| Check | How to Verify | Fail Example |
|-------|-----------------|--------------|
| Copy is complete (no placeholders) | Read the copy section. Are there any `[insert X]` or `[your product]` placeholders? | Any placeholder text whatsoever |
| Visual approach is specific enough | Could two different designers create something similar from this description? | "Clean, modern aesthetic" (too vague) |
| Hook is scripted verbatim (video) | Is the hook exact words a creator would say, with timestamps? | "Start with an attention-grabbing hook" |
| Concept is data-grounded | Can you trace the concept back to a specific data insight? | Concept seems to come from nowhere |
| Creator could execute without questions | Show the brief to a creator. Would they need to ask clarifying questions? | Brief requires follow-up to understand intent |

---

## 9. Create Concepts (`/create-concepts`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| "Why It Can Work" cites real data | Each concept's rationale should reference specific performance data, competitor signals, or taxonomy gaps | Every concept has at least one verifiable data point |
| Reference creatives exist | Verify all cited reference creatives exist in Motion | All references are real |
| Production difficulty matches config | If config says "max difficulty: medium", no concepts should be "hard" | Respects production constraints |

### Insight Quality

| Check | How to Verify | Fail Example |
|-------|-----------------|--------------|
| Concepts are specific enough to brief | Could you hand this concept card to a designer and they'd know what to make? | "A testimonial-style ad highlighting product benefits" |
| Concepts are different from each other | Do they explore different angles/formats/audiences, or are they variations of the same idea? | 3 concepts that are all "UGC testimonial about pain point X" |
| At least one concept is non-obvious | Is there a concept you wouldn't have thought of from the dashboard alone? | All concepts are obvious extensions of top performers |
| Taxonomy gap exploitation | Does at least one concept target an untested glossary combination? | All concepts reinforce existing patterns |

---

## 10. Concept Engine (`/concept-engine`)

### Data Fidelity

Same as Create Concepts, plus:

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Raw input is actually incorporated | Provide meeting notes mentioning a specific idea. Verify the output builds on it rather than ignoring it | Input ideas are reflected in concepts |
| Strategic position matrix makes sense | Are concepts placed correctly on pain × persona × awareness? | Placements are defensible |

### Insight Quality

Same as Create Concepts, plus:

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Raw input elevated, not just echoed | Does it take a rough idea and make it strategically stronger, or just reformat it? | Input: "maybe a UGC ad about sleep" → Output: "UGC ad about sleep" (no elevation) |
| Matrix reveals gaps | Does the strategic map show where the portfolio is concentrated and where it's thin? | Matrix shown but not interpreted |

---

## 11. Find Iterations (`/find-iterations`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Fatigue detection matches app | Check creatives the skill flags as fatiguing. Do they show declining metrics in Motion? | Declining trend confirmed in app |
| Diversity audit categories match glossary | Compare the diversity audit's categories to the glossary in Motion | Uses real glossary categories, not invented ones |
| Coverage gap identification is real | Check glossary for the "untested combinations" cited. Are they actually untested? | Combinations cited as untested actually have creatives tagged |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Iteration type is specific | Is it "Hook Refresh" vs. "Format Swap" vs. "Audience Pivot" — or vague "iterate"? | "Try a new version of this creative" |
| Exploitation vs. exploration balance assessed | Does it tell you whether you're over-indexing on proven patterns or under-investing in new territory? | Only suggests iterating on winners, never exploring |
| Expected impact is calibrated | Are impact estimates grounded in comparable precedents? | "High impact" with no justification |
| Iteration cards are briefable | Could you hand an iteration card directly to a designer? | Card requires extensive interpretation |

---

## 12. UGC Scripts (`/ugc-scripts`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Hook derived from top performers | Compare the scripted hook to top-performing UGC hooks in the account | Clear lineage to proven hooks |
| Creator profile matches demographic data | Compare the suggested creator demographics to who actually converts | Targeting the right demographic segments |
| Brand voice respected in talking points | Compare talking points to brand do's/don'ts | No violations |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Sounds like a real person | Read the talking points aloud. Do they sound natural, or like ad copy? | "Our revolutionary solution transforms your daily routine" |
| Hook is scripted, rest is talking points | Is the hook exact words, while the body gives direction without being word-for-word? | Everything is word-for-word scripted (too rigid for UGC) |
| CTA is soft | Is the close a natural invitation, not a hard sell? | "Click the link below and use code SAVE20" |
| Script structure serves the format | Does the pacing work for the specified duration? | 60 seconds of content crammed into a 15-second script |

---

## 13. QA Feedback (`/qa-feedback`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Brand guidelines applied correctly | Check each "Brand Alignment" score against actual brand do's/don'ts in config | Hard fails correctly identified (or correctly not flagged) |
| Comparison to top performers is fair | Are the referenced top performers comparable in format and objective? | Comparing a static image to a 60s video's metrics |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Scorecard dimensions are independently assessed | Each of the 6 dimensions should have its own specific evidence | All dimensions say "looks good" or all say "needs work" with same reason |
| Feedback is actionable | For each "Needs Work" or "Fail", is there a specific fix? | "Hook could be stronger" without saying how |
| Overall verdict matches scorecard | Does Ready/Iterate/Rethink align with the dimension scores? | 4/6 dimensions "Fail" but overall verdict is "Ready to Launch" |
| Catches real issues | Submit a creative with a known problem (wrong brand voice, weak hook). Does it catch it? | Misses the deliberate flaw |

---

## 14. Audience Research (`/audience-research`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Demographic assertions match app | Compare "who converts" claims to Motion's demographic breakdown | Same segments identified |
| Competitor positioning is accurate | Check competitor's actual messaging/positioning against what's reported | Accurate representation |

### Insight Quality

| Check | What to Look For | Fail Example |
|-------|-----------------|--------------|
| Tensions are from audience's POV, not brand's | Are tensions written in the audience's language? | "Users need better project management" (brand language) vs. "I spend half my Monday just figuring out what everyone's working on" (their language) |
| Review extraction preserves exact language | In review mode, are quotes verbatim from actual reviews? | Paraphrased or synthesized quotes |
| Audience cards are differentiated | Do segments represent genuinely different people with different motivations? | Three segments that are basically the same person described slightly differently |

---

## 15. Customize (`/customize`)

### Data Fidelity

| Check | How to Verify | Pass Criteria |
|-------|--------------|---------------|
| Pre-fill data matches Motion | Compare pre-filled brand info, competitors, glossary categories to Motion app | Accurate pre-fill |
| Config file is well-formed | Read the generated `motion-creative.config.md`. Is it parseable and complete? | Valid YAML frontmatter, all sections present |
| Config influences other skills | Run `/performance-analysis` before and after customization. Does the output respect new settings? | Primary KPI changes analysis focus; excluded metrics don't appear |

### Insight Quality

N/A — this is a setup skill, not an analysis skill. Quality check is whether the config it produces makes downstream skills better.

---

## Cross-Cutting Tests

### Data Accuracy Stress Tests

| Test | Method | Pass Criteria |
|------|--------|---------------|
| **Empty workspace** | Run skills on a workspace with no creatives | Graceful degradation with clear messaging, no hallucinated data |
| **No competitors tracked** | Run competitive skills with no competitors configured | Clear message, no fabricated competitor data |
| **Single creative** | Workspace with only 1 active creative | No "top performers" language; honest about limited data |
| **Mismatched date range** | Request analysis for a date range with no activity | Reports "no data" rather than showing stale data |
| **goalMetric variety** | Test across workspaces using different goal metrics (ROAS, CPA, CPC, etc.) | Each skill correctly adapts to the workspace's goal metric |

### Insight Quality Stress Tests

| Test | Method | Pass Criteria |
|------|--------|---------------|
| **The "so what?" test** | For every insight in the output, ask "so what?" — does it lead to an action? | Every insight has a clear implication |
| **The "dashboard screenshot" test** | Could someone get the same insight by looking at the Motion dashboard for 30 seconds? | At least 50% of insights require cross-referencing multiple views or connecting non-obvious dots |
| **The "any brand" test** | Remove the brand name from the output. Could this analysis apply to any brand? | Analysis is clearly specific to THIS brand's data and situation |
| **The "yesterday's news" test** | Run the same skill twice on different days. Is the second run substantively different? | Output changes reflect actual data changes, not just regeneration |
| **The "hallucination audit"** | Flag every factual claim. Verify each against Motion data | Zero fabricated data points. Every number traces to a real source |

### Tool Call Verification

| Test | Method | Pass Criteria |
|------|--------|---------------|
| **SPEND called first** | Monitor tool calls for every skill. Is `get_creative_insights` with SPEND always the first data call? | Always first (required to extract goalMetric and spendThreshold) |
| **spendThreshold respected** | Check if any creative below the threshold is used for performance claims | Never used for directional claims |
| **limit=500 on inspo pulls** | Monitor `get_inspo_creatives` calls for competitor skills | limit=500 for portfolio analysis |
| **Extracted vs. Inferred labeled** | Check if the output distinguishes between data directly from tools vs. calculated/inferred | Clear labeling or at minimum no inferred data presented as extracted |

---

## Execution Order

1. **Run `/customize` first** — establishes baseline config
2. **Run `/performance-analysis`** — validates core data pipeline against Motion app
3. **Run `/morning-briefing`** — validates delta calculation and conciseness
4. **Run `/analyze-ad` on 3 creatives** — validates per-creative accuracy and multi-turn
5. **Run `/industry-trends`** — validates competitor data pipeline
6. **Run `/competitor-watch` twice** (1 week apart) — validates delta tracking
7. **Run `/write-hooks`** — validates hook data grounding
8. **Run `/create-concepts`** then `/build-brief`** — validates concept-to-brief pipeline
9. **Run `/find-iterations`** — validates fatigue detection and diversity audit
10. **Run `/weekly-performance`** — validates WoW calculation and HTML output
11. **Run `/ugc-scripts`** — validates script quality and data grounding
12. **Run `/qa-feedback` on a brief output** — validates feedback loop
13. **Run `/audience-research` in all 3 modes** — validates each mode independently
14. **Run `/concept-engine` with raw input** — validates input processing
15. **Run cross-cutting stress tests** — validates edge cases
