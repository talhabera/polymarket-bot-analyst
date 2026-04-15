---
name: daily-report
description: >
  Generate a daily trading report — morning briefing or end-of-day summary.
  Use when the user asks "daily report", "morning briefing", "how did today go",
  "today's summary", "daily P&L", or wants a digest of trading activity.
argument-hint: "[date: YYYY-MM-DD, default today]"
---

# Daily Trading Report

Generate a daily digest of trading activity across all bots.

## Step 1: Determine the date

If `$ARGUMENTS` contains a date, use it. Otherwise, use today's date.

## Step 2: Gather data for all bots

1. **`list_bots`** — Get all bots
2. For each bot:
   - **`get_performance_metrics`** — Get performance metrics
   - **`get_daily_pnl`** — Get daily P&L breakdown for the target date
   - **`query_positions`** — Get positions opened or closed on the target date
   - **`query_orders`** — Get orders from the target date
3. **`get_risk_state`** — Get risk state for the day

## Step 3: Generate daily digest

```
## Daily Trading Report: {date}
{Day of week} | {time of generation}

### P&L Summary
| Bot | Strategy | Mode | Trades | Wins | Losses | P&L |
|-----|----------|------|--------|------|--------|-----|
| {name} | {strategy} | {mode} | {n} | {w} | {l} | ${pnl} |
| **TOTAL** | | | **{n}** | **{w}** | **{l}** | **${total}** |

### Highlights
- Best trade: {bot} {side} trade at {time}, +${pnl}
- Worst trade: {bot} {side} trade at {time}, -${pnl}
- Total trades: {count} across {bot_count} bots
- Overall win rate: {%}

### Risk Status
- Daily loss limit: ${used} / ${limit} ({%} consumed)
- Kill switch activations: {count, usually 0}
- Consecutive loss peak: {max during day}

### Notable Events
{List anything unusual:}
- {Time}: Kill switch activated due to {reason}
- {Time}: Bot "{name}" paused after {n} consecutive losses
- {Time}: Unusual spread widening detected

### Trend (last 7 days)
| Date | Trades | P&L | Cumulative |
|------|--------|-----|------------|
| {7 days of history} |
```

## Edge cases
- If no trades occurred, report "No trading activity" with bot status summary
- If the date is in the future, report an error
- If some bots were idle all day, still include them in the table with 0 trades
