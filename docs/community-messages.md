# Community Messages — ready to paste

Short-form messages for Discord / Steam forum / Paradox forum announcing v2.6 / v2.7.

---

## For Orion1988 (STNH logging-events contributor)

> Good news: **v2.7.1 is live** — all five of your new event types are now fully wired up in the analyzer:
>
> - **Fleet power**, **unemployed pops**, **unemployment %** (derived), and **influence** shipped in v2.6 and are visible across Stats, Rankings, Chart, Spreadsheet, Faction Detail, Compare, Timeline.
> - **Rare + Major income** got a dedicated **Income tab** in v2.7 (between Revenue and Stats) with a sub-toggle for Major (9 cols) / Rare (17 cols) / All (26 cols). Selecting empires + "Show Graph" opens a proper income line chart.
>
> The gross-vs-net comparison is now inspectable side by side. From your test log: Klingons 2180 show **+365 energy revenue** but **-67 energy income** — exactly the upkeep-burden signal that was invisible before.
>
> Also built in for future-proofing: a scalable date picker (searchable combobox + prev/next + slider). Your logging is currently decadal (3 snapshots), but if Russ ever switches to annual or monthly snapshots the UI will stay usable even at 2400 entries.
>
> Full release notes: [docs/v2.6-v2.7-orion-events.md](./v2.6-v2.7-orion-events.md). In-app patch notes under `?` → "What's New".
>
> Happy to tweak defaults, column order, or labels if anything feels off after you try it.

---

## For Russ (STH_test_events author) — upstream bug report

> Heads-up on a small thing I noticed while wiring up income parsing:
>
> The `... income` lines in `STH_test_events.txt` are logged **twice per date** for the same `(country, resource)` tuple. For example, dilithium income gets emitted at both line 1772 and line 1824, and I see the same pattern for the other major/rare income keys.
>
> Example from a 5 MB game.log:
> ```
> [2160.1.1] ... Klingon Empire 59.18212 dilithium income  (line 1772)
> [2160.1.1] ... Klingon Empire 59.18212 dilithium income  (line 1824)
> ```
>
> My parser dedupes on `(date, faction, key)` — first value wins — so there's no data corruption downstream. But the logs would be roughly **half the size** if one of the two emissions is dropped at the source. For players on longer playthroughs with periodic logging, that adds up quickly.
>
> No rush — just flagging in case it's unintended.

---

## For general STNH community (Discord / Steam announcement)

> **STNH Log Analyzer v2.7.1** is live: https://grinsel.github.io/stnh-log-analyzer/
>
> What's new since v2.5:
>
> - **Fleet Power** stat — first real military-strength metric alongside naval cap
> - **Unemployment tracking** — both raw pop counts and derived % of total
> - **Income tab** (new) — shows *net* income after upkeep, distinct from the gross Revenue tab. Major (9) / Rare (17) / All (26) sub-toggle
> - **Influence** added to the resource family
> - **Scalable date picker** — stays usable even with thousands of log snapshots
>
> Zero install, runs in your browser, no data ever uploaded. Drop your `game.log` on the page.
>
> Requires STH_test_events from **April 2026 or later** for the income / fleet power / unemployment data (older logs still work, those tabs just stay empty).
>
> In-app docs: click the `?` top-right → "What's New". Full changelog on [GitHub](https://github.com/Grinsel/stnh-log-analyzer/blob/master/CHANGELOG.md).

---

## Discord-sized TL;DR (one-liner variants)

> 🆕 **Log Analyzer v2.7.1** — Income tab (net after upkeep), Fleet Power stat, Unemployment tracking, scalable date picker. `?` button in app for full notes. https://grinsel.github.io/stnh-log-analyzer/

> Analyzer update: gross *Revenue* and net *Income* are now separate tabs. Klingons can have +365 energy revenue and -67 income at the same time — that delta is upkeep burden, previously invisible. https://grinsel.github.io/stnh-log-analyzer/

---

## For Orion1988 — v2.16.0 follow-up (2026-05-17)

> Quick update — your diagnostic-logging additions from the May 5 chat are now all wired up in **v2.16.0**. Tested against both logs you dropped in `input_code/`:
>
> - **Game id number** — parsed, shown on the Dashboard, and the Validation tab now has a "Game ID consistency" card that fires red when a loaded log contains multiple distinct IDs (auto-detects merged games).
> - **Galaxy map name** — auto-detects the map and switches the Galaxy tab to one of all 16 wiki layouts (Default, Lore, Mirror, TNG, BOTF, Alpha/Beta in 3 sizes, Delta, Gamma, Medium + variants). Random maps (sphere / elliptical / spiral / ring) get a "no fixed layout" notice — let me know if Russ ships a `galaxy_size` ID for those too. Also: this effectively resolves my older Discord request to Russ for a map-detection event, you got there first 🎉
> - **Subjects count** — added as a new `Subjects` stat. Klingons 4–5, Romulans 2, Dominion 1 — matches your example.
> - **Game Q-settings** (`Ethics and Factions OFF`, `Ethics and Factions OFF for AI`) + `Specified Seed` — all on a new Dashboard banner.
> - **Multi-game `is human player`**: handled correctly — when you played Antican Packs for the first 10 years and then went observer, both behaviours are visible (Antican Packs is human early, then goes silent; in `game(3).log` the same logic shows United Earth → United Federation of Planets transitioning cleanly at 2168).
>
> **Two mod-side things to flag back:**
>
> 1. In `improvedgameloggin2.log`, the `navy_size` payload tail is leaking into the next event with Stellaris color-code control bytes still attached. Raw line example: `^Snavy_size^S ^QUAntican Packs^Q! 0 subjects` (where `^Q` and `^S` are 0x11/0x13). I'm stripping them defensively in the analyzer so data stays clean, but the new **"Malformed log lines"** validation card lists all affected lines (37 in the test log) so you can spot the root cause — probably a missing newline or a stray color-code in the `navy_size` log effect. In `game(3).log` from the slightly newer mod build the issue is already gone.
> 2. Same log has the `Game id number is …` line emitted twice in 2160 (once from STH_test_events.txt line 1533, once from line 1919). Same ID both times, so it's just log spam, not a parser issue.
>
> The net-income-for-all-countries change is the one ask I haven't shipped yet — neither test log has any `... income` lines, so I'd rather wait for a log that exercises it and validate end-to-end. The income parser already knows all 26 keys, so it might just work as-is.
>
> Patch notes are in the in-app `?` → "What's New", or on GitHub.
