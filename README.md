# Motion Creative — Plugin for Claude Code

Your creative strategy team just got an upgrade. This plugin connects [Motion's](https://motionapp.com) creative analytics directly to Claude Code — so you can analyze performance, research competitors, generate concepts, and write briefs without leaving your terminal.

Think of it as a strategist who knows your account inside and out, ready to work the moment you type `/`.

---

## Install

The plugin bundles both the creative strategy skills and the Motion MCP server connection. Install it and you're ready to go — no separate MCP setup needed.

**Option A — Two commands in Claude Code:**

```
/plugin marketplace add Motion-Creative/motion-creative-plugin
```

```
/plugin install motion-creative@motion-mcp
```

**Option B — Just ask Claude:**

> Install the Motion Creative plugin from https://github.com/Motion-Creative/motion-creative-plugin

When you first use a command, you'll be prompted to authenticate with your Motion account. You need **Owner, Admin, or Collaborator permissions** in your workspace. Don't have an account? You can sign up during authentication and get a 14-day trial.

Once authenticated, run `/customize` to set up your brand, KPIs, competitors, and production constraints. This tailors every command to your workspace.

---

## What you can do

### Analyze your creative performance

| Command | What it does |
|---------|-------------|
| `/morning-briefing` | What changed overnight — performance deltas and competitor activity, ready for standup |
| `/weekly-performance` | Week-over-week performance deck with narrative and recommendations |
| `/performance-analysis` | What's working, what's scaling, what's dying — multi-metric deep dive with demographic splits |
| `/analyze-ad` | Deep-dive on a specific creative by name, ID, or URL |

### Develop new creative

| Command | What it does |
|---------|-------------|
| `/create-concepts` | 2–5 data-driven concept ideas pulled from your performance data and competitive signals |
| `/concept-engine` | Advanced concept generation from brainstorm notes, meeting transcripts, or blank-slate exploration |
| `/build-brief` | Structured creative brief ready to hand to your team or creators |
| `/find-iterations` | Specific iteration ideas for your top performers and fatiguing ads — new hooks, format variations, audience pivots |

### Write copy and scripts

| Command | What it does |
|---------|-------------|
| `/write-hooks` | Psychologically-driven hooks grounded in your workspace's actual top performers |
| `/ugc-scripts` | Authentic UGC talking points that sound like real people, not robots |

### Research and QA

| Command | What it does |
|---------|-------------|
| `/audience-research` | Tension-based audience mapping, customer review mining, and competitive creative analysis |
| `/industry-trends` | What competitors are running — creative trends, formats, hooks, and messaging strategies |
| `/qa-feedback` | Review a creative against your brand guidelines, performance patterns, and concept standards |

---

## How it works

This plugin adds creative strategy skills to Claude Code. Each command calls the Motion MCP server to pull your actual workspace data — performance metrics, competitor libraries, brand context, AI tags — and uses it to ground every output in what's really happening in your account.

No copy-pasting dashboards. No context switching. Just ask.

---

Built by [Motion](https://motionapp.com) — the creative analytics platform for paid social teams.
