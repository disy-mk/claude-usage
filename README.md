# claude-usage

A live view of your Claude Code token consumption in the terminal: rate limits,
today's and rolling 24-hour usage broken down **per session**, plus colored
history graphs over hours, days and weeks.

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ Claude Code — Max 5x                                               214 Prompts heute · Stand 15:02:32 CEST │
├──────────────────────┬───────┬────────────────────────────────────────────────────────┬────────────────────┤
│ Session (5-hour)     │   42% │ ██████████████████████································ │ Reset 10.09. 17:12 │
│ Weekly (7-day)       │   61% │ ████████████████████████████████······················ │ Reset 14.09. 15:02 │
│ Opus Weekly          │   23% │ ████████████·········································· │ Reset 14.09. 15:02 │
├──────────────────────┴───────┴──────────┬───────────┬───────────┬───────────┬─────────┴───────┬────────────┤
│ Tokens                                  │     heute │      24 h │    gesamt │ seit            │ Projekt    │
├─────────────────────────────────────────┼───────────┼───────────┼───────────┼─────────────────┼────────────┤
│ Alle Sessions                           │    33.61M │   37.742M │           │                 │            │
├─────────────────────────────────────────┼───────────┼───────────┼───────────┼─────────────────┼────────────┤
│ payment retry backoff                   │    24.31M │    24.31M │     31.8M │ heute 11:00     │ ~/code/api │
│ flaky integration tests                 │     6.12M │     9.44M │     58.7M │ gestern 16:00   │ ~/code/api │
│ docs rewrite onboarding                 │     3.18M │     3.18M │     3.18M │ heute 09:00     │ ~/code/web │
│ log parser prototype                    │         0 │    812.4k │    812.4k │ gestern 14:00   │ ~          │
├───────────────────────────────────┬─────┴───────────┴───────────┴──────┬────┴─────────────────┴────────────┤
│ 24 h · 3 h je Zeile               │ 8 Tage                             │ 8 Wochen                          │
├───────────────────────────────────┼────────────────────────────────────┼───────────────────────────────────┤
│ 18-21 ··················        0 │ 03.09. ⣿⣿⣿⣿⣿⣿············    18.3M │ KW 30 ··················        0 │
│ 21-00 ··················        0 │ 04.09. ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿····    41.2M │ KW 31 ··················        0 │
│ 00-03 ⣿⣿⣿···············    1.94M │ 05.09. ⣿⣿················     6.4M │ KW 32 ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿········     112M │
│ 03-06 ··················        0 │ 06.09. ··················        0 │ KW 33 ⣿⣿⣿⣿⣿⣿⣿⣿··········    87.4M │
│ 06-09 ⣿⣿⣿⣿⣿⣿············    4.26M │ 07.09. ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿    52.9M │ KW 34 ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿·····   143.9M │
│ 09-12 ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇···    9.87M │ 08.09. ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿·····    37.6M │ KW 35 ⣿⣿⣿⣿⣿⣿⣿⣿⡇·········    96.2M │
│ 12-15 ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿    12.4M │ 09.09. ⣿⣿⣿···············    9.44M │ KW 36 ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿·······   121.5M │
│ 15-18 ⣿⣿⣿⣿⣿⣿⣿⡇··········    5.02M │ 10.09. ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇······   33.61M │ KW 37 ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿  199.45M │
└───────────────────────────────────┴────────────────────────────────────┴───────────────────────────────────┘
```

Colors do not survive this code block — in a real terminal the bars run from
green through orange to red, and Saturday and Sunday are tinted in the day
column.

> **Note:** the interface is in German (`heute` = today, `gesamt` = total,
> `seit` = since, `KW` = calendar week). All strings live in `render()` and the
> column headers are plain literals, so translating is a short edit.

## Why

Claude Code tells you when you hit a limit, not where your tokens went. The
transcripts under `~/.claude/projects` hold the answer, but only as raw JSONL.
`claude-usage` reads them directly and aggregates what is actually useful:
which session burned the tokens, how much of it happened today, and how the
last hours, days and weeks compare.

## Requirements

- **Python 3.9+**, standard library only — no dependencies, no virtualenv.
- **Claude Code**, with transcripts under `~/.claude/projects`
  (`CLAUDE_CONFIG_DIR` is honored).
- **Optional: [Omarchy](https://omarchy.org).** The rate-limit block at the top
  comes from Omarchy's `omarchy-agent-usage-claude` collector, which talks to
  Anthropic's OAuth usage endpoint. Without it the tool still works and prints
  everything else — you just lose the limits row and see a note saying so.
  Everything below the limits is computed from the transcripts alone.

## Install

```bash
git clone https://github.com/disy-mk/claude-usage.git
ln -s "$PWD/claude-usage/claude-usage" ~/.local/bin/claude-usage
```

Any directory on your `PATH` will do; a symlink keeps you on the latest pull.

## Usage

```bash
claude-usage          # print once
claude-usage -w       # live view, refresh every 60 s
claude-usage -w 15    # live view, refresh every 15 s
claude-usage -h       # short help
```

`Ctrl-C` leaves the live view. Refresh intervals below 15 s are raised to 15 s —
see *Caveats*.

## Reading the table

| Column | Meaning |
|---|---|
| `heute` | Calendar day, from 00:00 local time. |
| `24 h` | Rolling window — the last 24 hours from now. Always at least as large as `heute`. |
| `gesamt` | Every token of that session since it started, regardless of window. Only meaningful per session, so the `Alle Sessions` row leaves it blank. |
| `seit` | When the session started. |
| `Projekt` | The session's working directory. Only shown when the terminal is wide enough. |

Sessions are listed when they saw activity in the last 24 hours, sorted by
consumption in that window. A `0` under `heute` means the session ran
yesterday but still falls inside the rolling window. The `heute` and `24 h`
columns add up exactly to the `Alle Sessions` row.

**Session names** come from the `ai-title` records Claude Code writes into the
transcripts — the title it gives a session and refines as it goes; the last one
wins. Sessions without a title fall back to the first 8 characters of their id.

### Number format

| Range | Format | Example |
|---|---|---|
| < 1,000 | as-is | `999` |
| from 1,000 | `k`, one decimal | `1.5k`, `124.7k` |
| from 1,000k | `M`, up to three decimals | `1.231M`, `216.442M` |

Trailing zeros are trimmed for `M`: `1.000M` prints as `1M`, `8.640M` as
`8.64M`. The switch to `M` happens at 999,950 so that `1000.0k` never appears.

### Times

Everything is local time, with the zone abbreviation on the timestamp. Session
starts render as `today HH:MM` / `yesterday HH:MM` / `DD.MM. HH:MM`. The
underlying data is UTC; conversion follows `TZ` or the system zone.

## History graphs

Three columns of eight rows each, newest at the bottom:

| Column | Bucket | Label |
|---|---|---|
| left (`24 h · 3 h je Zeile`) | 3-hour blocks | `12-15` (block start–end) |
| middle (`8 Tage`) | calendar days | `10.09.` |
| right (`8 Wochen`) | ISO weeks (Mon–Sun) | `KW 37` |

The bars are drawn with braille dots, the way `btop` does it. A braille cell is
two dot columns wide, so a bar has twice the resolution of its character width.
Any value above zero gets at least half a dot so it never reads as empty.

In the day column the **date is tinted by weekday**: Monday through Friday in
the normal terminal color, Saturday a muted orange, Sunday a muted red, so
weekends stand out at a glance.

**Each column scales to its own maximum**, not to a shared one — three hours,
one day and one week are orders of magnitude apart. A full bar therefore means
"peak of this column", not "a lot" in absolute terms.

## Width and color

The output fills the terminal width, at minimum 86 and at most 200 characters
(`MIN_WIDTH` / `MAX_WIDTH`). Space beyond the base layout goes to the bars, to
the session names, and — once at least 13 characters are left over — to the
extra `Projekt` column. At 86 characters that column disappears again.

Bars are **green when low, red when high**, using the same gradient as `btop`.
Tinting follows the position within the bar rather than the value, so a short
bar stays entirely green while a long one runs out through orange into red.
This applies to the limit bars too, which makes a limit at 90% readable at a
glance.

Color is disabled when the output is not a terminal (a pipe or a file), when
`NO_COLOR` is set, or when `TERM=dumb`.

## How it works

Two sources:

1. **Limits and plan** from `omarchy-agent-usage-claude --limits-only`.
2. **Token counts** from the tool's own scan of `~/.claude/projects`. Necessary
   because the collector only emits aggregates — daily total, per-model total,
   weekly trend — with no per-session breakdown and no 24-hour window.

Counted are assistant messages carrying a `usage` field; a message total is
`input_tokens + output_tokens + cache_read_input_tokens +
cache_creation_input_tokens`, deduplicated by message id.

## Caveats

- **Limits need a logged-in CLI.** Without valid credentials the limit rows are
  replaced by the collector's hint. The transcript-based numbers are unaffected.
  Fix with `claude auth login`.
- **Minimum interval 15 s.** The collector throttles the limits endpoint to one
  probe every 15 seconds, so smaller `-w` values are raised.
- **Roughly 0.3–0.7% higher than Omarchy's bar panel.** While a response is
  streaming, Claude Code writes several lines under the same message id with
  growing `output_tokens`. The collector takes the *first* occurrence and
  undercounts output; `claude-usage` takes the *last*, i.e. the final usage.
  Prompt counts match exactly.
- **The collector caches its transcript scan for 900 s**, which is why the
  panel's daily total lags. `claude-usage` rescans on every refresh. Pass
  `--force` to the collector if you want its number fresh.
- **The 3-hour blocks are clock-aligned**, not relative to *now* — they start at
  00:00, 03:00, 06:00 and so on. That keeps the labels honest, but it also means
  the left column spans 21 to 24 hours depending on the time of day, so its sum
  sits slightly below the `24 h` figure in the table. The newest block is also
  still in progress.
- **Old transcripts get cleaned up.** How long they stay is governed by
  `cleanupPeriodDays` in `~/.claude/settings.json`. For the week column to fill
  completely you need at least 60 days there, otherwise older weeks stay at zero
  forever.
- **Subagents** run under their parent's session id and are counted towards it
  rather than listed separately.

## Tuning

A single Python file, no dependencies — edit it directly. The interesting knobs
sit at the top: `WINDOW_HOURS` (length of the rolling window), `GRAPH_ROWS`,
`MIN_WIDTH` / `MAX_WIDTH`, `PROJECT_MIN` / `PROJECT_MAX`, `GRADIENT` (bar
gradient) and `WEEKEND` (weekend tints). `build_layout()` computes the column
widths from the terminal width, around an 84-character base grid:

```python
limits = (22, 7, 32 + extra, 20)              # window, percent, bar, reset
tokens = (30 + …, 11, 11, 11, 17 [, project]) # name, today, 24 h, total, since
graph  = (27 + …, 28 + …, 27 + …)             # 3-hour blocks, days, weeks
```

All three blocks must end up the same inner width, or the frames stop lining up:

```
sum(limits) + 3 == sum(tokens) + (len(tokens) - 1) == sum(graph) + 2 == inner
```

`separator()` draws the rules and places `┬ ┴ ┼` at the column boundaries on its
own, including where two blocks have different column counts.

## Related

Omarchy ships a graphical **agents panel** in its bar: always visible, with a
weekly chart and per-model split, but no per-session breakdown. Disable it with
`omarchy plugin disable omarchy.agents`. Collectors for other subscriptions
exist as `omarchy-agent-usage-codex` and `omarchy-agent-usage-fireworks`.

## License

MIT — see [LICENSE](LICENSE).
