---
name: trade-journal
description: >
  Export and annotate trade history as a structured journal. Use when the user asks
  "trade journal", "export trades", "trade history", "trade log",
  or wants a record of trading activity with annotations.
argument-hint: "[bot-name-or-id] [period: 24h|7d|30d|all]"
---

# Trade Journal Export

Export and annotate a bot's trade history into a structured trade journal.

## Step 1: Parse arguments

- `$ARGUMENTS[0]`: Bot name or ID (required — ask if not provided)
- `$ARGUMENTS[1]`: Period — `24h`, `7d`, `30d`, or `all` (default: `7d`)

Call `list_bots` if no bot specified, then ask the user.

## Step 2: Gather complete trade data

1. **`get_bot_details`** — Bot details and config
2. **`query_positions`** — All positions in the period
3. **`query_orders`** — All orders in the period
4. **`analyze_decisions`** — All decisions in the period
5. **`get_performance_metrics`** — Summary metrics

## Step 3: Build the export

Call **`export_bot_data`** to get a structured JSON export with all sections.

If the export tool is not available, construct the journal manually from the gathered data.

## Step 4: Generate annotated journal

For each position (trade), create an annotated entry:

```
## Trade Journal: {bot name}
**Period: {start} to {end}** | **{count} trades**

### Summary
- Total P&L: ${pnl}
- Win Rate: {%} ({wins}W / {losses}L)
- Best Trade: +${best}
- Worst Trade: -${worst}

---

### Trade #{n} — {WIN/LOSS} ${pnl}
**{timestamp}** | **{side}** | **{simulated ? "SIM" : "LIVE"}**

| Field | Value |
|-------|-------|
| Entry Price | {entryPrice} |
| Exit Price | {exitPrice} |
| Size | ${size} |
| Hold Duration | {duration} |
| Slippage | {fillPrice - price}c |

**Decision Signal:** {decision.reason}
- Edge at entry: {edge}c (threshold: {threshold}c)
- BTC Price: ${btcPrice}
- Volatility: {vol}

**Annotation:** {AI-generated note about what made this trade notable —
e.g., "Entered on a 8.2c edge, well above threshold. Price moved favorably
within 30 seconds. Clean winner with no slippage."}

---
{Repeat for each trade}
```

## Step 5: Save if requested

If the user wants the export saved, write it to a file.
Suggest filename: `{botName}_journal_{period}_{date}.md`

## Edge cases
- If no trades in the period, report that and suggest a wider time range
- Cap at 100 trades per export to keep context manageable — offer to paginate
- For positions still open, mark them as "OPEN" instead of WIN/LOSS
