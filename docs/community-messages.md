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
