---
name: conclude-experiment
description: >
  End an experiment — stop all variants in an experimentGroupId, then selectively archive
  losing variants while keeping survivors. Use when the user asks to "conclude experiment",
  "stop my sweep", "end exp-XXXX", "archive losers", or wants to wind down a forward-test.
argument-hint: "[experiment-group-id] [keep: top-N or specific-labels]"
---

# Conclude Experiment — Stop & Selectively Archive

Stop every variant in an experiment group, then archive the losers while keeping survivors.

## Step 1: Resolve the experiment group

Parse `$ARGUMENTS`:
- Required: `experimentGroupId` matching `^exp-[0-9a-f]{4}$`. If not a valid format, refuse and ask.
- Optional: "keep top 2", "keep edgeMult=1.75,edgeMult=2.00", "keep best", "keep all", "archive all".
- If no keep-hint, default to asking after showing the ranking.

## Step 2: Run the comparison first

Before stopping anything, run the `compare-variants` skill's logic (Steps 1–5) to produce the ranking. The user needs this to decide which variants to keep.

- If sample size is low across the board (`< 30 trades` for most variants), **warn prominently** and ask the user to confirm before concluding. Suggest running longer.
- Show the ranking table (abbreviated — rank, label, PF, trades, PnL).

## Step 3: Decide which variants to keep

Apply the user's keep-hint:
- **"keep top N"** → keep the top N by profit factor (or expectancy if mixed samples).
- **"keep {label1},{label2}"** → keep exactly those labels.
- **"keep best"** → keep only the top 1.
- **"keep all"** → stop the group but archive nothing.
- **"archive all"** → archive every variant.
- **No hint / ambiguous** → ask the user. Do not guess.

Build two lists:
- `toKeep`: variant IDs that will be stopped but NOT archived.
- `toArchive`: variant IDs that will be stopped AND archived.

Never include the source bot in either list — `stop_experiment` operates on `experimentGroupId` which doesn't match the source, and `archive_variant` rejects standalone bots anyway. Just be explicit to the user: "The source bot is not affected."

## Step 4: Confirm the plan

Show the user before calling any tool, unless they've said "just do it":

```
## Concluding Experiment {groupId}

**Stop (all variants in group):** {N} bots — will be moved to status=stopped
**Archive (losers):** {M} bots — {list of labels}
**Keep (survivors, stopped but visible):** {K} bots — {list of labels}
**Source bot:** not affected ({sourceBotName}, status={source status})

Proceed?
```

## Step 5: Stop the group

Call `stop_experiment` with `{ experimentGroupId }`. Capture the returned list of stopped bot IDs.

- Cross-check: the returned list should equal the union of `toKeep` and `toArchive`. If not, note the discrepancy (likely means a variant was already archived).

## Step 6: Archive the losers

For each bot ID in `toArchive`, call `archive_variant` with `{ botId }`.

- Do these sequentially (it's cheap, and sequential is easier to error-report than parallel).
- If any archive call fails, continue with the rest and collect errors; report at the end.

## Step 7: Final summary

```
## Experiment Concluded: {experimentGroupId}

**Stopped:** {N} variants
**Archived:** {M} variants (losers)
**Surviving variants (stopped, in UI):**
| Label | Final PF | Trades | Note |
|-------|----------|--------|------|
| edgeMult=2.00 | 1.45 | 52 | leader |
| edgeMult=1.75 | 1.22 | 48 | runner-up |

**Archived (cleaned up):**
{comma-separated labels}

**Source bot:** {sourceBotName} — unchanged, still {status}.

### Recommendation
{1–2 sentences: based on the winner's overrides, whether the user should apply those values to the live source bot. Always include the caveat that paper-mode fills are optimistic vs live slippage.}
```

## Edge cases

- **Group doesn't exist**: `stop_experiment` returns `stopped: []`. Tell the user the group ID didn't match any non-archived variants.
- **Variant already archived before conclude**: appears in `list_bots` with `isArchived=true` and won't be returned by `stop_experiment`. Skip it — don't error.
- **archive_variant fails for a variant** because it's somehow not a variant (no `experimentSourceBotId`): should never happen for variants from `spawn_variants`, but if it does, report which bot and move on.
- **User asks to keep all and archive none**: perfectly valid — just call `stop_experiment` and skip the archive loop.
- **Small sample size confirmed against user wish**: if the user insists on concluding early, respect it but include a "concluded early; ranking may not hold under larger sample" note in the summary.
- **No winner** (all variants worse than paper control): keep none, archive all — and suggest the user rethink the sweep design, not the live config.
