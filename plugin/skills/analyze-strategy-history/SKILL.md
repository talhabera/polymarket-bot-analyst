---
name: analyze-strategy-history
description: >
  This skill should be used when the user asks to "analyze strategy history",
  "what opportunities did my bot miss", "money left on the table",
  "missed opportunities", "strategy history", or "what trades should my bot
  have taken". Quantifies opportunity cost caused by config parameters and
  attributes each miss to the specific parameter that caused it.
argument-hint: "[bot-name-or-id]"
---

# Analyze Strategy History — Missed Opportunities

Quantify the money left on the table by a bot's config choices. Complements
`debug-strategy` (which looks at why actual trades lost) by looking at why
profitable trades didn't happen.

## Step 1: Identify the bot

If `$ARGUMENTS` specifies a bot, use it. Otherwise, call `list_bots` and ask
the user which bot to analyze.

## Step 2: Pull missed-opportunity data

Call `analyze_missed_opportunities` with defaults:

- `botId`: the bot
- `period`: `7d` (override if the user specifies a window)
- `mode`: `live` (override if the user wants to include simulated trades)
- `peerScope`: `same-strategy-same-asset`
- `obviousWinThreshold`: `0.85`

Also call `get_performance_metrics` for the same bot and period to get
realized P&L for the ratio.

## Step 3: Understand the miss taxonomy

The tool returns six buckets. Two are **latent** (return 0 on today's data);
four are **active** on real data:

**Active (derived from candles/orders/positions — reflect real behavior):**
- `idleCycles` — cycles where the bot never emitted a buy/open action but the
  cycle resolved obvious. Attributed to the strategy's gate params.
- `earlyExits` — positions closed via TP below the hold-to-resolution price.
  Attributed to `tpBase`.
- `lateEntries` — filled BUYs placed in the final ~100s of a cycle where the
  first candle's open was cheaper. Attributed to `entryWindowSecs`.
- `peerWins` — cycles this bot didn't trade but a peer bot closed a winning
  position.

**Latent (require WAIT decisions + filter reasons to be persisted — zero today):**
- `waitRegret` — cycles the strategy explicitly held through an obvious win.
- `filterRejections.spread_too_wide` — cycles rejected by the spread filter.
- `filterRejections.fair_below_min` — cycles rejected by the min-fair-value
  filter.

`summary.notice` will mention this. Do not treat a 0 in a latent bucket as
"nothing to tune" — state clearly that the bucket is data-limited.

## Step 4: Rank and present

Sort the active buckets by `wouldBePnl` descending. Present only buckets with
`count > 0`.

```
## Strategy History Analysis: {botName}

**Period:** {period} | **Cycles analyzed:** {totalCyclesAnalyzed} | **Traded:** {tradedCycles} ({pct}%)

### Headline
- **Realized P&L:** ${realized from get_performance_metrics.totalPnl}
- **Money left on table:** ${summary.wouldBePnlLeftOnTable}
- **Ratio:** {wouldBePnl / max(realized, 0.01), 1 decimal}x — {interpretation sentence}
- **Obvious misses:** {summary.obviousMissesCount} cycles

### Active miss buckets (ranked by $ impact)

| Rank | Miss Type | Count | $ Left | Attributed Param | Counterfactual |
|---|---|---|---|---|---|
| 1 | {type} | {count} | ${wouldBePnl} | {attributedParam}={currentValue} | {counterfactual.suggestedValue → +${counterfactual.additionalPnl} or "—"} |

### Top config issues

{For each of the top 3 buckets that have a counterfactual:}

**{attributedParam}** (current: {currentValue})
- Misses: {count} cycles = ${wouldBePnl} left on the table
- If {attributedParam} were **{suggestedValue}**, you would have caught
  {additionalHits} more cycles worth ${additionalPnl}.
- What this param does: {1 sentence from strategy knowledge}

### Peer wins (if count > 0)
- {peerWins.count} peer-bot wins on cycles this bot didn't trade, worth ${peerWins.wouldBePnl}.
- Peers: {peerBotIds.join(", ")}

### Data-coverage caveats
- Wait-regret and filter-rejection buckets are 0 because the decisions table
  does not persist WAIT rows today. Once those are instrumented, those buckets
  will become actionable.

### Recommended follow-up
- If counterfactuals look promising: run `/optimize-config {botId}` for
  specific tuning recommendations.
- If idle cycles dominate: compare gate params to peer bots via
  `/compare-strategies` and check risk controls via `/risk-report`.
- If early exits dominate: inspect `tpBase`/`tpMultiplier` — TP may be firing
  too early.
- If late entries dominate: tighten `entryWindowSecs` so entries happen when
  the first candle's open is still cheap.
- If peer wins dominate: `/compare-strategies` to understand what peers are
  doing differently.
```

## Step 5: Handle edge cases

- `summary.notice === "insufficient data"`: "Not enough data to analyze (fewer
  than 10 cycles in this period). Come back after the bot has run longer."
- `summary.wouldBePnlLeftOnTable` under $1 and realized P&L positive:
  congratulate the user — the config is well-tuned for this period.
- All counterfactuals `null`: explain that the sweep found no looser value
  that would have improved outcomes within the scan range.
- All active buckets are 0 but latent buckets would need persistence: say so
  explicitly instead of claiming the config is perfect.

## Avoiding double-counting with debug-strategy

`debug-strategy` analyzes trades that happened (actual wins/losses).
`analyze-strategy-history` analyzes trades that didn't happen (opportunity
cost). When both are run, treat their findings as complementary — never sum
P&L figures across the two.

## When to suggest this skill

Invite the user to run this skill after `debug-strategy` if the bot is
profitable but the user wants to know whether it could be doing better, or if
the bot has low trade frequency.
