---
name: compare-variants
description: >
  Rank the variants in a running experiment group against the source bot. Use when the
  user asks "how is exp-XXXX doing", "compare variants", "rank my experiment",
  "which variant is winning", or wants mid-experiment progress on a forward-test sweep.
argument-hint: "[experiment-group-id or source-bot-id]"
---

# Compare Variants — Rank an Experiment Group

Pull performance metrics for every variant in an experiment group, rank them, and flag early leaders/laggards against the source bot.

## Step 1: Identify the experiment group

Parse `$ARGUMENTS`:
- If it matches `^exp-[0-9a-f]{4}$` → it's an `experimentGroupId`, use directly.
- If it's a UUID → treat as a `sourceBotId`, find all variants with that `experimentSourceBotId`.
- If neither and the user mentioned a bot name, resolve via `list_bots` and pick the group containing the most recent variants for that bot.
- If unresolved, call `list_bots` and show the user the active experiment groups (any bots with `experimentGroupId != null` and `isArchived=false`).

## Step 2: Gather the variant set

Call `list_bots` and filter in memory for:
- `experimentGroupId === <groupId>` AND `isArchived === false` → these are the variants
- Plus the source bot (from any variant's `experimentSourceBotId`)

Record: `sourceBotId`, `sourceBotName`, and an array of `{variantId, variantName, label, overrides, status}`.

## Step 3: Collect per-bot performance

For the source bot AND each variant (parallel where possible):

1. `get_bot_details` — current config, status, mode
2. `get_performance_metrics` — win rate, profit factor, drawdown, expectancy, total PnL, trade count
3. `query_positions` with `limit: 100` — recent position outcomes
4. `get_trade_analysis` — fill rate, slippage, side analysis

Do NOT call `analyze_decisions` for every variant unless the user asks for deep signal analysis — it's expensive and usually not needed for the ranking.

## Step 4: Check sample size

Count filled positions per variant.
- If any variant has `< 30 trades`, mark its row with a `(small sample)` flag and de-weight the ranking for that row.
- If ALL variants have `< 30 trades`, show the data but state clearly that a conclusion is premature.

## Step 5: Rank

Primary ranking metric: **profit factor** (if all variants have `>= 30 trades`) or **expectancy** (if mixed sample sizes — expectancy handles low-sample-size better).

Use total PnL only as a secondary tiebreaker — it's biased toward variants that have been trading longer.

Mark the source bot as a benchmark row (not ranked), and the variant closest to the source config as the "paper control" baseline.

## Step 6: Present the ranking

```
## Experiment {experimentGroupId} — Variant Ranking

**Source:** {sourceBotName} ({strategy}) — live since {createdAt}
**Variants:** {N} running | **Paper control:** {variant id at current config}
**Trade counts range:** {minTrades}..{maxTrades}

### Ranking (by profit factor)

| Rank | Label | Overrides | Trades | Win% | PF | Expectancy | PnL | Notes |
|------|-------|-----------|--------|------|----|------------|-----|-------|
| -- | [SOURCE] {sourceBotName} | (live) | {n} | {%} | {pf} | ${exp} | ${pnl} | baseline |
| 1 | edgeMult=2.00 | edgeMultiplier: 2.00 | {n} | {%} | {pf} | ${exp} | ${pnl} | leader |
| 2 | edgeMult=1.75 | edgeMultiplier: 1.75 | {n} | {%} | {pf} | ${exp} | ${pnl} | |
| 3 | edgeMult=1.50 (paper control) | edgeMultiplier: 1.50 | {n} | {%} | {pf} | ${exp} | ${pnl} | matches source config |
| 4 | edgeMult=1.25 | edgeMultiplier: 1.25 | {n} | {%} | {pf} | ${exp} | ${pnl} | |
| 5 | edgeMult=1.00 | edgeMultiplier: 1.00 | {n} | {%} | {pf} | ${exp} | ${pnl} | laggard, small sample |

### Behavior Differences vs Source
- **Trade frequency**: variants are placing {X}% {more/fewer} trades per hour
- **Side balance**: leader favors {UP/DOWN/balanced}; laggard favors {UP/DOWN/balanced}
- **Fill rate / slippage** (paper vs live): {caveat note}

### Early Read
{1–2 sentence summary — which variant is leading, by how much, and whether the sample is big enough to trust it.}

### Caveats
- Sample sizes: {any variants with < 30 trades listed here}
- Paper-mode fills are optimistic vs live slippage — the source bot's slippage is the real benchmark.
- Rank can shift — check back after {X} more hours for more reliable comparison.
```

## Step 7: Offer next actions

Based on the ranking, suggest:
- **If clear leader exists with sufficient sample**: "Consider concluding — run `conclude-experiment {groupId}` and keep the leader."
- **If no clear leader yet**: "Check back in a few hours; variants need more data."
- **If paper control is losing badly** (suggesting a data/config issue, not strategy quality): "The paper control matches the live config but is underperforming the live source — investigate paper-mode fill simulation before trusting the sweep."

## Edge cases

- **Group has zero running variants**: all were stopped or archived. Tell the user and suggest `list_bots` to find the correct group.
- **Source bot archived or deleted**: still rank variants against each other, but drop the baseline row and note the missing source.
- **Single variant**: no ranking needed, just show metrics vs source.
- **All variants have identical configs** (user spammed the same override): show the data, flag the redundancy.
- **Mode mismatch**: if the source bot is itself in `mode=test`, note this — variants vs a test-mode source is a pure simulation-on-simulation comparison, so conclusions apply only to paper.
