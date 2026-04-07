# STNH Game Log Analyzer — Handover

## Project Summary

A single-file, zero-dependency HTML application for analyzing debug logs produced by **Stellaris: Star Trek New Horizons (STNH)** mod games. Everything (HTML + CSS + JS) lives in `game_log_analyzer.html`. No build step, no server, no external libraries — open it in a browser and drag a `.log` file onto it.

**Current version:** `v1.0.0` (build `2026-04-07`)

Version constants are defined once at the top of the `<script>` block:

```js
const APP_VERSION = '1.0.0';
const APP_BUILD   = '2026-04-07';
```

The version is auto-rendered into the document title, the header badge (top right), and the browser console on load. **Always bump these when releasing changes.**

---

## Repository Setup

Suggested layout for the new repo:

```
stnh-log-analyzer/
├── game_log_analyzer.html   # The whole app
├── samples/                 # Example log files for manual testing
│   └── game-1.log
├── HANDOVER.md              # This file
├── README.md                # User-facing docs
└── CHANGELOG.md             # Per-version notes
```

No `package.json`, no bundler, no framework. Keep it that way unless there's a strong reason — the "single HTML file you can email to anyone" property is a feature.

---

## What the App Does

### Input
A debug log file (`game.log` or similar) emitted by STNH event scripts. Relevant lines look like:

```
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 1607. Antedean Shoals is human player
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 42. United Federation of Planets 116.81 energy revenue
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 99. Klingon Empire entered war against Romulan Star Empire
```

### Parser Recognizes Five Entry Types
1. **Revenue:** `<faction> <number> <resource> revenue` — for resources defined in the `RESOURCES` array (energy, mineral, food, alloy, dilithium, ketracel white, etc.)
2. **Stats:** `<faction> <number> <metric>` — for metrics in the `STATS` array (used naval cap, owned planets, researched technologies, etc.)
3. **War declarations:** `<faction> entered war against <target>`
4. **Invasions:** `<faction> Invade Win|Fail On <date>: <target> - <planet>`
5. **War status:** `<faction> is at war`
6. **Human player flag:** `<faction> is human player` — uses a **broader regex** that matches any `events/STH_*.txt` file, not just `STH_test_events.txt`

The other five entry types currently still require the source file to be exactly `STH_test_events.txt`. If a future log version uses different event files for those, the regex in `parseLog()` (currently `re`) needs to be loosened the same way `humanRe` already is.

### UI Tabs
1. **Dashboard** — Aggregate counts, prominent human-player banner, date range
2. **Revenue** — Per-faction resource revenue table, multi-select to graph empires over time
3. **Stats** — Same as Revenue but for stat metrics
4. **Rankings** — Top-N empires by chosen metric and date
5. **Wars** — Filterable, paginated war event list
6. **Spreadsheet** — Pivot view: empire × resource (single date) or empire × date (single resource), with heatmap and growth column
7. **Faction Detail** — Per-empire deep dive: revenue table + sparklines, stats table, war history. Defaults to a human player if one exists.
8. **Meta Analysis** — Cross-file comparison (see below)

### Multi-File Mode
The file input accepts multiple `.log` files at once. Each is parsed sequentially with a progress indicator. Data aggregates into all existing tabs as if it were one big dataset. Per-file metadata (name, date range, factions, humans, counts) is tracked separately in `data.files` for the **Meta Analysis** tab, which surfaces:
- Per-file summary cards
- Global human player list with `(N/M)` file occurrence
- Cross-file faction presence matrix with green/gray dots
- Auto-generated systemic observations (varying human counts, missing humans per file, empty files, large record-volume variance, mismatched start dates)

### Human Player Highlighting
When a `<faction> is human player` line is detected, that faction is added to `data.humanPlayers`. Visible everywhere:
- Pulsing green banner on Dashboard
- Glowing `HUMAN` pill badge next to the name in every table, chart legend, and war entry
- Light green row highlight + 4px green left-border stripe in tables
- `[H]` prefix in dropdowns; humans sorted to the top
- Faction Detail tab opens on a human by default and shows a green "HUMAN PLAYER" header card

---

## Code Architecture

It's a single ~2350-line HTML file. The `<script>` block is organized into clearly commented sections (search the file for `// ==========`):

| Section | Lines (approx) | Purpose |
|---|---|---|
| `VERSION` | top of `<script>` | App version constants |
| `DATA STRUCTURES` | ~775 | The global `data` object + `RESOURCES`, `STATS`, label maps |
| `DATE UTILITIES` | ~820 | Parse/compare/format `YYYY.M.D` strings |
| `PARSER` | ~840 | `parseLog(text, fileName)` + `finalizeData()` |
| `UI INITIALIZATION` | ~1015 | `initUI()` orchestrator + `init*()` per tab |
| `DASHBOARD` | ~1033 | Card grid + human banner |
| `REVENUE` | ~1095 | Table + multi-select + graph trigger |
| `CHART OVERLAY` | ~1230 | Custom canvas line chart used by Revenue and Stats |
| `STATS` | ~1480 | Same shape as Revenue |
| `RANKINGS` | ~1539 | Top-N table |
| `WARS` | ~1584 | Filterable paginated list |
| `SPREADSHEET` | ~1645 | Pivot table with two modes |
| `FACTION DETAIL` | ~2013 | Per-empire deep dive |
| `META ANALYSIS` | ~2050 | Cross-file tab |
| `HELPERS` | ~2200 | `esc`, `fmtNum`, `isHuman`, `humanMark`, `humanPrefix` |
| `TAB SWITCHING` | ~2235 | Tab button click handler |
| `FILE LOADING` | ~2245 | `loadFiles()` async multi-file loader + drop zone |

### The `data` Object (Global State)

```js
const data = {
  revenue: {},        // { date: { faction: { resource: value } } }
  stats: {},          // { date: { faction: { metric: value } } }
  wars: [],           // [{ date, type, faction, target, ... }]
  allDates: [],       // sorted date strings
  allFactions: new Set(),
  humanPlayers: new Set(),
  files: [],          // [{ name, firstDate, lastDate, factions:Set, humans:Set, counts:{} }]
  counts: { revenue, stats, warDecl, invasion, warStatus }
};
```

`finalizeData()` is called **once** after all files are parsed. It sorts `allDates` and `wars`. Don't call it inside `parseLog()` — that would break multi-file mode.

### Helper Functions
- `isHuman(faction)` → boolean
- `humanMark(faction)` → returns ` <span class="human-badge">HUMAN</span>` HTML or empty string. Use this when rendering faction names into HTML.
- `humanPrefix(faction)` → returns `'[H] '` or empty string. Use this for `<option>` text where HTML isn't allowed.
- `esc(s)` → HTML escape. **Always use** when injecting faction names or any user data into innerHTML.
- `fmtNum(v)` → locale-formatted number with 2 decimal places.

---

## Conventions

- **No external dependencies.** Vanilla JS, vanilla CSS, no fonts from CDNs, no build step. The file must remain openable from `file://`.
- **Always escape faction names** with `esc()` when building HTML strings — STNH faction names can contain quotes, ampersands, etc.
- **CSS variables** for colors (`var(--accent)`, `var(--success)`, etc.) — defined in `:root` at the top of the stylesheet. Don't hardcode hex colors in new components.
- **Theme:** Dark theme with red accent (`#e94560`) and teal success color (`#4ecdc4`) — matches the broader STNH wiki aesthetic. A light theme would be a future enhancement.
- **German umlauts** in any user-facing German text must be the real characters (ä ö ü ß), never `ae oe ue ss`. The current UI is English, but if German strings are added later, this rule applies.
- **Bump `APP_VERSION`** for any user-visible change. Use semver:
  - PATCH (1.0.1): bug fixes, visual tweaks
  - MINOR (1.1.0): new features, new tabs, new heuristics
  - MAJOR (2.0.0): breaking parser/data-format changes
- **Update `APP_BUILD`** to the release date (ISO format `YYYY-MM-DD`).
- **Add a `CHANGELOG.md` entry** for every version bump.

---

## Known Limitations / Future Ideas

These are open improvements, not bugs:

1. **Parser regex is `STH_test_events.txt`-locked** for everything except human-player detection. Loosen to `STH_*.txt` if logs from other event files need to be parsed.
2. **No persistence.** Reloading the page loses all loaded files. Could use IndexedDB to remember the last session, but be careful — log files can be large.
3. **No CSV/JSON export.** Useful for further analysis in Excel/Python.
4. **Spreadsheet tab can be slow** with 150+ factions × many dates. Could virtualize rows.
5. **Chart is a custom canvas implementation** — works but is bare-bones. Tooltips, zoom, axis customization could all be improved.
6. **Meta Analysis observations are hand-rolled heuristics.** Could add more (e.g., factions that disappear mid-game, sudden resource crashes, war frequency spikes).
7. **No light theme.** The broader stnh wiki project has plans for half-dark and light themes — this analyzer should eventually match.
8. **Faction Detail war history** isn't paginated and could blow up with very long games.
9. **No file-level filtering in main tabs.** When multiple files are loaded, you can't yet view just one file's data in the Revenue/Stats/etc tabs without reloading.
10. **No tests.** Manual QA only. A small Node-based parser test (the kind we ran in development against `samples/game-1.log`) would be cheap to add.

---

## Test Procedure

Until automated tests exist:

1. Drag `samples/game-1.log` (or any real game log) onto the drop zone.
2. Verify the Dashboard shows the correct human player(s) in the green banner.
3. Check the Revenue, Stats, and Rankings tabs — the human empire row should be visibly highlighted (green background, left-border stripe, `HUMAN` badge).
4. Open the Faction Detail tab — it should default to a human player and show the green "HUMAN PLAYER" header card.
5. Switch to the Meta Analysis tab — should list the loaded file with correct counts and the human in its file card.
6. Drop a second log file (refresh first — current build doesn't append) and verify cross-file presence dots and observations populate.
7. Open the browser console — verify the version banner is logged.
8. Check the header badge top-right — should match `APP_VERSION`.

---

## Common Tasks

### Add a new resource or stat
Edit `RESOURCES` / `STATS` arrays and the corresponding `RESOURCE_SHORT` / `STAT_SHORT` label maps in the `DATA STRUCTURES` section.

### Add a new tab
1. Add a `<button>` to `<nav class="tab-nav">`
2. Add a `<div class="tab-panel" id="tab-yourname">` after the existing panels
3. Write `initYourTab()` and add it to `initUI()`
4. Tab switching is handled automatically by the existing `.tab-btn` click listener

### Add a Meta observation
Edit `renderMeta()` in the META ANALYSIS section. Push to the `observations` array with `{ kind: 'info' | 'ok' | 'warn', text: 'HTML string' }`.

### Bump version
1. Edit `APP_VERSION` and `APP_BUILD` at the top of `<script>`
2. Update `CHANGELOG.md`
3. Commit with message `Release v1.x.y`

---

## Context for Claude Code

When opening this project in Claude Code for the first time, **read `game_log_analyzer.html` end-to-end before making changes** — it's a single file and the architecture only makes sense holistically. Pay particular attention to:

- The shape of the `data` object (everything depends on it)
- How `parseLog()` and `finalizeData()` are split for multi-file support
- The `humanMark()` / `humanPrefix()` / `isHuman()` helpers — every place that renders a faction name should use one of these
- The CSS variables in `:root` — use them, don't hardcode colors

The user (Marc) is technically proficient, prefers concise responses, and works in German. He uses Claude Code on Windows. Keep changes minimal and surgical — this is a working tool, not a playground for refactors.

Good luck. 🖖
