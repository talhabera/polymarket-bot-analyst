---
name: debug-strategy
description: >
  Diagnose why a bot is underperforming or losing money. Use when the user asks
  "why is my bot losing", "debug strategy", "what's wrong", "diagnose",
  "why aren't trades profitable", or expresses frustration with bot performance.
argument-hint: "[bot-name-or-id]"
---

# Strategy Debugger

Diagnose underperformance by systematically checking each component of the trading pipeline.

## Step 1: Identify the bot

If `$ARGUMENTS` specifies a bot, use it. Otherwise, call `list_bots` and ask.

## Step 2: Gather diagnostic data

1. **`get_bot_details`** — Config and status
2. **`get_performance_metrics`** — Performance metrics
3. **`analyze_decisions`** — Recent decisions (last 50)
4. **`query_positions`** — Recent positions (last 50)
5. **`query_orders`** — Recent orders (last 50)
6. **`get_risk_state`** — Risk state
7. **`get_trade_analysis`** — Detailed trade analysis with slippage, fill rates, side analysis

## Step 3: Run diagnostic checks

Check each component in the trading pipeline, from signal to execution:

### Check 1: Signal Quality
- What % of BUY signals lead to profitable trades?
- Is there a pattern to bad signals? (time of day, edge size, side)
- Are signals too frequent (low conviction) or too rare (missing opportunities)?

### Check 2: Entry Execution
- Fill rate: What % of orders get filled?
- Slippage: How much worse is fillPrice vs intended price?
- Timing: Are entries happening at bad cycle times (slot 8-9)?

### Check 3: Position Management
- Average hold time: Appropriate for the market duration?
- TP hit rate: How often does take-profit execute vs holding to resolution?
- Are positions closing at worse prices than entry?

### Check 4: Side Balance
- Is one side (UP/DOWN) much worse than the other?
- Compare win rates: UP trades vs DOWN trades

### Check 5: Risk & Operations
- Is the kill switch tripping too often?
- Are consecutive loss limits being hit?
- Is the bot getting paused and missing opportunities?

### Check 6: Market Conditions
- Are spreads wider than usual? (makes fills harder)
- Has volatility changed? (affects fair value calculations)

## Step 4: Present diagnosis

```
## Diagnosis: {bot name}
**Status: {running/paused/stopped}** | **Recent P&L: ${pnl} over last {N} trades**

### Pipeline Health Check
| Component | Status | Issue |
|-----------|--------|-------|
| Signal Generation | {OK/WARN/FAIL} | {detail} |
| Order Execution | {OK/WARN/FAIL} | {detail} |
| Position Management | {OK/WARN/FAIL} | {detail} |
| Side Balance | {OK/WARN/FAIL} | {detail} |
| Risk Controls | {OK/WARN/FAIL} | {detail} |
| Market Conditions | {OK/WARN/FAIL} | {detail} |

### Root Cause
**Primary Issue: {one-line summary}**

{Detailed explanation with evidence from the data. For example:}
"The bot's edge threshold (3c) is too low for current market conditions. 68% of entries
at edges below 5c are losing trades. Historically, edges above 7c have a 62% win rate.
The Aggressive preset uses edgeBase: 3, but the Conservative preset's edgeBase: 8 would
filter out most of these losing trades."

### Evidence
- {Supporting data point 1}
- {Supporting data point 2}
- {Supporting data point 3}

### Fix Recommendation
1. **Immediate**: {Quick fix, e.g., "Raise edgeBase from 3 to 6"}
2. **Short-term**: {Config adjustment, e.g., "Switch to Conservative preset for live mode"}
3. **Monitor**: {What to watch, e.g., "Track win rate at the new threshold for 50 trades"}
```

## Next steps

After presenting the diagnosis, suggest relevant follow-up skills:
- If config changes are needed: "Run `/optimize-config` to get specific parameter recommendations"
- If signal quality is the root cause: "Run `/strategy-review` for deeper signal analysis"
- If risk controls are the issue: "Run `/risk-report` for a comprehensive risk assessment"

## Edge cases
- If the bot is actually profitable, note this and look for areas of improvement instead
- If the bot has < 10 trades, say there is not enough data to diagnose
- If multiple issues are found, rank them by impact (estimated P&L improvement)
