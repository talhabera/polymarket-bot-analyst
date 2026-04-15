---
name: analyze-bot
description: >
  Comprehensive bot health check and performance analysis. Use when the user asks
  "how is my bot doing", "bot health check", "analyze bot", "bot status",
  or wants a quick overview of a bot's performance and health.
argument-hint: "[bot-name-or-id]"
---

# Bot Health Check

Perform a comprehensive health check on a trading bot. Follow these steps exactly.

## Step 1: Identify the bot

If `$ARGUMENTS` specifies a bot name or ID, use it. Otherwise, call the MCP tool
`list_bots` to show all bots, then ask the user which one to analyze.

## Step 2: Gather data (call these MCP tools)

1. **`get_bot_details`** — Get bot details (name, strategy, status, mode, config, P&L, strategy descriptor)
2. **`get_performance_metrics`** — Get performance metrics (win rate, profit factor, drawdown, expectancy)
3. **`query_positions`** — Get recent positions (last 20, both open and closed)
4. **`get_risk_state`** — Get current risk state (kill switch, daily P&L, exposure, consecutive losses)
5. **`analyze_decisions`** — Get recent decisions (last 20) to assess signal quality

## Step 3: Compute a health score (0-100)

Calculate a composite score from these weighted factors:

| Factor | Weight | Scoring |
|--------|--------|---------|
| Profitability | 25% | 100 if P&L > 0, scale down to 0 at -$50 |
| Win Rate | 20% | 100 if >= 60%, scale linearly from 40% (0) to 60% (100) |
| Profit Factor | 15% | 100 if >= 1.5, 50 if 1.0, 0 if < 0.8 |
| Risk Health | 20% | Start at 100, subtract: -30 if kill switch active, -20 per consecutive loss over 3, -20 if daily loss > 50% of limit |
| Activity | 10% | 100 if bot is running and traded in last hour, 50 if running but no recent trades, 0 if idle/paused |
| Signal Quality | 10% | % of BUY decisions that resulted in profitable positions |

## Step 4: Generate report

Present findings in this format:

```
## Bot Health Report: {name}
**Health Score: {score}/100** {emoji based on score: >=80 excellent, >=60 good, >=40 needs attention, <40 critical}

### Key Metrics
| Metric | Value | Assessment |
|--------|-------|------------|
| Status | {status} | — |
| Total P&L | ${pnl} | {good/bad} |
| Win Rate | {rate}% | {above/below 50%} |
| Profit Factor | {pf} | {above/below 1.0} |
| Max Drawdown | ${dd} | {severity} |
| Open Positions | {count} | — |

### Risk Status
- Kill Switch: {active/inactive}
- Daily P&L: ${daily} / -${limit} limit
- Consecutive Losses: {n} / {max}
- Exposure: ${exposure} / ${max}

### Findings
1. {Most important finding}
2. {Second finding}
3. {Third finding}

### Recommendations
1. {Most impactful recommendation}
2. {Second recommendation}
```

## Next steps

After presenting the report, suggest relevant follow-up skills:
- If health score < 60 or specific problems found: "Run `/debug-strategy` for a detailed pipeline diagnosis"
- If signal quality is low: "Run `/strategy-review` for a deep signal quality analysis"
- If config seems suboptimal: "Run `/optimize-config` for parameter tuning recommendations"

## Edge cases
- If the bot has no trades yet, report "Insufficient data" and recommend waiting for at least 10 trades
- If the bot is stopped/paused, note this prominently and check if there are open positions that need attention
- If the kill switch is active, make this the primary finding with urgency
