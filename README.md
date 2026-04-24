# STNH Game Log Analyzer

A browser-based tool for analyzing debug logs and save files from the [Star Trek: New Horizons](https://steamcommunity.com/workshop/filedetails/?id=688086068) mod for Stellaris.

**Live app:** https://grinsel.github.io/stnh-log-analyzer/

Everything runs locally in your browser — no data is ever uploaded to a server.

## What it does

Two modes:

### Log mode (`game.log` or STNH debug logs)
Parses structured log entries and visualizes them across 11 tabs:

- **Dashboard** — summary, faction counts, detected human players, game version
- **Revenue** — per-empire gross resource production at any date (13 resources)
- **Income** — per-empire *net* income after upkeep (9 major + 17 rare resources; requires STH_test_events from April 2026 or later)
- **Stats** — Naval Cap, Fleet Power, Planets, Technologies, Traditions, Pops, Unemployment %
- **Rankings** — top-N empires by any resource or stat at any date
- **Wars** — declarations, invasions, war-status timeline
- **Spreadsheet** — flexible cross-date / cross-empire matrix with Run-A-vs-Run-B compare mode
- **Faction Detail** — deep dive on one empire across its entire game history
- **Compare** — 2–4 factions side by side
- **Timeline** — chronological event stream with auto-detected spikes/drops
- **Insights** — auto-flagged anomalies (resource crashes, naval spikes, vanished empires)
- **Meta Analysis** — cross-file diagnostics when multiple logs are loaded

### Save mode (`.sav` files)
Parses Stellaris save files directly in a Web Worker (no Paradox tooling needed):

- **Overview, Empires, Economy, Tech & Traditions, Diplomacy, Wars** — standard inspection tabs
- **Events & Flags** — event chains, global flags, country flags (dual-mode views), variables, fired & player events — all with STNH Wiki links

## Highlights

- **Zero dependencies.** Single `index.html` file. No build step, no npm install, no server required.
- **Works from `file://`.** Double-click the HTML and it runs. Also hosted on GitHub Pages.
- **Multi-log support.** Load several logs at once — merge them or keep them as separate "runs" for A/B comparison.
- **Persistent library.** Logs can optionally be stored in your browser's IndexedDB for reuse across sessions.
- **CSV export** on every data-bearing tab.
- **Wiki-integrated.** Save-mode tabs link out to the STNH Wiki for traditions, events, civics, perks.

## Getting started

### Option 1: Use the live version
Go to https://grinsel.github.io/stnh-log-analyzer/ and drop a log or save file onto the upload area.

### Option 2: Run locally
```sh
git clone git@github.com:Grinsel/stnh-log-analyzer.git
cd stnh-log-analyzer
# Just open index.html in a browser — no build needed
```

A tiny Python server is included for convenience:
```sh
python serve.py   # serves on http://localhost:8080
```

### Where to find the logs
- **Windows:** `%USERPROFILE%\Documents\Paradox Interactive\Stellaris\logs\game.log`
- **macOS:** `~/Documents/Paradox Interactive/Stellaris/logs/game.log`
- **Linux:** `~/.local/share/Paradox Interactive/Stellaris/logs/game.log`

For Save mode, point at any `.sav` file from `…/Stellaris/save games/<empire>/`.

## Documentation

- **In-app help:** click the `?` button top-right — full per-tab docs with a "What's New" patch-notes section at the top.
- **Changelog:** see [`CHANGELOG.md`](./CHANGELOG.md) for release history.
- **Developer docs:** [`CLAUDE.md`](./CLAUDE.md) (AI-agent guide) and [`HANDOVER.md`](./HANDOVER.md) (full dev handover).

## Log format requirements

The analyzer reads lines matching the structured pattern emitted by STNH debug events:

```
[YYYY.M.D] Log effect, file: events/STH_*.txt line: NNN. <payload>
```

Payload patterns it understands include (not exhaustive):
- `<Faction> <value> <resource> revenue` (gross production)
- `<Faction> <value> <resource> income` (net, v2.6.0+)
- `<Faction> <value> <stat>` (naval cap, fleet power, pops, etc.)
- `<Faction> entered war against <Target>`
- `<Faction> Invade Win/Fail On <Date>: <Target> - <Planet>`
- `<Faction> is human player`
- `<Faction> is <category>` (major_faction, significant_power, etc.)

See the in-app help's "Log Format & Parsing" section for the full catalog.

## Tech

Vanilla HTML + CSS + JavaScript. No framework, no bundler, no external CDN at runtime. The entire app is ~7500 lines in one file. Save-file parsing runs in a Web Worker to keep the UI responsive on multi-megabyte files.

## Credits

- **Grinsel** (Marc) — analyzer author and maintainer
- **Orion1988 / Russ** — STNH team; the structured logging events the analyzer depends on
- **STNH mod** — [Star Trek: New Horizons](https://steamcommunity.com/workshop/filedetails/?id=688086068) for Stellaris

## License

No license file is present; treat as "all rights reserved" unless Grinsel publishes otherwise. Open an issue if you want to reuse parts of the code.
