---
name: run-experiment
description: >
  Design and spawn a forward-test experiment — paper-trade variants of a running bot
  with alternate configs, grouped under a shared experimentGroupId. Use when the user
  asks to "spawn variants", "run a sweep", "forward-test configs", "A/B test parameters",
  "run an experiment", or wants to try alternate settings without risking real capital.
argument-hint: "[source-bot-name-or-id] [parameter=values ...]"
---

# Run Experiment — Spawn Config Variants

Spawn 1–20 paper-trade variants of a running bot with config overrides, all grouped under one `experimentGroupId` and started immediately as `mode=test`.

## Step 1: Identify the source bot

Parse `$ARGUMENTS` for a bot name or UUID.
- If a bot is specified, call `get_bot_details` on it.
- If no bot specified, call `list_bots` and ask the user which bot to clone from.
- Reject if the source bot is `isArchived=true` — the MCP tool will reject this anyway, save a round trip.
- Reject if the source is itself a variant (`experimentSourceBotId` is set) — experiments-of-experiments get confusing fast; ask the user to clone from the original live bot.

Record: `sourceBotId`, `sourceBotName`, `strategy`, current `config`.

## Step 2: Read the strategy descriptor

Use `get_strategy_info` for the source bot's strategy. You need:
- The full list of valid config keys (from `defaultConfig`)
- Any presets for anchoring sweep endpoints
- Parameter ranges/types if documented

This lets you validate override keys client-side and suggest sensible values.

## Step 3: Design the variant set

### If the user specified parameters and values (e.g., "edgeMultiplier at 1.0, 1.25, 1.5")

Build one variant per value:
- Key check: each key must be in `defaultConfig`. Reject `mode` — the tool enforces `mode=test`.
- Label each variant with the changed value(s), e.g., `"edgeMult=1.25"`.

### If the user asked you to design a sweep (e.g., "sweep minFairValue")

Pick 3–5 values laddered around the current config value:
- If current is centered in the preset range → symmetric sweep (e.g., current 0.55 → `[0.45, 0.50, 0.55, 0.60, 0.65]`).
- If current is at an extreme → one-sided sweep toward the other end.
- Always include a variant at the current value as the paper-mode control.

### If the user asked for a grid (multiple params)

Cap total variants at 10–15 even though the tool allows 20. State the grid plainly before spawning.

## Step 4: Fleet-cap check

Call `list_bots`, count bots where `mode="test"`, `status="running"`, `isArchived=false`. The MCP tool enforces `runningTestCount + variants.length <= 50`.

- If the new batch would exceed the cap, tell the user how many running test bots exist and either:
  - Shrink the sweep, or
  - Recommend concluding an old experiment first (suggest the `conclude-experiment` skill).

## Step 5: Confirm with the user

Show the full variant set as a table before calling the tool:

```
## Proposed Experiment

**Source:** {sourceBotName} ({strategy}) — {sourceBotId}
**Variants:** {N}
**Running test bots before launch:** {runningTestCount} / 50

| # | Label | Overrides |
|---|-------|-----------|
| 1 | edgeMult=1.00 | edgeMultiplier: 1.00 |
| 2 | edgeMult=1.25 | edgeMultiplier: 1.25 |
| ... | ... | ... |

Ready to spawn? (If the user said "just do it" upstream, skip this and go to Step 6.)
```

## Step 6: Spawn

Call `spawn_variants` with:

```json
{
  "sourceBotId": "{uuid}",
  "variants": [
    { "label": "edgeMult=1.00", "overrides": { "edgeMultiplier": 1.00 } },
    { "label": "edgeMult=1.25", "overrides": { "edgeMultiplier": 1.25 } }
  ]
}
```

Capture the response:
- `experimentGroupId` (e.g., `exp-7f2a`) — save this prominently, the user will need it to compare and conclude.
- `variants[]` with each `id`, `name`, `label`, `overrides`, `mergedConfig`.

## Step 7: Present the launched experiment

```
## Experiment Launched: {experimentGroupId}

**Source:** {sourceBotName}
**Variants running:** {N} (mode=test, status=running)

| Variant ID | Label | Key Overrides |
|------------|-------|---------------|
| {short uuid} | edgeMult=1.00 | edgeMultiplier: 1.00 |
| {short uuid} | edgeMult=1.25 | edgeMultiplier: 1.25 |

### Next Steps
- Variants need ~30+ simulated trades (2–4 hours) before comparison is meaningful.
- To check progress: `compare-variants {experimentGroupId}`.
- To end it: `conclude-experiment {experimentGroupId}`.

### Caveats
- Variants run in paper mode — simulated fills may be optimistic vs live slippage.
- The variant at the current config value is your paper-mode control, not the live bot itself.
```

## Edge cases

- **Source bot is idle/stopped**: the tool doesn't require the source to be running, but warn the user — a stopped bot means no live benchmark to compare variants against.
- **Override key not in defaultConfig**: pre-validate with the descriptor from Step 2 and show the user the exact invalid key before the tool rejects it.
- **Numeric overrides**: make sure numbers are sent as numbers, not strings. JSON literals, not quoted.
- **Object-valued overrides (e.g., `bands`)**: send the full object; merge is shallow so a partial `bands` object replaces the whole thing. Warn the user when sweeping nested-object keys.
- **User typos a parameter name**: don't guess — ask which key they meant from the descriptor's key list.
