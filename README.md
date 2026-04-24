# Polymarket Analyst

A Claude Code plugin for analyzing and managing Polymarket 5-minute BTC binary options trading bots. Provides 12 specialized AI agents, 13 executable skills, and MCP-powered workflows covering bot health checks, strategy debugging, risk management, performance analysis, configuration optimization, per-trade forensics, fleet observability, write-path operations, and forward-test experiments.

## Table of Contents

- [Installation](#installation)
- [MCP Server](#mcp-server)
- [Agents](#agents)
- [Skills](#skills)
- [Project Structure](#project-structure)

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/talhabera/polymarket-analyst.git
   ```

2. Copy the environment file and add your API key:
   ```bash
   cp plugin/.env.example plugin/.env
   ```

3. Install as a Claude Code plugin:
   ```bash
   claude plugin add /path/to/polymarket-analyst
   ```

## MCP Server

The plugin connects to the **polymarket-trading-bot** MCP server, which provides live trading data through an HTTP endpoint.

**Configuration:** `plugin/.mcp.json`

### Available MCP Tools

| Category | Tools |
|----------|-------|
| **Bot Management** | `list_bots`, `get_bot_details`, `get_bot_status`, `get_bot_snapshot` |
| **Performance** | `get_performance_metrics`, `get_daily_pnl`, `get_pnl_history`, `get_trade_analysis` |
| **Trading Data** | `query_positions`, `query_orders`, `query_logs`, `explain_trade`, `get_cycle_detail` |
| **Strategy** | `get_decision_signals`, `analyze_decisions`, `analyze_missed_opportunities`, `get_strategy_info`, `analyze_config`, `get_skip_reason_histogram`, `get_edge_trader_slot_table`, `get_sniper_cycle_breakdown` |
| **Risk** | `get_risk_state`, `get_risk_report`, `toggle_kill_switch` |
| **Market** | `get_market_conditions`, `get_candle_data`, `get_orderbook_depth`, `get_historical_price_at`, `get_polymarket_event`, `resolve_market` |
| **Fleet & Infra** | `get_fleet_stats`, `get_data_availability`, `get_rtds_health` |
| **Comparison** | `compare_bots`, `compare_configs`, `export_bot_data`, `export_pine_script` |
| **Experiments** | `spawn_variants`, `get_experiment_status`, `stop_experiment`, `archive_variant`, `promote_variant` |
| **Operations** | `update_bot_config`, `bulk_bot_action` |

All 44 MCP tools exposed by the `polymarket-trading-bot` server are covered by at least one agent.

## Agents

Twelve specialized AI agents provide different analytical perspectives. All agents inherit the parent Claude model and have access to MCP tools.

### Trading Analyst (Blue)

Senior analyst for cross-cutting investigations spanning performance, strategy, and risk. Has access to all 9 skills. Use for multi-bot pattern analysis, pre-live readiness assessments, and broad "what's wrong" investigations.

### Performance Analyst (Cyan)

Bot health checks, trade tracking, daily reports, and strategy performance reviews. Tracks key metrics like profit factor, win rate, expectancy, max drawdown, fill rate, and slippage.

**Skills:** `analyze-bot`, `daily-report`, `strategy-review`, `trade-journal`

### Strategy Engineer (Yellow)

Strategy debugging, configuration optimization, and strategy comparison. Troubleshoots underperformance by walking through the trading pipeline: Signal Generation → Order Execution → Position Management → Side Balance → Risk Controls → Market Conditions.

**Skills:** `debug-strategy`, `optimize-config`, `compare-strategies`

### Risk Manager (Red)

Monitors risk exposure, enforces limits, and assesses market conditions for safety. Tracks kill switch status, daily loss limits, max exposure, consecutive losses, and slippage.

**Skills:** `risk-report`, `market-analysis`

### Edge Trader Specialist (Green)

Deep expertise in the edge-trader strategy: signal generation, edge thresholds, volatility model, fair value calculation, and take-profit mechanics. Understands time slot thresholds (0-9, 30s each), edge formulas, and GBM-based fair value using Chainlink BTC/USD feed.

**Skills:** `debug-strategy`, `optimize-config`, `strategy-review`

### Sniper Specialist (Orange)

Deep expertise in the last-minute-sniper strategy: entry window timing, band sizing, spread filtering, and fair value near expiry. Understands the signal pipeline (window checks → spread filter → fair value floor → band table lookup) and the 10-tier risk/reward band table.

**Skills:** `debug-strategy`, `optimize-config`, `strategy-review`

### Market Resolver (Purple)

Validates bot decisions against actual Polymarket Gamma API market resolutions. Verifies win rates, backtests signal accuracy, and cross-references decisions with on-chain outcomes. Uses `resolve_market`, `get_polymarket_event`, and `get_historical_price_at` for precise outcome cross-referencing.

**Skills:** `analyze-bot`, `strategy-review`, `debug-strategy`

### Experiment Runner (Magenta)

Designs, launches, monitors, and concludes forward-test experiments. Spawns paper-trade variants of a live bot with alternate configs, tracks them mid-flight via `get_experiment_status`, promotes winners via `promote_variant`, and archives losers.

**Skills:** `run-experiment`, `compare-variants`, `conclude-experiment`

### Trade Forensics (Teal)

Per-trade and per-cycle investigation specialist. Reconstructs the exact state around a single trade — decision reasoning (`explain_trade`), cycle detail (`get_cycle_detail`), orderbook snapshot (`get_orderbook_depth`), historical BTC price (`get_historical_price_at`), and bot-state point-in-time (`get_bot_snapshot`). Also does skip-reason histogram analysis for "why wasn't the bot trading?" windows.

**Skills:** `trade-journal`, `debug-strategy`

### Fleet Monitor (Blue)

Fleet-wide observability and data-infrastructure health. Monitors aggregate bot stats (`get_fleet_stats`), historical data coverage (`get_data_availability`), and the real-time data service (`get_rtds_health`). First-line check before trusting any downstream analysis — other agents' conclusions are only as good as the data this agent verifies is flowing.

**Skills:** none (read-only observability)

### Bot Operator (Gray)

Write-path agent that executes configuration changes, bulk fleet actions, experiment promotions, Pine Script exports, and kill-switch toggles. The only agent that mutates state — runs a diff/confirm pre-flight on every write. Handles `update_bot_config`, `bulk_bot_action`, `promote_variant`, `export_pine_script`, and `toggle_kill_switch`.

**Skills:** none (write-path operator)

## Skills

Thirteen executable skills provide structured workflows invoked via slash commands.

| Skill | Command | Description |
|-------|---------|-------------|
| **Analyze Bot** | `/analyze-bot [bot]` | Comprehensive bot health check with a 0-100 health score based on profitability, win rate, profit factor, risk health, activity, and signal quality |
| **Daily Report** | `/daily-report [date]` | Daily trading digest with P&L summary per bot, best/worst trades, risk status, and 7-day trend |
| **Strategy Review** | `/strategy-review [bot]` | Deep strategy performance scorecard covering decision accuracy, signal distribution, edge-at-entry, side balance, fill rate, and slot timing |
| **Debug Strategy** | `/debug-strategy [bot]` | Diagnose underperformance with a 6-point pipeline check, root cause analysis, and fix recommendations |
| **Optimize Config** | `/optimize-config [bot]` | Parameter optimization recommendations with current vs. suggested values, rationale, and estimated impact |
| **Compare Strategies** | `/compare-strategies [bot1] [bot2]` | Side-by-side comparison of two bots or strategies with head-to-head metrics and a recommendation |
| **Risk Report** | `/risk-report` | System-wide risk dashboard with status, limit utilization, per-bot exposure, and alerts |
| **Market Analysis** | `/market-analysis` | Current market conditions assessment with spreads, volatility, edges, and strategy-specific recommendations |
| **Trade Journal** | `/trade-journal [bot] [period]` | Export and annotate trade history with entry/exit prices, slippage, decision signals, and notes |

## Project Structure

```
polymarket-analyst/
├── .claude-plugin/
│   ├── plugin.json              # Root plugin metadata
│   └── marketplace.json         # Marketplace configuration
├── plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json          # Plugin-level metadata
│   ├── .mcp.json                # MCP server configuration
│   ├── .env.example             # Environment template
│   ├── agents/
│   │   ├── trading-analyst.md
│   │   ├── performance-analyst.md
│   │   ├── strategy-engineer.md
│   │   ├── risk-manager.md
│   │   ├── edge-trader-specialist.md
│   │   ├── sniper-specialist.md
│   │   ├── market-resolver.md
│   │   ├── experiment-runner.md
│   │   ├── trade-forensics.md
│   │   ├── fleet-monitor.md
│   │   └── bot-operator.md
│   └── skills/
│       ├── analyze-bot/
│       ├── compare-strategies/
│       ├── daily-report/
│       ├── debug-strategy/
│       ├── market-analysis/
│       ├── optimize-config/
│       ├── risk-report/
│       ├── strategy-review/
│       └── trade-journal/
└── README.md
```

## License

MIT
