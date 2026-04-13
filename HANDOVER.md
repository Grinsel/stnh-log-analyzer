# STNH Game Log Analyzer — Handover

## Project Summary

A single-file, zero-dependency HTML application for analyzing **debug logs** and **save files** from **Stellaris: Star Trek New Horizons (STNH)**. Everything (HTML + CSS + JS) lives in `index.html`. No build step, no server, no external libraries — hosted on GitHub Pages.

**Current version:** `v2.3.0` (build `2026-04-13`)

Version constants at the top of `<script>`:
```js
const APP_VERSION = '2.3.0';
const APP_BUILD   = '2026-04-13';
```

---

## Repository

- **GitHub:** https://github.com/Grinsel/stnh-log-analyzer
- **Live:** https://grinsel.github.io/stnh-log-analyzer/
- **Branch:** `master` (GitHub Pages serves from root)
- **SSH note:** Port 22 blocked — `~/.ssh/config` routes `github.com` to `ssh.github.com:443`

```
stnh-log-analyzer/
├── index.html                     # The whole app (~7,267 lines)
├── serve.py                       # Python local server (stdlib, port 8080)
├── game_log_analyzer.html         # Frozen v1.0.0 reference
├── HANDOVER.md                    # This file
├── CLAUDE.md                      # AI skeleton — architecture map & conventions
├── CHANGELOG.md                   # Per-version notes
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
| **Nav** | `#tabNav` (11 tabs) | `#saveTabNav` (8 tabs) |
| **Filter** | Global date/file bar | Per-tab controls |
| **Switching** | `switchMode('log')` | `switchMode('save')` |

Both modes can be loaded simultaneously. The mode-switch button toggles visibility.

---

## Log Analysis (11 Tabs)

### Input Format
Debug log lines from STNH event scripts:
```
[09:47:04][effect_impl.cpp:22189]: [2210.1.1] Log effect, file: events/STH_test_events.txt line: 42. United Federation of Planets 116.81 energy revenue
```

### Parser (6 Entry Types)
1. **Revenue:** `<faction> <number> <resource> revenue`
2. **Stats:** `<faction> <number> <metric>`
3. **War declarations:** `<faction> entered war against <target>`
4. **Invasions:** `<faction> Invade Win|Fail On <date>: <target> - <planet>`
5. **War status:** `<faction> is at war`
6. **Human player:** `<faction> is human player`

### Tabs
1. **Dashboard** — Aggregate counts, human-player banner, date range
2. **Revenue** — Per-faction resource table, multi-select charting, CSV export
3. **Stats** — Same as Revenue for stat metrics
4. **Rankings** — Top-N empires by metric and date, search, CSV
5. **Wars** — Filterable list, force-directed network graph, CSV
6. **Spreadsheet** — Pivot view with heatmap, virtual scrolling, CSV
7. **Faction Detail** — Per-empire deep dive with search, CSV
8. **Compare** — 2–4 faction side-by-side with sparklines, CSV
9. **Timeline** — Unified event stream (wars + resource changes), CSV
10. **Insights** — Auto-detected anomalies (crashes, spikes, dominance shifts)
11. **Meta Analysis** — Cross-file comparison

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
| 2431 | VERSION |
| 2445 | DATA STRUCTURES |
| 2521 | DATE UTILITIES |
| 2537 | PARSING |
| 2718 | UI INITIALIZATION |
| 2914 | REVENUE TABLE |
| 2997 | CHART |
| 3336 | STATS TABLE |
| 3461 | RANKINGS |
| 3510 | WARS |
| 3573 | SPREADSHEET |
| 3900 | FACTION DETAIL |
| 4048 | META ANALYSIS |
| 4200 | HELPERS |
| 4308 | CATEGORY QUICK-SELECT |
| 4346 | EXPORT HELPERS |
| 4370 | TAB EXPORT FUNCTIONS |
| 4455 | GLOBAL FILTER STATE |
| 4477 | WAR NETWORK |
| 4657 | COMPARE TAB |
| 4754 | TIMELINE TAB |
| 4863 | INSIGHTS TAB |
| 5039 | SAVE PARSER — WORKER SOURCE |
| 5396 | SAVE PARSER — WORKER MANAGEMENT |
| 5469 | SAVE PARSER — RENDERING |
| 5698 | SAVE ECONOMY |
| 5763 | SAVE TECH & TRADITIONS |
| 5836 | SAVE DIPLOMACY |
| 5978 | SAVE WARS |
| 6041 | SAVE PARSER — EVENTS & FLAGS |
| 6482 | SAVE PARSER — FILE HANDLING |
| 6504 | MODE SWITCHING |
| 6647 | TAB SWITCHING |
| 6857 | HELP OVERLAY |
| 6884 | INDEXED DB — LOG LIBRARY |
| 6947 | FILE LOADING + LOG MANAGER |

### Global State

```js
// Log mode
const data = {
  revenue: {},        // { date: { faction: { resource: value } } }
  stats: {},          // { date: { faction: { metric: value } } }
  wars: [],           // [{ date, type, faction, target, ... }]
  allDates: [],       // sorted date strings
  allFactions: new Set(),
  humanPlayers: new Set(),
  files: [],          // per-file metadata
  counts: { revenue, stats, warDecl, invasion, warStatus, human, difficulty, factionCategory },
  difficulty: {},
  factionCategories: {},
  gameVersion: null,
  insights: [],
  timelineEvents: []
};

// Save mode
let loadedSaveData = null;   // { meta, index, parsed, gamestateBytes, gamestateLines }
let saveEmpires = [];        // [{ id, name, species, military_power, ... }]
let saveCountryMap = {};     // empire ID → raw country object
let saveEventsData = null;   // extracted events/flags data
```

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
