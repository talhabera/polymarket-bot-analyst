---
name: market-analysis
description: >
  Assess current market conditions and trading environment. Use when the user asks
  "market analysis", "how's the market", "should I trade", "market conditions",
  "spread analysis", or wants to understand the current trading environment.
---

# Market Conditions Analysis

Assess the current state of the Polymarket BTC 5-minute binary options market.

## Step 1: Gather market data

1. **`list_bots`** — Find active bots to get market context
2. **`get_bot_details`** for an active bot — Get market ID and recent strategy state
3. **`get_market_conditions`** — Get current market snapshot (spreads, volatility, BTC price)
4. **`get_decision_signals`** — Recent decision signals to extract market metrics from JSONB
5. **`query_positions`** — Recent positions to assess execution quality

## Step 2: Extract market metrics from decision signals

From the decisions' `signals` JSONB, extract:
- `btcPrice` — Recent BTC price and trend direction
- `volatility` — Current volatility level
- `fairUp` / `fairDown` — Fair value estimates
- `marketUp` / `marketDown` — Actual market prices
- `spreadUp` / `spreadDown` — Spread widths (for sniper)
- `edgeUp` / `edgeDown` — Available edges

## Step 3: Analyze conditions

Assess:
- **Spread health**: Are spreads narrow enough for profitable trading?
- **Volatility level**: Is volatility in a normal range or extreme?
- **Edge availability**: Are there consistent edges or is the market efficient?
- **Fill environment**: Are orders getting filled at reasonable prices?

## Step 4: Present analysis

```
## Market Conditions Report
**Timestamp: {now}** | **Market: BTC 5-min Binary Options**

### Current Snapshot
| Metric | Value | Assessment |
|--------|-------|------------|
| BTC Price | ${price} | {trending up/down/sideways} |
| Volatility (sigma) | {vol} | {low/normal/high/extreme} |
| Avg Spread (UP) | {spread} | {tight/normal/wide} |
| Avg Spread (DOWN) | {spread} | {tight/normal/wide} |
| Avg Edge Available | {edge}c | {good/marginal/poor} |
| Recent Fill Rate | {%} | {good/concerning} |

### Trading Conditions
**Overall: {FAVORABLE / NEUTRAL / UNFAVORABLE}**

{Explanation of current conditions and what they mean for trading:}
- Volatility is {level}, which means {implication for fair value accuracy}
- Spreads are {level}, which means {implication for fill quality}
- Edges are {level}, suggesting {market efficiency assessment}

### Recommendations
- **For edge-trader**: {Specific advice — e.g., "Widen edge threshold in low-vol"}
- **For last-minute-sniper**: {Specific advice}
- **General**: {Overall trading recommendation}
```

## Edge cases
- If no bots are running, explain that market data requires an active bot for context
- If decision signals are empty or missing, note which analyses could not be performed
- Note that this is a snapshot, not a prediction — conditions change rapidly
