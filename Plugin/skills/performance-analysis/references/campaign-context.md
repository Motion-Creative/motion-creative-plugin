# Campaign & Ad Set Context Reference

## Metric Terminology

This document uses **"goal metric"** to mean whatever efficiency metric the workspace optimizes for — ROAS, CPA, purchase_value, or a custom conversion. On your first `get_creative_insights` call, extract `goalMetric` from the top-level response and substitute it throughout this framework.

- **ROAS-type metrics** (ROAS, purchase_value): higher is better. "Above breakeven" = above 1x ROAS or above the brand's target.
- **CPA-type metrics** (CPA, CPC, cost_per_lead): lower is better. "Above breakeven" = below the brand's target CPA.
- **Custom conversions**: check `goalMetric.isCustomConversion`. The direction (higher/lower = better) depends on the specific metric — infer from name or ask.

When this document says "strong efficiency" or "declining efficiency," always interpret in the correct direction for the workspace's goal metric.

---

## Why Campaign Context Matters

A creative's performance metrics are meaningless without knowing where it runs. The same creative can show weak efficiency in a prospecting campaign and strong efficiency in retargeting — that's not two different creatives, it's two different jobs.

**Every performance finding must be qualified by campaign tier.** Never compare efficiency across campaign tiers without acknowledging the difference.

---

## Campaign Tier Interpretation

### Prospecting / TOFU / New Customer Acquisition
**Common naming signals:** "AOFU", "NC", "New Customer", "Prospecting", "TOF", "Cold", "Awareness"

**Efficiency expectations:** Weakest in the account. Below-breakeven efficiency can be acceptable if customer LTV justifies the acquisition cost. A prospecting creative with weak first-touch efficiency that acquires high-LTV customers is profitable on a cohort basis.

**What "good" looks like here:**
- Consistent spend with stable (not declining) efficiency
- Strong thumbstop rate — stopping cold scrollers is the hardest job
- CPA below account average for this tier

**Mistake to avoid:** Calling a prospecting creative "underperforming" because its goal metric is below breakeven. Compare it to other prospecting creatives, not to the account average.

### Scaling / Proven Winners
**Common naming signals:** "Scaling", "CBO", "Top performers", "Evergreen", "Scale", "Winners"

**Efficiency expectations:** Should be at or above breakeven. These creatives have already been validated — they're here to generate returns at volume.

**What "good" looks like here:**
- Goal metric at or above breakeven with increasing or stable spend
- Consistent conversion volume (not just strong efficiency on low volume)

**Mistake to avoid:** Letting fatigued creatives linger in "Top performers" ad sets. If efficiency has declined below breakeven for 7+ days, it's no longer a top performer regardless of historical performance.

### Testing / Creative Experimentation
**Common naming signals:** "Testing", "Test", "Creative Test", "CT", "Experiment", numbered variants ("Testing 1", "Testing 2")

**Efficiency expectations:** Highly variable. The point is learning, not immediate returns. Some tests will fail — that's expected.

**What "good" looks like here:**
- Quick signal — strong early efficiency or thumbstop on low spend
- Clear differentiation from existing winners (testing should explore, not replicate)

**Mistake to avoid:** Judging test creatives against scaling benchmarks. A test creative at $78 spend with strong efficiency hasn't proven anything yet — but it's earned the right to more budget.

### Retargeting / BOFU / Warm Audiences
**Common naming signals:** "Retargeting", "BOF", "Warm", "Reengagement", "RT", "Remarketing"

**Efficiency expectations:** Strongest in the account. These audiences already know the brand — conversion should be efficient.

**What "good" looks like here:**
- Goal metric well above account average
- Strong purchase rate / low CPA

**Mistake to avoid:** Over-crediting retargeting creatives. Strong efficiency here is partly a function of warm audiences, not just creative quality. Don't use retargeting efficiency to justify scaling a creative into cold audiences.

---

## Ad Set Strategy Signals

### Budget Strategy (from ad set naming)
| Signal | Meaning | Efficiency Interpretation |
|---|---|---|
| "Cost Cap" / "CC" | Meta restricts spend to efficient impressions only | **Inflates efficiency.** The creative only gets cheap conversions. Efficiency will drop if moved to uncapped. |
| "Highest Volume" / "HV" | Meta spends freely to maximize conversions | **Honest signal.** No efficiency filter — what you see is what you get. |
| "Bid Cap" / "BC" | Hard ceiling on cost per result | Somewhere between cost cap and uncapped. More restrictive than highest volume. |

**Critical rule:** Never compare a cost-cap creative's efficiency to an uncapped creative's efficiency without flagging the difference. A cost-cap creative showing 8x the efficiency might only deliver 2x uncapped.

### Audience Strategy (from ad set naming)
| Signal | Meaning |
|---|---|
| "Broad" / "BRD" | No interest targeting — Meta finds the audience. Hardest test but most scalable if it works. |
| "LAL" / "Lookalike" | Modeled audiences. Medium difficulty. |
| Interest keywords (e.g., "Fitness", "Parents") | Interest-based targeting. Easiest to convert but smallest scale. |
| Age/gender segments (e.g., "30+", "F25-34") | Demographic targeting. |

### Ad Set Composition
| Pattern | What to Watch For |
|---|---|
| "Top performers" + "Testing" in same CBO | CBO allocates budget between proven and unproven creatives. Testing creatives get starved. **Flag this as a structural issue.** |
| Multiple "Testing 1/2/3/4" ad sets | Parallel testing — good structure if each has a distinct hypothesis. Bad if they're random. |
| Single ad set with many creatives | Meta's ad-level optimization picks winners. Works for testing, risky for scaling (one creative can dominate). |

---

## Structural Issues to Flag

### Testing Inside Scaling Campaigns
**Problem:** When "Testing" ad sets live inside a "Scaling" CBO campaign, the CBO allocates budget across all ad sets including tests. Proven ad sets with historical signal absorb most budget. New test creatives get $50-100 and never get a fair shot.

**Recommendation:** Move testing ad sets to a dedicated testing campaign. Only promote to Scaling once validated.

### Stale "Top Performers" Ad Sets
**Problem:** Ad sets labeled "Top performers" attract CBO budget based on historical signal. But if the creatives inside have fatigued, the ad set becomes a budget trap — absorbing spend at declining efficiency while starving newer, better creatives elsewhere.

**Recommendation:** Audit "Top performers" ad sets regularly. Remove any creative with the goal metric below breakeven for 7+ consecutive days. Replace with recently validated winners.

### Cost Cap Masking True Performance
**Problem:** Cost-cap ad sets only spend when conversions are cheap. This makes every creative inside look efficient — but it's the constraint doing the work, not the creative. When these creatives get promoted to uncapped campaigns, efficiency drops sharply.

**Recommendation:** Before declaring a cost-cap creative a winner, validate it in an uncapped "highest volume" testing environment. If it maintains meaningful efficiency uncapped, it's real.

### CBO Budget Concentration
**Problem:** Within a CBO campaign, Meta concentrates budget on one or two ad sets/creatives with the strongest historical signal. This creates a rich-get-richer dynamic where high-spend creatives keep getting budget even as they fatigue, while potentially better low-spend creatives never get enough impressions to prove out.

**Recommendation:** Monitor spend distribution across ad sets within each CBO. If one ad set absorbs >60% of campaign budget, check whether its efficiency justifies that concentration. If not, consider splitting into separate campaigns.

---

## Creative Graduation Path

The ideal lifecycle of a creative through campaign tiers:

```
Testing (uncapped) → $200-500 spend, signal check
    ↓ if goal metric > threshold
Scaling (CBO, uncapped) → $500-2000 spend, prove at volume  
    ↓ if goal metric holds
Top Performers → steady-state budget allocation
    ↓ if goal metric declines 7+ days
Pause or Refresh → new hooks on proven body, or retire
```

When recommending actions, specify where in this path each creative should move next.
