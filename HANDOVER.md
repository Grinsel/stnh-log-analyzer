# STNH Game Log Analyzer — Handover

## Project Summary

A single-file, zero-dependency HTML application for analyzing **debug logs** and **save files** from **Stellaris: Star Trek New Horizons (STNH)**. Everything (HTML + CSS + JS) lives in `index.html`. No build step, no server, no external libraries — hosted on GitHub Pages.

**Current version:** `v2.7.1` (build `2026-04-24`)

Version constants at the top of `<script>`:
```js
const APP_VERSION = '2.7.1';
const APP_BUILD   = '2026-04-24';
```

> This document was written against v2.3.0 and is kept updated at the level of
> "current state + what changed". For per-version implementation details see
> `CHANGELOG.md` and the long-form release notes under `docs/`
> (e.g. `docs/v2.5.0-run-comparison.md`, `docs/v2.6-v2.7-orion-events.md`).

---

## Repository

- **GitHub:** https://github.com/Grinsel/stnh-log-analyzer
- **Live:** https://grinsel.github.io/stnh-log-analyzer/
- **Branch:** `master` (GitHub Pages serves from root)
- **SSH note:** Port 22 blocked — `~/.ssh/config` routes `github.com` to `ssh.github.com:443`

```
stnh-log-analyzer/
├── index.html                     # The whole app (~8,190 lines as of v2.7.1)
├── README.md                      # GitHub landing page (added v2.7.1)
├── serve.py                       # Python local server (stdlib, port 8080)
├── game_log_analyzer.html         # Frozen v1.0.0 reference
├── HANDOVER.md                    # This file
├── CLAUDE.md                      # AI skeleton — architecture map & conventions
├── CHANGELOG.md                   # Per-version notes
├── docs/                          # Long-form release notes per major release
│   ├── v2.5.0-run-comparison.md
│   └── v2.6-v2.7-orion-events.md
├── .gitignore                     # Ignores *.sav, samples/, *.log
└── saves/
    ├── save_parser_poc.html       # Clausewitz parser PoC (1,061 lines)
    └── HANDOVER_SAVE_PARSER.md    # PoC documentation
```

---

## Dual-Mode Architecture (v2.2.0+)

The app has two independent analysis modes controlled by `currentMode`:

| | Log Mode | Save Mode |
|---|---|---|
| **Input** | `.log` debug files | `.sav` Stellaris saves (ZIP) |
| **Parser** | `parseLog()` + `finalizeData()` | Web Worker (`SAVE_WORKER_SOURCE`) |
| **State** | `data` object | `loadedSaveData`, `saveCountryMap`, `saveEmpires` |
| **Nav** | `#tabNav` (12 tabs) | `#saveTabNav` (8 tabs) |
| **Filter** | Global date/file bar | Per-tab controls |
| **Switching** | `switchMode('log')` | `switchMode('save')` |

Both modes can be loaded simultaneously. The mode-switch button toggles visibility.

---

## Log Analysis (12 Tabs)

### Input Format
Debug log lines from STNH event scripts:
```
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 42. United Federation of Planets 116.81 energy revenue
```

### Parser (8 Entry Types)
1. **Revenue** (gross): `<faction> <number> <resource> revenue` — 13 resources (includes `influence` since v2.6.0)
2. **Income** (net, after upkeep, v2.6.0+): `<faction> <number> <key> income [AI only]` — 9 major + 17 rare keys + `ketracel white`
3. **Stats:** `<faction> <number> <metric>` — 8 metrics (includes `fleet power`, `unemployed pops estimate` since v2.6.0); `unemployment %` derived in `finalizeData()`
4. **Difficulty:** `Game difficulty is <level>`
5. **Faction category:** `<faction> is <major_faction|significant_power|sth_fallen_empire|static_empire|sth_medium_country|sth_minor_country>`
6. **War declarations:** `<faction> entered war against <target>`
7. **Invasions:** `<faction> Invade Win|Fail On <date>: <target> - <planet>`
8. **War status:** `<faction> is at war`
9. **Human player:** `<faction> is human player` (matched broadly across all STH_*.txt files)

Income lines are **deduplicated** per `(date, faction, key)` — first value wins, because upstream STH_test_events emits each key twice per date.

### Tabs
1. **Dashboard** — Aggregate counts (incl. `Income Entries` since v2.6.0), human-player banner, date range
2. **Revenue** — Per-faction gross production, multi-select charting, CSV export
3. **Income** (v2.7.0+) — Per-faction net income after upkeep, sub-toggle Major/Rare/All, scalable date picker, charting, CSV
4. **Stats** — Same as Revenue for stat metrics
5. **Rankings** — Top-N empires by metric and date, search, CSV
6. **Wars** — Filterable list, force-directed network graph, CSV
7. **Spreadsheet** — Pivot view with heatmap, virtual scrolling, Run-A-vs-Run-B compare (v2.5.0+), CSV
8. **Faction Detail** — Per-empire deep dive with search, CSV
9. **Compare** — 2–4 faction side-by-side with sparklines, CSV
10. **Timeline** — Unified event stream (wars + resource changes), CSV
11. **Insights** — Auto-detected anomalies (crashes, spikes, dominance shifts)
12. **Meta Analysis** — Cross-file comparison, Income counters per file

### Global Features
- **Date range filter** + **file filter** in persistent bar
- **Log Library** with IndexedDB persistence
- **Chart**: custom canvas with zoom/pan, hover crosshair, smart Y-axis

---

## Save Analysis (8 Tabs)

### Parser Pipeline
1. File dropped → ZIP magic bytes detected in `stageFiles()`
2. `handleSaveFile()` → `initSaveWorker()` creates Web Worker from `SAVE_WORKER_SOURCE`
3. Worker: unzip → parse meta → build shallow index → parse critical sections
4. Worker posts `{type: 'loaded', meta, index, parsed, gamestateBytes, gamestateLines}`
5. `renderSaveAll()` renders all tabs

### Critical Sections (Worker)
`player`, `species_db`, `country`, `war`, `federation`, `flags`, `fired_event_ids`, `player_event`

### Tabs
1. **Overview** — Save metadata, DLC list, player list, section counts
2. **Empires** — Sortable table: name, species, government, ethos, power, planets, pops, techs
3. **Economy** — Per-empire income for 12 resources with net totals
4. **Tech & Traditions** — Tech/tradition counts, trees, ascension perks
5. **Diplomacy** — Relations table, heatmap matrix (top 30), federation cards
6. **Wars** — War cards with attacker/defender badges, battle counts
7. **Events & Flags** — 6 sub-views:
   - Event Chains (cards grouped by chain, status badges)
   - Global Flags (sortable table, permanent/timed)
   - Country Flags (dual mode: flag-centric cross-empire / empire-centric)
   - Variables (dual mode: variable-centric with stats / empire-centric)
   - Fired Events (searchable by prefix)
   - Player Events (pending events with country/date)
8. **Raw Sections** — Index browser for all gamestate sections

### Key Save Functions
- `saveEmpireName(id)` — resolve empire ID → name
- `resolveLocalizedName(obj)` — Clausewitz `{key:"NAME_..."}` → string
- `extractBudget(c)` / `flattenBudget(cat)` — budget extraction
- `cwTicksToDate(ticks)` — Clausewitz timestamps → game dates
- `extractSaveEventsData()` — builds cross-reference maps for events tab

---

## Code Architecture

### Section Map (search for `// ==========`)

| Line | Section |
|------|---------|
| 2571 | VERSION |
| 2586 | DATA STRUCTURES |
| 2733 | DATE UTILITIES |
| 2749 | **DATE PICKER CONTROL** *(new in v2.7.0)* |
| 2838 | PARSING |
| 3075 | UI INITIALIZATION |
| 3273 | REVENUE TABLE |
| 3356 | **INCOME** *(new in v2.7.0 — net, after upkeep)* |
| 3498 | CHART *(mode: `'revenue' \| 'income' \| 'stats'` since v2.7.0)* |
| 3939 | STATS TABLE |
| 4064 | RANKINGS |
| 4113 | WARS |
| 4176 | SPREADSHEET |
| 4775 | FACTION DETAIL |
| 4923 | META ANALYSIS |
| 5077 | HELPERS |
| 5196 | CATEGORY QUICK-SELECT |
| 5234 | EXPORT HELPERS |
| 5258 | TAB EXPORT FUNCTIONS |
| 5343 | GLOBAL FILTER STATE |
| 5365 | WAR NETWORK |
| 5545 | COMPARE TAB |
| 5642 | TIMELINE TAB |
| 5751 | INSIGHTS TAB |
| 5927 | SAVE PARSER — WORKER SOURCE |
| 6284 | SAVE PARSER — WORKER MANAGEMENT |
| 6357 | SAVE PARSER — RENDERING |
| 6586 | SAVE ECONOMY |
| 6651 | SAVE TECH & TRADITIONS |
| 6724 | SAVE DIPLOMACY |
| 6866 | SAVE WARS |
| 6929 | SAVE PARSER — EVENTS & FLAGS |
| 7370 | SAVE PARSER — FILE HANDLING |
| 7392 | MODE SWITCHING |
| 7544 | TAB SWITCHING |
| 7754 | HELP OVERLAY |
| 7781 | INDEXED DB — LOG LIBRARY |
| 7844 | FILE LOADING + LOG MANAGER |

### Global State

```js
// Log mode
const data = {
  revenue: {},        // { date: { faction: { resource: value } } }  — GROSS production
  income:  {},        // { date: { faction: { key: value } } }        — NET, v2.6.0+
  stats:   {},        // { date: { faction: { metric: value } } }     — includes derived 'unemployment %'
  wars: [],           // [{ date, type, faction, target, ... }]
  allDates: [],       // sorted date strings
  allFactions: new Set(),
  humanPlayers: new Set(),
  files: [],          // per-file metadata (counts also include income)
  counts: { revenue, income, stats, warDecl, invasion, warStatus, human, difficulty, factionCategory },
  difficulty: {},
  factionCategories: {},
  gameVersion: null,
  insights: [],
  timelineEvents: []
};

// Income tab state (v2.7.0+)
let incomeSortCol = 0, incomeSortAsc = true;
const incomeSelected = new Set();
let incomeSubView = 'major';          // 'major' | 'rare' | 'all'
let incomeDatePickerCtrl = null;      // { getValue, setValue, setDates } from createDatePicker()

// Multi-run (v2.5.0+)
let runs = [];                        // [{ id, name, data: <freshDataObj() bucket> }]

// Save mode
let loadedSaveData = null;   // { meta, index, parsed, gamestateBytes, gamestateLines }
let saveEmpires = [];        // [{ id, name, species, military_power, ... }]
let saveCountryMap = {};     // empire ID → raw country object
let saveEventsData = null;   // extracted events/flags data
```

### Reusable Date Picker (v2.7.0+)

`createDatePicker(container, { dates, onChange, initial })` returns `{ getValue, setValue, setDates }`.
Injects a combobox + ◀/▶ nav + slider (hidden if < 10 dates) + span label. All four sub-controls stay in sync. Used by the Income tab; can be dropped into any tab where the plain `<select>` would struggle with thousands of entries.

---

## Conventions

- **No external dependencies.** Must work from `file://` and GitHub Pages.
- **`esc()`** all user data in innerHTML — STNH names can contain quotes/ampersands.
- **CSS variables** for colors (`--accent`, `--success`, `--bg-card`, etc.) — never hardcode hex.
- **HTML ID prefix** `save` for all save-mode elements (avoids collisions).
- **German umlauts** as real characters (ä ö ü ß), never ae/oe/ue/ss.
- **Semver versioning** — bump `APP_VERSION` + `APP_BUILD` + CHANGELOG.md.
- **`resetToUpload()`** — any new global state MUST be reset here.

---

## Common Tasks

### Add a new log tab
1. Add `<button>` to `#tabNav` (before `logModeUploadBtn`)
2. Add `<div class="tab-panel" id="tab-yourname">` in `#mainContent`
3. Write `initYourTab()`, add to `initUI()`
4. Tab switching is automatic via `data-tab` attribute

### Add a new save tab
1. Add `<button>` to `#saveTabNav` (before `saveModeUploadBtn`)
2. Add `<div class="tab-panel" id="tab-save-yourname">` after existing save panels
3. Write `renderSaveYourTab()`, add to `renderSaveAll()`
4. Reset state in `resetToUpload()`
5. Add event listeners after the save war search listener block

### Add a new save worker section
Add the section name to `criticalSections` array in `SAVE_WORKER_SOURCE` (~line 5355).

### Bump version
1. Edit `APP_VERSION` and `APP_BUILD` (~line 2431)
2. Update `CHANGELOG.md`
3. Commit + push (GitHub Pages auto-deploys)

---

## Test Procedure

### Log Mode
1. `python serve.py` → browser opens to `localhost:8080`
2. Upload `.log` file → staged → Analyze
3. Dashboard: correct human players, counts
4. Revenue/Stats: human highlighting, CSV export
5. Global filter: date range + file filter affect all tabs
6. Chart: zoom, hover crosshair
7. Log Library: save, rename, delete, reload
8. "← New Analysis" returns to upload

### Save Mode
1. Drop `.sav` file → progress bar → tabs appear
2. Overview: metadata, DLC, players
3. Empires: sort, filter, CSV export
4. Economy: 12 resources, sort, filter
5. Tech: tech counts, tradition trees, CSV
6. Diplomacy: relations table + heatmap matrix + federation cards
7. Wars: war cards with attacker/defender badges
8. Events & Flags: 6 sub-views (chains, global flags, country flags dual-mode, variables dual-mode, fired events, player events), search, CSV
9. Mode switch: toggle between log/save views
10. "← New Analysis" resets everything

---

## Context for AI

When modifying this project:
- Read the relevant section of `index.html` before editing — architecture makes sense holistically
- Use `CLAUDE.md` for the structural skeleton (function map, ID map, section locations)
- The `data` object shape is critical for log mode
- The `saveCountryMap` / `loadedSaveData.parsed` structure is critical for save mode
- `humanMark()` / `humanPrefix()` / `isHuman()` — use in every faction render
- `esc()` — every innerHTML injection
- CSS variables from `:root` — never hardcode colors
- `resetToUpload()` — reset ALL new state here

The user (Marc / Grinsel) is technically proficient, prefers concise responses, and works in German on Windows. Keep changes minimal and surgical.

Good luck. 🖖
