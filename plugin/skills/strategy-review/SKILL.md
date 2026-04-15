---
name: strategy-review
description: >
  Deep strategy performance review with signal quality analysis and decision accuracy.
  This is a deep-dive complement to analyze-bot's quick health check.
  Use when the user asks "strategy review", "how is edge-trader doing", "review my strategy",
  "signal quality", or wants to understand how well their strategy is performing.
argument-hint: "[bot-name-or-id]"
---

# Strategy Performance Review

Conduct a deep review of a bot's strategy performance, focusing on signal quality and decision accuracy.

## Step 1: Identify the bot

If `$ARGUMENTS` specifies a bot, use it. Otherwise, call `list_bots` and ask.

## Step 2: Gather data

1. **`get_bot_details`** — Get bot details and strategy type
2. **`get_performance_metrics`** — Get performance metrics
3. **`analyze_decisions`** — Get decisions (last 100) for signal analysis
4. **`query_positions`** — Get positions (last 100) for outcome correlation
5. **`query_orders`** — Get orders (last 100) for fill rate and slippage

## Step 3: Understand strategy logic

Use `get_strategy_info` MCP tool for metadata and presets. Only read source if you need to understand the signal generation algorithm internally:
- For edge-trader: `packages/bot-engine/src/strategies/edge-trader/strategy.ts`
- For last-minute-sniper: `packages/bot-engine/src/strategies/last-minute-sniper/strategy.ts`

## Step 4: Analyze

Compute these strategy-specific metrics:

**Decision Accuracy:**
- Match decisions with `action` containing "buy" to the positions they created (via orderId)
- Calculate: % of BUY decisions that led to profitable positions

**Signal Distribution:**
- Count decisions by action type (buy_up, buy_down, wait, no_trade_zone)
- Show distribution over time — are signals changing?

**Edge Analysis (edge-trader):**
- From decision signals JSONB: extract `edgeUp`, `edgeDown`, `edgeThreshold`
- Bin trades by edge-at-entry: how does win rate change with edge size?
- "When edge > X cents, win rate = Y%" — find the sweet spot

**Side Analysis:**
- Compare UP vs DOWN trade performance (win rate, avg P&L)
- Is the bot better at one side?

**Fill Rate & Slippage:**
- Orders placed vs filled
- Average slippage: abs(fillPrice - price) from orders

**Timing Analysis:**
- From signals, extract slotIndex — which cycle slots produce the best entries?

## Step 5: Present strategy score card

```
## Strategy Review: {bot name} ({strategy type})
**Review Period: {date range}** | **Trades Analyzed: {count}**

### Strategy Score Card
| Dimension | Score | Detail |
|-----------|-------|--------|
| Decision Accuracy | {%} | {good/needs work} |
| Win Rate | {%} | -- |
| Profit Factor | {pf} | -- |
| Signal Quality | {rating}/10 | Based on edge-at-entry correlation |
| Fill Rate | {%} | {count} filled / {count} placed |
| Side Balance | {balanced/skewed} | UP: {%} win, DOWN: {%} win |

### Signal Distribution
{Bar chart of decision action types}
BUY_UP:     ============ 34%
BUY_DOWN:   ==========   28%
WAIT:       ================ 35%
NO_TRADE:   == 3%

### Edge-at-Entry Analysis
| Edge Range | Trades | Win Rate | Avg P&L |
|------------|--------|----------|---------|
| 3-5c | {n} | {%} | ${avg} |
| 5-8c | {n} | {%} | ${avg} |
| 8-12c | {n} | {%} | ${avg} |
| 12c+ | {n} | {%} | ${avg} |

### Slot Timing Analysis
| Slot (0-9) | Entries | Win Rate | Note |
|------------|---------|----------|------|
{For each slot with entries}

### Key Findings
1. {Primary insight about signal quality}
2. {Insight about timing or side performance}
3. {Insight about fill rate or slippage}

### Recommendations
1. {Actionable recommendation based on findings}
2. {Second recommendation}
```

## Edge cases
- If fewer than 30 trades, warn about small sample size
- If decisions JSONB signals are missing fields, note which analyses could not be performed
- For last-minute-sniper, adapt the edge analysis to use spread and fair value instead
