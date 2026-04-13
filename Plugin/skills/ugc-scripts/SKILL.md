---
name: ugc-scripts
description: >
  Write authentic UGC (user-generated content) ad scripts that sound like real people
  sharing real experiences. Produces talking points — not word-for-word scripts — with
  a scripted hook and soft CTA. Grounded in Motion workspace data for hooks that convert.
  Activate when users ask for "UGC script", "creator script", "influencer brief",
  "talking points for a creator", "authentic ad script", or need scripts for
  creator-style paid social ads.
argument-hint: "[hook or concept] [product] [--creator-type any] [--duration 30s]"
allowed-tools:
  - Read
  - "mcp__motion__get_auth_context"
  - "mcp__motion__get_workspace_brand"
  - "mcp__motion__get_creative_insights"
  - "mcp__motion__get_creative_transcript"
  - "mcp__motion__get_glossary_values"
  - "mcp__motion__get_demographic_breakdown"
model: opus
---

# UGC Script Writer

Create authentic creator-style scripts for paid social ads. The core test: **would a real person actually say this out loud?**

Scripts are talking points with a scripted hook — not teleprompter copy. The creator should rephrase everything in their own words except the opening line.

---

## Phase 1: Discovery

### 1a. Parse Arguments

- **Hook or concept** (`$ARGUMENTS`): The opening line, concept direction, or product angle. Can be a specific hook from `/write-hooks` or a general direction.
- **Product**: What's being sold. If not in args, ask.
- **`--creator-type`**: micro-influencer, customer-testimonial, expert, founder, or any. Default: any.
- **`--duration`**: Target length. Default: 30s (75-100 words). Options: 15s, 30s, 45s, 60s.

If hook and product aren't provided, ask before proceeding.

### 1b. Load Context

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for brand voice, creative do's/don'ts, target demographics, production constraints. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `default_date_preset`: LAST_30_DAYS
   - Brand guidelines: pull from `get_workspace_brand`
   - `production_capabilities`: all formats
2. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/SKILL.md` for methodology.
3. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/brief-template.md` for output structure reference.
4. Use `default_date_preset` from settings as the datePreset for all calls unless the user specified a date range.

### 1c. Pull Performance Data

**The SPEND call must come first** (it returns goalMetric and spendThreshold). Then dispatch remaining calls in parallel.

1. `get_auth_context()` — resolve workspace
2. `get_creative_insights(workspaceId, insightType="SPEND", datePreset="LAST_30_DAYS", limit=10, withAggregatedInsights=true)` — spend leaders + account-level aggregates

→ Extract `goalMetric` and `spendThreshold` from the response. Use goalMetric for all efficiency-sorted calls.

3. `get_workspace_brand(workspaceId)` — brand voice, positioning, creative constraints
4. `get_creative_insights(workspaceId, insightType="HOOK", datePreset="LAST_30_DAYS", limit=10)` — top-performing hooks
5. `get_creative_insights(workspaceId, insightType=goalMetric, datePreset="LAST_30_DAYS", limit=10)` — efficiency leaders by workspace goal metric
6. `get_glossary_values(workspaceId)` — taxonomy (especially UGC-related categories)
7. `get_demographic_breakdown(workspaceId)` — who's responding

Then pull transcripts from the top 3-5 UGC-style performers (identify from glossary tags or visual format):
8. `get_creative_transcript(creativeEntityId, creativeOrigin)` — see what hooks and pacing work

---

## Phase 2: Emotional Mapping

Before writing, plan how the viewer moves from attention through to action.

### 2a. Define the Emotional Arc

Every UGC script follows this arc:

```
ATTENTION (0-3s) → PROBLEM (3-8s) → SOLUTION (8-15s) → PROOF (15-22s) → CTA (22-30s)
```

For each beat:
- **Attention**: The hook. This IS scripted — exact words matter. What stops the scroll?
- **Problem**: Name the specific pain or frustration. Mirror their internal monologue.
- **Solution**: Introduce the product naturally. NOT a pitch — a discovery or recommendation.
- **Proof**: Specific detail that makes it believable. One feature, one result, one moment.
- **CTA**: Soft invitation. NOT "buy now." More like "try it for yourself" or "link's in my bio."

### 2b. Identify the Single Key Message

UGC fails when it tries to say too much. Choose ONE:
- One pain point
- One key benefit
- One surprising feature
- One transformation

Everything in the script serves this one message.

### 2c. Gather Authenticity Details

From brand context and performance data, collect:
- Specific product features (NOT marketing claims — real functional details)
- Social proof snippets (review quotes, numbers, real results)
- Sensory details that make the product tangible ("leaf blower outside, gone" > "great noise canceling")

---

## Phase 3: Build the Script

### Creator Profile

Before writing, define who's speaking:

- **Demographics**: Age range, gender, look/feel
- **Persona match**: Which pain × persona intersection from the Creative Strategy Engine does this creator represent? The creator should BE the persona, not act as one.
- **Energy/vibe**: How they come across on camera (e.g., "casual and direct, mid-conversation energy, not performative")
- **Credibility signal**: What makes them believable — actual customer, expertise, lifestyle match
- **Casting note**: 1-2 sentences a casting director could use

### Script Structure

```
HOOK (0:00-0:03): "[EXACT SCRIPTED WORDS]"
— This is the one thing that's verbatim. Every other beat is talking points.

PROBLEM/SETUP (0:03-0:08):
Talking point: [What the creator should communicate — in their own words]
Tone: [How they should sound — frustrated, amused, confessional]

SOLUTION/DISCOVERY (0:08-0:15):
Talking point: [How they discovered or started using the product]
Tone: [Genuine surprise, casual recommendation, matter-of-fact]
Show: [What they do on camera — hold product, demonstrate, react]

PROOF/DETAIL (0:15-0:22):
Talking point: [One specific detail that makes it real]
Tone: [Honest, specific — not hyperbolic]
Optional: [Quote a review, show a before/after, mention a timeframe]

CTA (0:22-0:30):
Talking point: [Soft invitation — NOT a hard sell]
Examples: "Try it for yourself" / "Link's in my bio" / "I'm telling everyone about this"
```

### Word Count Guidelines

| Duration | Word Count | Beats |
|---|---|---|
| 15s | 35-50 words | Hook + Problem + Quick CTA |
| 30s | 75-100 words | Full arc |
| 45s | 100-125 words | Full arc with extended proof |
| 60s | 125-150 words | Full arc with storytelling |

### The Read-Aloud Test

Read the entire script out loud. If ANY line:
- Sounds like it was written by a brand → rewrite it
- Uses words like "revolutionary", "game-changing", "innovative" → delete them
- Feels like reading a teleprompter → make it conversational
- Includes more than 2 product features → cut to 1

### Authenticity Guidelines

**Setting**: Where to film — must match persona's life context
- Customer testimonial: their home, car, bathroom, kitchen
- Expert: office, studio, relevant professional setting
- Founder: behind-the-scenes, warehouse, office

**Do's:**
- Pause and think before key points (real people don't have instant answers)
- Show the actual product in use, not a beauty shot
- Include one imperfect moment (stumble, laugh, aside)
- Reference a specific moment or experience, not a general benefit
- Use "I" language: "I noticed..." "For me..." "What I like..."

**Don'ts:**
- Don't read from a teleprompter (talking points, not scripts)
- Don't use marketing language ("clinically proven", "dermatologist recommended")
- Don't start with "Hey guys!" (oversaturated, instantly signals ad)
- Don't pack multiple features (one message per script)
- Don't end with aggressive urgency ("Buy NOW before it's GONE!")

### Brand Safety Guardrails

From workspace brand + settings:
- Claims that CANNOT be made (legal/compliance)
- Competitor mentions: allowed or not
- Required disclosures (FTC, #ad, etc.)
- Brand elements that must appear (logo, product shot, URL)

---

## Quality Gate

Before presenting scripts, verify each against these tests: Does the hook pass the 5-test quality bar (genuine cognitive friction, strong trigger, complete thought, authentic voice, personal stakes)? Do the talking points sound like a real person sharing an experience, not a scripted ad? Is the CTA soft and natural, not salesy? Revise any script that fails.

---

## Phase 4: Output

### Pre-Script Context

3-5 bullets showing:
- What performance insight shaped the talking points
- Which persona the creator maps to
- What reference creatives informed the hook
- What brand constraints shaped the guardrails

### The Script

Present the full script structure with creator profile, talking points per beat, and authenticity guidelines.

### Deliverables Section

- Number of takes expected (recommend 3-5 for variety)
- Aspect ratios needed
- Audio requirements (clean audio, no background music unless specified)
- Raw footage requirements

### Success Metrics

- **Primary**: HOOK rate — UGC lives or dies by the first 3 seconds
- **Secondary**: goalMetric (workspace efficiency metric)
- **Evaluation window**: After sufficient spend (reference workspace's spendThreshold)
- **Diagnostic**:
  - HOOK is low → the opening line isn't working. Test a different hook.
  - HOOK is high but CPA is poor → talking points need refinement. The hook promises something the body doesn't deliver.

### Close

"Want me to write variations with different hooks? Or turn this into a full production brief with `/build-brief`?"

---

## Common Mistakes to Catch

- **Feature dumping**: More than 2 features in a 30s script → cut to 1
- **Fake enthusiasm**: "OH MY GOD you guys" without substance → replace with specific detail
- **Missing the hook's promise**: Hook says one thing, body talks about something else → realign
- **Corporate tone**: Any sentence that sounds like a press release → rewrite in first person
- **No specific proof**: Generic "it works so well" → replace with "after 2 weeks, [specific result]"
- **Hard sell CTA**: "Use code X for 20% off!" → soften to "honestly just try it"

---

## Error Handling

Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/error-handling.md` for degraded mode and error handling guidance.
