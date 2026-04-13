---
name: write-hooks
description: >
  Generate psychologically-driven hooks for paid social ads. Uses Schwartz's 5 awareness
  stages and 8 psychological triggers, grounded in the workspace's actual top-performing
  hooks from Motion. Activate when users ask to "write hooks", "generate hooks",
  "hook ideas", "scroll-stopping openings", "attention grabbers", or need opening lines
  for ads. Pairs with write-hooks/references for tactic and template libraries.
argument-hint: "[product] [pain/desire] [persona] [--stage problem-aware] [--format video] [--count 10]"
allowed-tools:
  - Read
  - "mcp__motion__get_auth_context"
  - "mcp__motion__get_workspace_brand"
  - "mcp__motion__get_creative_insights"
  - "mcp__motion__get_creative_transcript"
  - "mcp__motion__get_glossary_values"
  - AskUserQuestion
model: opus
---

# Hook Writing Engine

Generate psychologically-driven hooks that capture attention in the first 1-3 seconds of paid social ads. Every hook is grounded in workspace performance data from Motion.

## Phase 0: Orient

1. **Acknowledge**: "I'll generate hooks grounded in what's actually converting in your account."
2. **Ask if missing**: "Product, target persona, and pain point? Or should I pull these from your workspace data?" If any required input is ambiguous, ask — don't assume.
3. **Important distinction**: Hooks that stop the scroll and hooks that convert are often different. After generating, note which hooks optimize for attention vs. which optimize for action.
4. **Connected workflow**: After hooks, you can brief the full ad (`/build-brief`) or analyze how existing hooks perform (`/analyze-ad`).

If the user provided product, persona, and pain point clearly, skip questions.

## Core Principle

A hook is the tactical expression of a messaging angle at a specific awareness stage. Before writing hooks, understand what the audience already knows and feels.

---

## Hook Sources

Pain points are the most reliable hook source. Start here before trying other angles. See `references/hook-sources.md` for the full breakdown.

6 categories of pain-based hook material:
1. **Emotional pain points** (start here) — what keeps them stuck, worried, or frustrated
2. **Daily annoyances** — small friction the audience lives with every day
3. **Embarrassing habits** — what they do but don't talk about
4. **Silent frustrations** — what they tolerate because they think it's just normal
5. **Protective beliefs** — what they tell themselves to avoid having to change
6. **Self-awareness moments** — the truth they already half-know but haven't admitted

When generating a set of hooks, mine at least 2-3 different source categories to ensure emotional variety.

---

## Phase 1: Gather Inputs

### 1a. Parse Arguments

Extract from `$ARGUMENTS`:
- **Product**: What does it do?
- **Pain or Desire**: What specific need does it address?
- **Persona**: Who are you targeting? (life context, not just demographics)
- **`--stage`**: Awareness stage (unaware, problem-aware, solution-aware, product-aware, most-aware). Default: generate across all stages.
- **`--format`**: Video or static. Default: video.
- **`--count`**: How many hooks. Default: 10.

If any required input is missing, ask the user before proceeding.

### 1b. Load Context

1. Read `${CLAUDE_PLUGIN_ROOT}/motion-creative.config.md` for brand guidelines, primary KPI, target demographics. If the file does not exist, use these defaults and suggest the user run `/customize`:
   - `primary_kpi`: use goalMetric from first `get_creative_insights` response
   - `default_date_preset`: LAST_30_DAYS
   - Brand guidelines: pull from `get_workspace_brand`
2. Read `${CLAUDE_SKILL_DIR}/../creative-strategist/SKILL.md` for methodology.
3. Use `default_date_preset` from settings as the datePreset for all calls unless the user specified a date range.
4. Read `${CLAUDE_SKILL_DIR}/references/hook-tactics.md` for the 35+ tactic library.
5. Read `${CLAUDE_SKILL_DIR}/references/hook-voice-patterns.md` for proven template clusters.
6. Read `${CLAUDE_SKILL_DIR}/references/hook-sources.md` for pain-based hook material. Pain points are the most reliable hook source — start here before trying other angles.
7. Read `${CLAUDE_SKILL_DIR}/references/hook-eval-judge.md` for the 5-test quality bar and calibrated examples.

### 1c. Pull Performance Data (Parallel)

Dispatch all in parallel:
1. `get_auth_context()` — resolve workspace
2. `get_workspace_brand(workspaceId)` — brand voice, positioning, creative do's/don'ts
3. `get_creative_insights(workspaceId, insightType="HOOK", datePreset="LAST_30_DAYS", limit=10)` — top hooks in the account
4. `get_glossary_values(workspaceId, includeCreativeIds=true)` — hook taxonomy tags

Then for the top 3-5 video creatives from hook results:
5. `get_creative_transcript(creativeEntityId, creativeOrigin)` — extract the actual hooks word-for-word

---

## Phase 2: Analyze What's Working

Before generating new hooks, understand the workspace's proven patterns:

### 2a. Extract Existing Hooks
From transcripts, pull the first 1-3 seconds of each top performer. Identify:
- What psychological trigger does each use?
- What awareness stage does each target?
- What makes each hook specific (names a moment, uses a number, calls out an identity)?

### 2b. Map Taxonomy Patterns
From glossary values, check which hook-related categories exist:
- Which hook tactics are over-represented in top performers?
- Which are absent (untested opportunities)?
- What messaging angles pair with the best hooks?

### 2c. Identify Brand Constraints
From workspace brand + settings:
- What tone must hooks match?
- What claims are off-limits (Creative Don'ts)?
- What language or framing does the brand avoid?

---

## Phase 3: Generate Hooks

### Awareness Stages & Psychological Triggers

Reference the 5 Awareness Stages and 8 Psychological Triggers from `${CLAUDE_SKILL_DIR}/../concept-engine/references/creative-strategy-engine.md`. Each awareness stage demands a different hook strategy; each hook leverages one or two psychological triggers. The strongest hooks combine two triggers (e.g., Identity Call-Out + Pain Agitation).

### Generation Rules

For each hook:
1. **Select awareness stage** (from args or cycle through all 5)
2. **Select 1-2 psychological triggers** (vary across the set)
3. **Apply a specific tactic** from `references/hook-tactics.md`
4. **Consider voice patterns** from `references/hook-voice-patterns.md` for natural phrasing
5. **Ground in data** — reference what's working in the account if relevant
6. **Check against brand constraints** — no violations of Creative Don'ts

### Hook Quality Standards

Strong hooks:
- Mirror the reader's voice — sounds like something they'd think or say
- Include specific details (numbers, moments, sensory details)
- Lead with pain or curiosity, never with the product
- Feel personally targeted, not broadcast
- Create tension that demands resolution

Weak hooks (never generate these):
- "Introducing..." or "Discover..." openings
- Corporate or marketing language
- Generic category claims ("The best X for Y")
- Product-first messaging at early awareness stages
- Hooks that could apply to any competitor's product

### Common Hook Mistakes

**Describing instead of hitting.** Feature statements, product claims, and benefit summaries are not hooks. Nobody stops scrolling for facts. If the line is about the product rather than the viewer's life, it's a description, not a hook.

**Being too generic.** If every competitor in the category could run the same line, it's not a hook. Hooks should feel like they're talking to one specific person in one specific situation.

**Telegraphing that it's an ad.** "Introducing," "discover," "learn how," "have you tried" — these phrases sound like ads and trigger the urge to skip. Hooks should sound like something a real person might think or say.

**Curiosity without stakes.** Teaser-style hooks that promise something surprising without explaining why it matters to the viewer are just noise. The viewer needs to feel why this is relevant to them.

**Benefits without tension.** The tension — the gap between where the viewer is now and where they want to be — is what makes people stop. Benefits alone describe the destination; tension makes the viewer feel the distance from where they currently are.

**Not hitting a pain point.** If the hook doesn't connect to something the viewer is actually struggling with, frustrated by, or worried about, it has no grip. Pain points are the most reliable way to make someone feel seen.

### Video vs. Static

**Video hooks** (first 3 seconds spoken):
- Conversational. Would sound natural said to a friend.
- Build into a narrative. The hook is a door, not a destination.
- Read it aloud — if it sounds scripted, rewrite it.

**Static hooks** (headline):
- 5-8 words max. Scannable instantly.
- Complete thought on its own — no context needed.
- Creates visual tension with the image.

### Refining Existing Hooks

When the user provides existing hooks to improve or iterate on:

1. **Identify what's working** before changing anything — don't lose strong elements
2. **Identify what the user wants**: variations (same angle, different tactic), improvements (same tactic, sharper execution), or iterations (evolve the concept)
3. **Lead with the improved version**. Note what changed in one line. Don't include the original unless asked.

---

## Phase 4: Output

### Format

Present hooks organized by awareness stage, with the psychological trigger labeled:

```
## [Awareness Stage]

1. "[Hook text]"
   Trigger: [Primary trigger] + [Secondary trigger]
   Tactic: [From hook-tactics library]
   Psychological trigger: [Name the specific trigger mechanism]
   Why: [One sentence — what makes this stop the scroll for this audience]

2. "[Hook text]"
   ...
```

### Video Hook Components

When `--format video`, label each hook with three distinct layers:

- **Spoken**: The first words said on camera. Must work even without visuals.
- **Visual**: What happens in the first frame — the action, scene, or visual element that stops the scroll before a word is spoken.
- **Text overlay**: On-screen text that appears over the visual. Can reinforce the spoken hook, add a second layer, or carry the hook entirely for silent viewers.

The three should work together — not repeat each other. Label each component clearly in the output.

### Grounding Note

Open with 2-3 sentences about what you found in the workspace's top hooks — what patterns are proven, what's missing. This grounds the output in data, not generic advice.

### Close

End with: "Want me to develop any of these into full ad concepts? Run `/create-concepts` with the hooks you like, or I can build a brief with `/build-brief`."

---

## Phase 5: Self-Eval Loop

Before returning hooks to the user, evaluate every hook against the quality bar in `references/hook-eval-judge.md`.

1. **Run the 5-test check** on each hook: genuine cognitive friction, strong psychological trigger, complete thought, authentic voice, personal stakes. A hook must pass all 5 tests.
2. **Discard failures.** Do not try to fix a failing hook — generate a replacement from a different tactic and source.
3. **Generate replacements** until you hit the requested `--count` with all-passing hooks.
4. **Be ruthless.** About 35% of hooks typically pass this bar. If the first draft yields 10 hooks, expect to keep 3-4 and replace the rest.

### Output per hook after self-eval:

Each hook in the final output should include:
- Hook text
- Tactic used
- Psychological trigger
- One sentence on why it stops the scroll

Cross-reference the "What ALWAYS Fails" list and calibrated examples in `references/hook-eval-judge.md`. If a hook matches any anti-pattern, discard it immediately.

---

## Error Handling

Read `${CLAUDE_SKILL_DIR}/../creative-strategist/references/error-handling.md` for degraded mode and error handling guidance.
