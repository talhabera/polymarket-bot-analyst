---
name: risk-report
description: >
  Generate a risk analysis report showing exposure, drawdowns, kill switch status,
  and risk limit utilization. Use when the user asks "risk report", "risk analysis",
  "am I safe", "what's my exposure", "risk status", or wants to understand their risk posture.
---

# Risk Analysis Report

Generate a comprehensive risk report across all bots or for a specific bot.

## Step 1: Gather system-wide risk data

1. **`get_risk_state`** — Get global risk state (kill switch, daily P&L, exposure, config)
2. **`list_bots`** — Get all active bots

## Step 2: Gather per-bot data

For each running bot:
1. **`get_bot_details`** with the bot ID — Get bot details and mode
2. **`query_positions`** filtering for open positions — Calculate per-bot exposure
3. **`get_performance_metrics`** — Get drawdown and loss streaks

## Step 3: Generate risk dashboard

```
## Risk Analysis Report
**Generated: {timestamp}**

### System Status
| Indicator | Status | Detail |
|-----------|--------|--------|
| Kill Switch | {ON/OFF} | {reason if active} |
| Daily P&L | ${pnl} | {pnl/limit}% of limit used |
| Daily Trades | {count} | -- |
| Consecutive Losses | {n}/{max} | {severity} |
| Total Exposure | ${exposure} / ${max} | {%} utilized |

### Risk Limit Utilization
{ASCII bar chart showing each limit's utilization:}
Daily Loss:    [========........] 48% ($24/$50)
Exposure:      [======..........] 35% ($70/$200)
Consec Losses: [====............] 2/5

### Per-Bot Exposure
| Bot | Strategy | Mode | Open Positions | Exposure | Today P&L |
|-----|----------|------|----------------|----------|-----------|
| {name} | {strategy} | {live/test} | {count} | ${amount} | ${pnl} |

### Alerts
{List any conditions that warrant attention:}
- {CRITICAL}: Kill switch is active -- all trading halted
- {WARNING}: Daily P&L at 80% of loss limit
- {WARNING}: 3 consecutive losses -- approaching limit of 5
- {INFO}: Bot "Alpha" has 2 open positions in live mode

### Recommendations
1. {Most urgent action if any}
2. {Proactive suggestion}
```

## Edge cases
- If kill switch is active, make this the dominant finding with a prominent banner
- If no bots are running, report the risk state but note no active exposure
- If all bots are in test mode, note that no real capital is at risk
