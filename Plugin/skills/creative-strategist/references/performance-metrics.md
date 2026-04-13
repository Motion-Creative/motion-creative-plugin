# Performance Metrics Reference

## Metric Definitions

Each `insightType` in Motion's `get_creative_insights` sorts and surfaces creatives by a different performance dimension. Pull multiple simultaneously — single-metric views lie.

### SPEND
**What it measures:** Total ad spend allocated to this creative.
**What movement signals:** Budget is flowing toward (or away from) this creative. High spend = the algorithm or media buyer is betting on it.
**Watch for:** High spend + declining ROAS = scaling past efficiency. High spend + low CPA = a reliable workhorse.

### SCALING
**What it measures:** Whether spend on this creative is increasing over time — it's gaining budget share.
**What movement signals:** The algorithm is finding more efficient pockets for this creative, or a media buyer is actively scaling it.
**Watch for:** Scaling creatives are the ones to watch. A creative that's scaling with stable ROAS is the best signal in the account. Scaling + declining ROAS = hitting saturation.

### HOOK
**What it measures:** Thumbstop rate — the percentage of viewers who stop scrolling to watch. First-3-seconds effectiveness.
**What "good" looks like:** Highly category-dependent. Compare within your own account, not against industry benchmarks.
**Watch for:** High HOOK + low CPA = strong creative that converts. High HOOK + high CPA = attention without action — the hook promises something the ad doesn't deliver. Low HOOK = the opening isn't working, regardless of what follows.

### ROAS
**What it measures:** Return on ad spend — revenue generated per dollar spent.
**What movement signals:** Rising ROAS = creative is finding its audience. Declining ROAS = fatigue, saturation, or audience mismatch.
**Watch for:** ROAS is the closest proxy to "is this working?" but it can be gamed by low spend. Always cross-reference with SPEND — a creative with 10x ROAS on $50 spend is not proven.

### PURCHASES
**What it measures:** Total purchase conversions attributed to this creative.
**What movement signals:** Volume of downstream action. High purchases = the creative drives action at scale.
**Watch for:** High purchases + high CPA = scaling but expensive. Compare purchase volume against spend to spot efficiency opportunities.

### PURCHASE_VALUE
**What it measures:** Total revenue from purchases attributed to this creative.
**What movement signals:** Not just conversions, but the value of those conversions. A creative that drives fewer but higher-value purchases may outperform one with more conversions.

### CTR_ALL
**What it measures:** Click-through rate across all click types (link clicks, profile clicks, etc.).
**Watch for:** CTR without conversion data is a vanity metric. Always pair with CPA or ROAS. High CTR + high CPA = curiosity clicks that don't convert.

### CPA
**What it measures:** Cost per acquisition — how much it costs to drive one purchase.
**What "good" looks like:** Lower is better. Compare against account average and category benchmarks.
**Watch for:** Best efficiency metric for direct-response. Low CPA + low spend = an underexploited gem. Low CPA + scaling = the dream.

### CPC
**What it measures:** Cost per click.
**Watch for:** Useful for traffic campaigns, less meaningful for conversion-optimized campaigns. Low CPC + high CPA = cheap clicks that don't convert.

---

## Metric Combinations That Tell Stories

| Combination | What It Means | Action |
|---|---|---|
| High SPEND + high ROAS + SCALING | Best creative in account — proven and growing | Protect and iterate |
| High SPEND + declining ROAS | Scaling past efficiency — fatigue or saturation | Consider pausing or refreshing |
| High HOOK + low CPA | Strong creative that hooks and converts | Scale aggressively |
| High HOOK + high CPA | Attention without action — hook/body disconnect | Rework the body/CTA, keep the hook |
| Low HOOK + low CPA | Weak hook but converts whoever sees it | Test new hooks on the same body |
| High ROAS + low SPEND | Underexploited gem — efficient but not scaling | Increase budget, test in new audiences |
| SCALING + stable ROAS | The best signal — growing without losing efficiency | Scale and monitor |
| SCALING + declining ROAS | Hitting saturation — growing past its audience | Cap spend, find new angles |
| Strong goal metric + cost-cap ad set | Efficiency may be inflated by the constraint, not the creative | Validate in uncapped environment before scaling |
| Weak goal metric + prospecting campaign | Expected — cold audiences convert less on first touch | Compare to other prospecting creatives, not account average |
| High SPEND + weak goal metric + "Top performers" ad set | Stale creative in a privileged position — budget trap | Remove from top performers, replace with validated winner |
| Strong goal metric + low SPEND + testing campaign | Strong early signal but unproven at scale | Graduate to scaling campaign for volume validation |

---

## datePreset Guidance

| Preset | When to Use |
|---|---|
| TODAY | Intraday monitoring — may have limited data early in the day. Unreliable before noon. |
| YESTERDAY | Daily delta — the workhorse for morning briefings. Full day of data. |
| LAST_7_DAYS | Short-term trends — what's happening this week. Good for spotting emerging winners/losers. |
| LAST_14_DAYS | Medium-term patterns — smooths out daily noise. |
| LAST_30_DAYS | Baseline analysis — the default for performance-analysis. Enough data to see real patterns. |
| LAST_90_DAYS | Strategic patterns — what's been working over a quarter. Good for identifying durable winners vs. flash-in-the-pan creatives. |
| THIS_MONTH / LAST_MONTH | Calendar-aligned reporting — useful when comparing month-over-month. |

For delta analysis (morning briefing), always pull at least two windows (YESTERDAY + LAST_7_DAYS) to compute what changed.

---

## Demographic Interpretation

Age/gender breakdowns reveal audience-creative fit:

- **Skew toward one segment:** The creative's messaging resonates with that audience specifically. This is signal, not noise — it tells you who the message is for.
- **Flat performance across segments:** Either the message is universal (rare) or the creative isn't strong enough to differentiate.
- **Strong with one gender, weak with another:** Usually a messaging or casting issue, not a targeting issue.
- **Performance drops with age:** Often signals a trend-dependent hook that doesn't translate to older demographics.
- **Strong with older, weak with younger:** May indicate a trust/credibility-focused message that younger audiences don't need.

Use demographic data to inform concept generation — if 25-34 women drive your best CPA, concepts should include angles specifically designed for that segment.
