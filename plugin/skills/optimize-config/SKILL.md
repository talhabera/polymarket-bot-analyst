---
name: optimize-config
description: >
  Analyze bot configuration and suggest parameter optimizations. Use when the user asks
  "optimize config", "tune parameters", "improve strategy", "better settings",
  "should I change my config", or wants to tweak strategy parameters.
argument-hint: "[bot-name-or-id]"
---

# Configuration Optimizer

Analyze a bot's current configuration against its performance data and suggest optimizations.

## Step 1: Identify the bot

If `$ARGUMENTS` specifies a bot, use it. Otherwise, call `list_bots` and ask the user.

## Step 2: Gather data

1. **`get_bot_details`** — Get current config and strategy type
2. **`get_performance_metrics`** — Get performance metrics
3. **`analyze_decisions`** — Get recent decisions (last 50) to analyze signal patterns
4. **`query_positions`** — Get recent positions (last 50) for outcome analysis

## Step 3: Read the strategy source to understand parameters

Read the strategy file to understand what each config parameter does:
- For edge-trader: `packages/bot-engine/src/strategies/edge-trader/strategy.ts`
- For last-minute-sniper: `packages/bot-engine/src/strategies/last-minute-sniper/strategy.ts`

Also read `packages/shared/src/strategy-metadata.ts` for preset definitions.

## Step 4: Analyze each parameter

For each config parameter, assess:

**Edge Trader parameters:**
- `edgeBase` / `edgeMultiplier`: Are entries too aggressive (low edge) or too conservative (missing trades)?
  - Check: What % of BUY decisions were profitable? If < 45%, edge threshold may be too low.
  - Check: What % of WAIT decisions had edge close to threshold? If many, threshold may be too high.
- `tpBase` / `tpMultiplier`: Are take-profits being reached or is the bot holding to resolution?
  - Check positions: How many closed via TP vs market resolution?
- `tradeSize`: Is sizing appropriate for the bankroll and risk limits?
  - Check against risk config maxOrderSize and maxTotalExposure
- `candleCount` / `volDamping`: Is volatility estimate appropriate?
- `cooldownSeconds`: Are early-cycle entries performing worse than later ones?

**Last Minute Sniper parameters:**
- `entryWindowSeconds`: Does narrowing/widening the window change win rate?
- `spreadThreshold`: Are good opportunities being filtered out?
- `minFairValue`: Is the floor appropriate?

## Step 5: Compare with presets

Compare the current config against the strategy presets from strategy-metadata.ts.
Note which preset is closest and what the differences imply.

## Step 6: Present recommendations

```
## Config Optimization Report: {bot name}

### Current Config vs Suggested Changes

| Parameter | Current | Suggested | Rationale |
|-----------|---------|-----------|-----------|
| edgeBase | {val} | {new} | {why} |
| ... | ... | ... | ... |

### Closest Preset: {preset name}
{How current config compares to the preset and whether switching makes sense}

### Analysis Detail
{For each parameter change, explain the evidence:}
- **edgeBase {current} -> {suggested}**: {X}% of BUY decisions at edges below {threshold} were losers. Raising the base should filter these out. Expected impact: fewer trades but higher win rate.

### What Would Change (estimated)
- Trade frequency: {higher/lower} by ~{X}%
- Win rate: likely {increase/decrease} by ~{X}%
- Average P&L per trade: likely {increase/decrease}

### Caution
These are estimates based on historical data. Consider testing changes in simulation mode first.
```

## Edge cases
- If fewer than 20 trades, warn that the sample size is too small for reliable optimization
- If the bot is already performing well (profit factor > 1.5, win rate > 55%), note it's in good shape and suggest only minor tweaks
- Always recommend testing in simulation mode before switching to live
