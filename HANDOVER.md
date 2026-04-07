# STNH Game Log Analyzer — Handover

## Project Summary

A single-file, zero-dependency HTML application for analyzing debug logs produced by **Stellaris: Star Trek New Horizons (STNH)** mod games. Everything (HTML + CSS + JS) lives in `index.html`. No build step, no server, no external libraries — hosted on GitHub Pages at **https://grinsel.github.io/stnh-log-analyzer/**.

**Current version:** `v1.1.0` (build `2026-04-07`)

Version constants are defined once at the top of the `<script>` block:

```js
const APP_VERSION = '1.1.0';
const APP_BUILD   = '2026-04-07';
```

The version is auto-rendered into the document title, the header badge (top right), and the browser console on load. **Always bump these when releasing changes.**

---

## Repository

- **GitHub:** https://github.com/Grinsel/stnh-log-analyzer
- **Live:** https://grinsel.github.io/stnh-log-analyzer/
- **Branch:** `master` (GitHub Pages serves from root of `master`)
- **SSH note:** Port 22 is blocked on Marc's network. `~/.ssh/config` routes `github.com` to `ssh.github.com:443`.

```
stnh-log-analyzer/
├── index.html                # The whole app (served by GitHub Pages)
├── game_log_analyzer.html    # Original v1.0.0 (kept for reference)
├── HANDOVER.md               # This file
├── CHANGELOG.md              # Per-version notes
└── .gitignore                # Ignores samples/, *.log
```

No `package.json`, no bundler, no framework. Keep it that way — the "single HTML file" property is a feature.

---

## What the App Does

### Input
A debug log file (`game.log` or similar) emitted by STNH event scripts. Relevant lines look like:

```
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 1607. Antedean Shoals is human player
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 42. United Federation of Planets 116.81 energy revenue
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 99. Klingon Empire entered war against Romulan Star Empire
```

### Parser Recognizes Six Entry Types
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

### Log Library (v1.1.0)
The landing page shows a two-panel layout:

- **Left panel:** Upload area (drag & drop / click). Staged files shown with option to name them and save to library.
- **Right panel:** Log Library — all previously saved logs listed by name, upload timestamp, and file size. Checkboxes for multi-select.

Users can:
- Upload new files and optionally save them to the library (IndexedDB)
- Select one or multiple saved logs from the library
- Combine new uploads with saved logs in a single analysis
- Delete individual or bulk-selected logs from the library

The **"Analyze"** button triggers parsing of all selected/staged files. After analysis, a **"New Analysis"** button in the tab bar resets all state and returns to the log selection screen.

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

It's a single ~2910-line HTML file. The `<script>` block is organized into clearly commented sections (search the file for `// ==========`):

| Section | Line (approx) | Purpose |
|---|---|---|
| `VERSION` | 1117 | `APP_VERSION` and `APP_BUILD` constants |
| `DATA STRUCTURES` | 1131 | The global `data` object + `RESOURCES`, `STATS`, label maps |
| `DATE UTILITIES` | 1177 | Parse/compare/format `YYYY.M.D` strings |
| `PARSER` | 1193 | `parseLog(text, fileName)` + `finalizeData()` |
| `UI INITIALIZATION` | 1340 | `initUI()` orchestrator + `init*()` per tab |
| `REVENUE TABLE` | 1443 | Table + multi-select + graph trigger |
| `CHART` | 1529 | Custom canvas line chart used by Revenue and Stats |
| `STATS TABLE` | 1758 | Same shape as Revenue |
| `RANKINGS` | 1879 | Top-N table |
| `WARS` | 1924 | Filterable paginated list |
| `SPREADSHEET` | 1985 | Pivot table with multiple modes |
| `FACTION DETAIL` | 2276 | Per-empire deep dive |
| `META ANALYSIS` | 2399 | Cross-file tab |
| `HELPERS` | 2549 | `esc`, `fmtNum`, `isHuman`, `humanMark`, `humanPrefix` |
| `TAB SWITCHING` | 2572 | Tab button click handler + "New Analysis" reset |
| `INDEXED DB — LOG LIBRARY` | 2613 | `openDB`, `dbGetAll`, `dbPut`, `dbDelete`, `dbGet` |
| `FILE LOADING + LOG MANAGER` | 2676 | Stage files, render library, analyze button, all library UI logic |

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

### IndexedDB Schema

- **Database:** `stnh_log_analyzer`, version 1
- **Object store:** `logs`, auto-increment `id` key
- **Record shape:** `{ id, name, uploadedAt (timestamp ms), content (raw text), size (bytes) }`

Helper functions: `openDB()`, `dbGetAll()`, `dbPut(record)`, `dbDelete(id)`, `dbGet(id)` — all return Promises.

### Log Manager State

```js
let pendingFileData = [];        // [{ name, content, size }] — staged uploads not yet analyzed
let librarySelected = new Set(); // Set of IndexedDB log IDs selected for analysis
```

Key functions:
- `stageFiles(fileList)` — reads File objects into `pendingFileData`, does NOT parse yet
- `renderPendingFiles()` — shows staged files with remove buttons
- `renderLogLibrary()` — reads all logs from IndexedDB, renders checkbox list
- `updateAnalyzeState()` — enables/disables Analyze button, shows count summary
- `deleteLog(id)` — removes a single log from IndexedDB
- The `analyzeBtn` click handler combines pending + library-selected, optionally saves new uploads, then calls `parseLog` + `finalizeData` + `initUI`
- `newAnalysisBtn` click handler resets all data and returns to log selection screen

### Helper Functions
- `isHuman(faction)` → boolean
- `humanMark(faction)` → returns ` <span class="human-badge">HUMAN</span>` HTML or empty string
- `humanPrefix(faction)` → returns `'[H] '` or empty string (for `<option>` text)
- `esc(s)` → HTML escape. **Always use** when injecting faction names or any user data into innerHTML.
- `fmtNum(v)` → locale-formatted number with 2 decimal places.
- `formatFileSize(bytes)` → human-readable file size (B/KB/MB).
- `formatTimestamp(ts)` → German locale date+time string.

---

## Conventions

- **No external dependencies.** Vanilla JS, vanilla CSS, no fonts from CDNs, no build step. The file must remain openable from `file://` as well as hosted.
- **Always escape faction names** with `esc()` when building HTML strings — STNH faction names can contain quotes, ampersands, etc.
- **CSS variables** for colors (`var(--accent)`, `var(--success)`, etc.) — defined in `:root` at the top of the stylesheet. Don't hardcode hex colors in new components.
- **Theme:** Dark theme with red accent (`#e94560`) and teal success color (`#4ecdc4`) — matches the broader STNH wiki aesthetic.
- **German umlauts** in any user-facing German text must be the real characters (ä ö ü ß), never `ae oe ue ss`. The current UI is English, but if German strings are added later, this rule applies.
- **Bump `APP_VERSION`** for any user-visible change. Use semver:
  - PATCH (1.1.1): bug fixes, visual tweaks
  - MINOR (1.2.0): new features, new tabs, new heuristics
  - MAJOR (2.0.0): breaking parser/data-format changes
- **Update `APP_BUILD`** to the release date (ISO format `YYYY-MM-DD`).
- **Add a `CHANGELOG.md` entry** for every version bump.
- **Commit + push** after every version bump so GitHub Pages updates.

---

## Known Limitations / Future Ideas

1. **Parser regex is `STH_test_events.txt`-locked** for everything except human-player detection. Loosen to `STH_*.txt` if logs from other event files need to be parsed.
2. **IndexedDB is per-browser.** Logs saved in Chrome are not visible in Firefox. No cross-device sync.
3. **No CSV/JSON export.** Useful for further analysis in Excel/Python.
4. **Spreadsheet tab can be slow** with 150+ factions × many dates. Could virtualize rows.
5. **Chart is a custom canvas implementation** — works but is bare-bones. Tooltips, zoom, axis customization could all be improved.
6. **Meta Analysis observations are hand-rolled heuristics.** Could add more (e.g., factions that disappear mid-game, sudden resource crashes, war frequency spikes).
7. **No light theme.** The broader STNH wiki project has plans for half-dark and light themes.
8. **Faction Detail war history** isn't paginated and could blow up with very long games.
9. **No file-level filtering in main tabs.** When multiple files are loaded, you can't yet view just one file's data in the Revenue/Stats/etc tabs without starting a new analysis.
10. **No tests.** Manual QA only.
11. **Log Library has no rename/edit.** Once a log is saved, its name can't be changed.
12. **No storage quota warning.** Large log files could fill up IndexedDB without user feedback.

---

## Test Procedure

1. Open https://grinsel.github.io/stnh-log-analyzer/ (or `index.html` locally).
2. **Upload test:** Drag a `.log` file onto the drop zone. Verify it appears in "Staged for Analysis".
3. **Naming:** Enter a custom name, keep "Save to Log Library" checked, click Analyze.
4. **Dashboard:** Verify correct human player(s) in green banner, correct counts.
5. **Revenue/Stats/Rankings tabs:** Human empire row should be highlighted (green, badge, left border).
6. **Faction Detail:** Should default to a human player.
7. **New Analysis:** Click "← New Analysis" in tab bar. Verify return to log selection.
8. **Library persistence:** The previously uploaded log should appear in the Log Library. Select it and click Analyze — should work without re-uploading.
9. **Multi-select:** Select multiple saved logs, click Analyze. Meta Analysis tab should show cross-file data.
10. **Delete:** Delete a log from the library. Verify it's gone after page reload.
11. **Console:** Verify version banner is logged. Header badge should show `v1.1.0`.

---

## Common Tasks

### Add a new resource or stat
Edit `RESOURCES` / `STATS` arrays and the corresponding `RESOURCE_SHORT` / `STAT_SHORT` label maps in the `DATA STRUCTURES` section (~line 1131).

### Add a new tab
1. Add a `<button>` to `<nav class="tab-nav">` (after the existing tab buttons, NOT before `newAnalysisBtn`)
2. Add a `<div class="tab-panel" id="tab-yourname">` after the existing panels
3. Write `initYourTab()` and add it to `initUI()`
4. Tab switching is handled automatically by the existing `.tab-btn[data-tab]` click listener

### Add a Meta observation
Edit `renderMeta()` in the META ANALYSIS section. Push to the `observations` array with `{ kind: 'info' | 'ok' | 'warn', text: 'HTML string' }`.

### Bump version
1. Edit `APP_VERSION` and `APP_BUILD` at the top of `<script>` (~line 1118)
2. Update the `<title>` tag and `<b id="appVersion">` in the HTML header (auto-synced on load, but keep markup in sync)
3. Update `CHANGELOG.md`
4. Commit with message `Release v1.x.y`
5. Push to `master` — GitHub Pages auto-deploys

### Reset / New Analysis flow
The "New Analysis" button (`newAnalysisBtn`) resets the global `data` object, clears `revSelected`/`statsSelected`/`pendingFileData`/`librarySelected`, hides tabs, shows the file input area, and re-renders the log library. If you add new global state, remember to reset it here too.

### Modify IndexedDB schema
If you need to add fields to the log records (e.g., tags, parsed metadata), bump `DB_VERSION` in the `INDEXED DB` section and handle the migration in `onupgradeneeded`. Existing records will need to be migrated or the user will lose their library.

---

## Context for Claude Code

When opening this project in Claude Code for the first time, **read `index.html` end-to-end before making changes** — it's a single file and the architecture only makes sense holistically. Pay particular attention to:

- The shape of the `data` object (everything depends on it)
- How `parseLog()` and `finalizeData()` are split for multi-file support
- The `humanMark()` / `humanPrefix()` / `isHuman()` helpers — every place that renders a faction name should use one of these
- The CSS variables in `:root` — use them, don't hardcode colors
- The log manager flow: `stageFiles` → `renderPendingFiles` → analyze button → `parseLog` + `finalizeData` + `initUI`
- The "New Analysis" reset handler — any new global state must be reset there

The user (Marc / Grinsel) is technically proficient, prefers concise responses, and works in German. He uses Claude Code on Windows. Keep changes minimal and surgical — this is a working tool, not a playground for refactors.

Good luck. 🖖
