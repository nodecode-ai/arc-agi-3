# ARC-AGI-3: Nodecode, Claude Code and the Codex CLI

Every run behind Foresight Labs' study [FL-2026-006](https://foresight-labs.eu/research/fl-2026-006): the 25 public
[ARC-AGI-3](https://arcprize.org/arc-agi/3) games played by Nodecode and by each model maker's coding agent, on
Opus 5.5 (against Claude Code) and on GPT-6.1 Sol (against the Codex CLI), all at xhigh reasoning. That is one run per
harness and game, 100 runs in all. Each run comes with its score card, its full transcript and the game's recording.

## Results

All four harnesses win every level of all 25 games (183 levels).

| Opus 5.5 | Nodecode | Claude Code |
| --- | --- | --- |
| Score, mean of 25 games | 100 | 100 |
| Cost | $123.79 | $209.67 |
| Model calls | 2,332 | 2,241 |
| Tokens, all kinds | 180.5M | 393.0M |
| Output tokens | 2.85M | 4.11M |
| Cache reads | 171.1M | 382.6M |
| Cache writes | 6.5M | 6.4M |
| Game actions | 7,103 | 6,695 |

| GPT-6.1 Sol | Nodecode | Codex CLI |
| --- | --- | --- |
| Score, mean of 25 games | 99.54 | 99.93 |
| Cost | $32.55 | $42.97 |
| Model calls | 2,822 | 2,298 |
| Tokens, all kinds | 57.8M | 171.7M |
| Output tokens | 1.08M | 1.12M |
| Cache reads | 50.0M | 164.5M |
| Uncached input | 6.69M | 6.12M |
| Game actions | 8,920 | 7,575 |

Cost of each game, with the score where it is below 100:

| Game | Nodecode, Opus 5.5 | Claude Code | Nodecode, GPT-6.1 Sol | Codex CLI |
| --- | --- | --- | --- | --- |
| ar25 | $1.63 | $3.40 | $0.43 | $0.71 |
| bp35 | $15.81 | $28.44 | $4.28 (90.50) | $3.81 |
| cd82 | $1.29 | $2.07 | $0.37 | $0.56 |
| cn04 | $2.97 | $5.93 | $1.01 | $0.85 |
| dc22 | $5.35 | $8.98 | $2.27 | $3.29 |
| ft09 | $0.69 | $1.06 | $0.41 (98.97) | $0.59 |
| g50t | $3.94 | $3.82 | $0.69 | $1.16 (98.25) |
| ka59 | $4.48 | $10.72 | $1.53 | $2.49 |
| lf52 | $14.60 | $22.74 | $4.49 | $3.35 |
| lp85 | $2.21 | $3.71 | $0.81 | $0.73 |
| ls20 | $6.59 | $8.14 | $1.89 | $2.62 |
| m0r0 | $2.71 | $4.64 | $0.77 | $1.05 |
| r11l | $2.11 | $3.96 | $0.77 | $1.91 |
| re86 | $4.58 | $7.41 | $1.22 | $1.88 |
| s5i5 | $8.09 | $15.69 | $1.14 | $1.40 |
| sb26 | $0.67 | $1.07 | $0.22 | $0.58 |
| sc25 | $2.43 | $3.86 | $0.67 | $1.14 |
| sk48 | $7.91 | $13.54 | $1.91 (99.02) | $4.79 |
| sp80 | $6.81 | $11.89 | $1.00 | $0.76 |
| su15 | $5.41 | $19.05 | $1.75 | $2.91 |
| tn36 | $3.36 | $3.94 | $0.79 | $0.88 |
| tr87 | $3.85 | $3.42 | $0.72 | $1.10 |
| tu93 | $5.11 | $7.26 | $0.88 | $1.31 |
| vc33 | $3.10 | $3.73 | $1.00 | $1.14 |
| wa30 | $8.06 | $11.19 | $1.51 | $1.96 |

## Scoring and cost

A level scores 100 × (human baseline actions / actions taken)², capped at 115. Level *i* has weight *i*, and a won game is
capped at 100. This is the ARC-AGI-3 score card's rule.

Cost is every model call's tokens at the providers' official prices of 11 October 2026, in dollars per million tokens:

| Model | Input | Cache read | Cache write | Output |
| --- | --- | --- | --- | --- |
| Opus 5.5 | $4.00 | $0.20 | $5.00 (5 minutes), $8.00 (1 hour) | $20.00 |
| GPT-6.1 Sol | $2.00 | $0.10 | $2.50 | $10.00 |

On GPT-6.1 Sol every uncached input token is priced as a cache write, the higher of the two rates, for both harnesses.
Reasoning tokens are counted as output tokens.

## Layout

```
results.csv                          one row per run: score, actions, calls, tokens by kind, cost
runs/<model>/<harness>/<game>/
  result.json                        the run's score card, model calls, tokens and cost
  transcript.jsonl                   the conversation, one item per line
  recording.jsonl.gz                 the game as played: every action and the frames it produced
```

`<model>` is `opus-5.5` or `gpt-6.1-sol`, and `<harness>` is `nodecode`, `claude-code` or `codex-cli`.

**result.json** has the game, the harness, the model, the final state, the levels won out of the total, the actions taken,
the actions per level against the human baseline, each level's score, the game score, the number of model calls, the
tokens by kind, and the cost.

**transcript.jsonl** is the same for every harness. Each line has a `type`:

| type | fields | what it is |
| --- | --- | --- |
| `system` | `text`, (`sections`) | instructions the harness gave the model |
| `user` | `text` | the task prompt |
| `reasoning` | `text` | reasoning the provider returned in readable form, where it did |
| `assistant` | `text` | the model's visible text |
| `tool_call` | `id`, `name`, `input` | a tool call |
| `tool_result` | `id`, `output`, (`error`) | its result |
| `usage` | token counts, `cost_usd` | one model call's tokens and cost |
| `context_edit` | `removed_blocks`, `kept_blocks`, `note` | Nodecode on GPT-6.1 Sol: the model edits its own context |
| `compaction` | `text` | Codex CLI: the summary that replaced a compacted context |

The `usage` lines of a transcript add up to the run's `result.json`. Tool-call ids are renumbered `t1`, `t2`, ...
Local paths, user names and dates are redacted.

**recording.jsonl.gz** has one game step per line, in order. Each step has the action taken (`action_input`), the game
state, the levels completed, the available actions and `frame`, the list of 64×64 grids the action produced, ending on
the settled board.

## Acknowledgements

ARC-AGI-3 is by the [ARC Prize Foundation](https://arcprize.org). Its games and scoring rules are what this study is
built on.
