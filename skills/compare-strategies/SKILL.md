---
name: compare-strategies
description: >
  Side-by-side comparison of strategies or bots. Use when the user asks
  "compare strategies", "edge-trader vs sniper", "compare bots",
  "which strategy is better", or wants to see how different bots/strategies stack up.
argument-hint: "[bot1] [bot2] (or strategy names)"
---

# Strategy Comparison

Compare two or more bots/strategies side by side.

## Step 1: Identify bots to compare

Parse `$ARGUMENTS` for bot names, IDs, or strategy names.
- If specific bots given, compare those
- If strategy names given (e.g., "edge-trader vs last-minute-sniper"), find bots using each strategy
- If no arguments, call `list_bots` and ask the user to pick two or more

## Step 2: Gather data for each bot

For each bot:
1. **`get_bot_details`** — Details and config
2. **`get_performance_metrics`** — Performance metrics
3. **`query_positions`** — Position history
4. **`analyze_decisions`** — Decision patterns

You can also use **`compare_bots`** and **`compare_configs`** for direct comparisons.

## Step 3: Present side-by-side comparison

```
## Strategy Comparison
{Bot A name} ({strategy}) vs {Bot B name} ({strategy})

### Head-to-Head Metrics
| Metric | {Bot A} | {Bot B} | Winner |
|--------|---------|---------|--------|
| Total P&L | ${pnl} | ${pnl} | {name} |
| Win Rate | {%} | {%} | {name} |
| Profit Factor | {pf} | {pf} | {name} |
| Expectancy | ${exp} | ${exp} | {name} |
| Max Drawdown | ${dd} | ${dd} | {name} |
| Total Trades | {n} | {n} | -- |
| Avg Trade Duration | {time} | {time} | -- |
| Fill Rate | {%} | {%} | {name} |
| Avg Slippage | {c} | {c} | {name} |

**Overall Winner: {name}** (won {X}/{Y} metrics)

### Configuration Comparison
| Parameter | {Bot A} | {Bot B} |
|-----------|---------|---------|
{For shared config keys, compare values}

### Behavioral Differences
- **Trade Frequency**: {A} trades ~{X}/day vs {B} trades ~{Y}/day
- **Side Preference**: {A} favors {UP/DOWN/balanced}, {B} favors {UP/DOWN/balanced}
- **Risk Profile**: {A} is {conservative/aggressive}, {B} is {conservative/aggressive}

### Recommendation
{Based on the data, which strategy/config is performing better and why.
Note any caveats about sample size or time period.}
```

## Edge cases
- If comparing bots with very different trade counts, warn about sample size disparity
- If only one bot exists, suggest creating a second in test mode to compare
- If comparing live vs test mode bots, note the mode difference prominently
