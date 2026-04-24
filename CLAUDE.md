# CLAUDE.md — AI Development Guide

> This file is the primary reference for AI agents working on this codebase.
> It provides the structural skeleton, conventions, and patterns needed to make changes safely.

## Quick Facts

- **App**: STNH Game Log Analyzer v2.7.1
- **File**: `index.html` (~8,190 lines — the ENTIRE app)
- **Dependencies**: Zero. Vanilla HTML + CSS + JS.
- **Hosting**: GitHub Pages (master branch root)
- **Modes**: Log analysis (`.log` files) + Save analysis (`.sav` files)
- **Other files**: `CHANGELOG.md`, `README.md`, `CLAUDE.md` (this file), `HANDOVER.md`, `docs/v*.md` (long-form release notes), `serve.py` (optional local server)

## Architecture Rules

1. Everything lives in `index.html` — no splitting, no modules, no build step
2. Must work from `file://` protocol (no CORS, no imports, no fetch of local files)
3. All save-mode HTML IDs prefixed with `save` to avoid collisions
4. CSS colors ONLY via variables: `--accent`, `--success`, `--bg-card`, `--border`, `--text-primary`, `--text-secondary`, `--text-muted`, `--bg-tertiary`
5. All user strings in innerHTML MUST use `esc()` for XSS safety
6. Any new global state MUST be reset in `resetToUpload()` (currently ~line 7403)
7. Version bump: change `APP_VERSION` + `APP_BUILD` (~line 2574) + `CHANGELOG.md` + in-app "What's New" section (~line 2148)
8. New user-facing features: also add a `help-*` section in the Help overlay + a sidebar link

## Log tabs (11)
Dashboard, Revenue, Income, Stats, Rankings, Wars, Spreadsheet, Faction Detail, Compare, Timeline, Insights, Meta Analysis.

## Recent changes (keep in mind when drafting plans)

- **v2.7.1** (2026-04-24): docs-only — in-app "What's New" + help-income section, updated Revenue/Stats/Log-Format help, `README.md` created.
- **v2.7.0** (2026-04-24): new **Income tab** (surfaces `data.income` parsed in v2.6.0), reusable `createDatePicker()` control, chart gained `mode === 'income'`. `resetToUpload()` now also clears `data.income` (was missed in v2.6.0).
- **v2.6.0** (2026-04-24): parser expanded for Orion's new STH_test_events (fleet power, unemployed pops estimate, influence revenue, 9 major + 17 rare income keys). Added `data.income` stream, derived `unemployment %` in `finalizeData()`, dedup for double-logged income lines.
- **v2.5.0** (2026-04-19): multi-run support + Spreadsheet A/B compare mode. See `docs/v2.5.0-run-comparison.md` for architecture.

Full release notes for v2.6/v2.7: `docs/v2.6-v2.7-orion-events.md`.

## File Structure (Single File)

```
index.html
├── <style>           lines 1–1460       CSS
├── <body>            lines 1462–2570    HTML (11 log tabs, 8 save tabs, help overlay)
│   ├── Header + mode switch
│   ├── File input area + log manager
│   ├── Save progress bar
│   ├── Tab navs (#tabNav for log, #saveTabNav for save)
│   ├── Global filter bar
│   ├── Log tab panels (11 tabs, incl. #tab-income since v2.7.0)
│   ├── Save tab panels (8 tabs)
│   └── Help overlay (includes What's New / help-income since v2.7.1)
└── <script>          lines 2571–8190   JavaScript
    ├── VERSION                          2571
    ├── DATA STRUCTURES                  2586
    ├── DATE UTILITIES                   2733
    ├── DATE PICKER CONTROL              2749  ← new in v2.7.0
    ├── PARSING                          2838
    ├── UI INITIALIZATION                3075
    ├── REVENUE TABLE                    3273
    ├── INCOME (net, after upkeep)       3356  ← new in v2.7.0
    ├── CHART                            3498
    ├── STATS TABLE                      3939
    ├── RANKINGS                         4064
    ├── WARS                             4113
    ├── SPREADSHEET                      4176
    ├── FACTION DETAIL                   4775
    ├── META ANALYSIS                    4923
    ├── HELPERS                          5077
    ├── CATEGORY QUICK-SELECT            5196
    ├── EXPORT HELPERS                   5234
    ├── TAB EXPORT FUNCTIONS             5258
    ├── GLOBAL FILTER STATE              5343
    ├── WAR NETWORK                      5365
    ├── COMPARE TAB                      5545
    ├── TIMELINE TAB                     5642
    ├── INSIGHTS TAB                     5751
    ├── SAVE PARSER — WORKER SOURCE      5927
    ├── SAVE PARSER — WORKER MANAGEMENT  6284
    ├── SAVE PARSER — RENDERING          6357
    ├── SAVE ECONOMY                     6586
    ├── SAVE TECH & TRADITIONS           6651
    ├── SAVE DIPLOMACY                   6724
    ├── SAVE WARS                        6866
    ├── SAVE PARSER — EVENTS & FLAGS     6929
    ├── SAVE PARSER — FILE HANDLING      7370
    ├── MODE SWITCHING                   7392
    ├── TAB SWITCHING                    7544
    ├── HELP OVERLAY                     7754
    ├── INDEXED DB — LOG LIBRARY         7781
    └── FILE LOADING + LOG MANAGER       7844
```

> **Note:** the per-function line numbers in the Function Maps below reflect v2.3.0 and have drifted after v2.5–v2.7. Use the section anchors above + `grep` to locate current lines. The intent/purpose column stays accurate.

## Function Map (123 functions)

### Core / Utilities
| Function | Line | Purpose |
|----------|------|---------|
| `parseDateKey(d)` | 2522 | Parse `YYYY.M.D` → `{y,m,d}` |
| `compareDates(a,b)` | 2527 | Compare two date strings |
| `formatDate(d)` | 2532 | Date key → readable string |
| `esc(s)` | 4201 | HTML escape (XSS safety) |
| `resolveLocalizedName(obj)` | 4206 | Clausewitz `{key:"NAME_..."}` → string |
| `fmtNum(v)` | 4233 | Locale-format number |
| `cwTicksToDate(ticks)` | 4240 | Clausewitz ticks → game date |
| `isHuman(faction)` | 4250 | Check human player |
| `humanMark(faction)` | 4254 | HTML badge for human |
| `humanPrefix(faction)` | 4258 | `[H] ` prefix for dropdowns |
| `getCategoryForFaction(f,d)` | 4262 | Get faction category |
| `categoryMark(f,d)` | 4270 | HTML badge for category |
| `sortFactionsByCategory(factions)` | 4276 | Sort by category priority |
| `exportCSV(filename,headers,rows)` | 4347 | Generic CSV export |
| `exportJSON(filename,obj)` | 4362 | Generic JSON export |
| `isDateInRange(d)` | 4459 | Check global filter range |
| `getVisibleFactions()` | 4465 | Factions matching filters |
| `isFactionVisible(f)` | 4471 | Single faction filter check |

### Log Parsing
| Function | Line | Purpose |
|----------|------|---------|
| `parseLog(text, fileName)` | 2538 | Parse log file → populate `data` |
| `finalizeData()` | 2704 | Sort dates, post-process (call ONCE) |

### UI / Log Tabs
| Function | Line | Purpose |
|----------|------|---------|
| `initUI()` | 2719 | Orchestrate all tab inits |
| `initGlobalFilters()` | 2739 | Setup filter bar |
| `applyGlobalFilters()` | 2760 | Apply filters to all tabs |
| `initDashboard()` | 2792 | Dashboard cards |
| `initRevenue()` | 2918 | Revenue tab |
| `renderRevenue()` | 2931 | Render revenue table |
| `openChart()` | 3007 | Open chart overlay |
| `drawChart()` | 3091 | Canvas chart rendering |
| `initStats()` | 3340 | Stats tab |
| `renderStats()` | 3399 | Render stats table |
| `initRankings()` | 3462 | Rankings tab |
| `renderRankings()` | 3478 | Render rankings |
| `initWars()` | 3514 | Wars tab |
| `renderWars()` | 3538 | Render war list |
| `initSpreadsheet()` | 3574 | Spreadsheet tab |
| `renderSpreadsheet()` | 3639 | Route to sheet view |
| `initFactionDetail()` | 3901 | Faction detail tab |
| `renderFactionDetail()` | 3927 | Render faction view |
| `initMeta()` | 4049 | Meta analysis tab |
| `renderMeta()` | 4053 | Render meta view |
| `toggleWarNetwork()` | 4481 | War network toggle |
| `renderWarNetwork()` | 4490 | Force-directed graph |
| `initCompare()` | 4660 | Compare tab |
| `renderCompare()` | 4683 | Render comparison |
| `initTimeline()` | 4758 | Timeline tab |
| `renderTimeline()` | 4841 | Render timeline |
| `initInsights()` | 4864 | Insights tab |
| `buildInsights()` | 4873 | Generate insights |
| `renderInsights()` | 5014 | Render insight cards |

### Save Parser (Worker)
| Function | Line | Purpose |
|----------|------|---------|
| `buildShallowIndex(text)` | 5211 | Index gamestate sections |
| `parseSection(text, section)` | 5326 | Parse one section |
| `initSaveWorker()` | 5423 | Create Web Worker |
| `handleSaveWorkerMessage(e)` | 5431 | Worker message handler |
| `showSaveError(msg)` | 5455 | Display error |
| `handleSaveFile(file)` | 6482 | Entry point for .sav files |

### Save Rendering
| Function | Line | Purpose |
|----------|------|---------|
| `saveEmpireName(id)` | 5470 | Empire ID → name |
| `extractBudget(c)` | 5475 | Extract country budget |
| `flattenBudget(cat)` | 5483 | Flatten budget category |
| `renderSaveAll()` | 5495 | Render all save tabs |
| `renderSaveOverview()` | 5506 | Overview tab |
| `renderSaveEmpires()` | 5565 | Empires tab (+ builds saveCountryMap) |
| `renderSaveEmpireTable()` | 5649 | Empires data table |
| `renderSaveRawSections()` | 5682 | Raw sections browser |
| `renderSaveEconomy()` | 5704 | Economy tab |
| `renderSaveEconTable()` | 5738 | Economy table |
| `renderSaveTech()` | 5766 | Tech tab |
| `renderSaveTechTable()` | 5806 | Tech table |
| `renderSaveDiplomacy()` | 5837 | Diplomacy tab |
| `renderSaveFederations()` | 5849 | Federation cards |
| `renderSaveDiploContent()` | 5887 | Diplomacy content |
| `renderSaveDiploMatrix(c)` | 5930 | Diplomacy heatmap |
| `renderSaveWars()` | 5981 | Wars tab |
| `renderSaveWarCards()` | 6011 | War cards |
| `extractSaveEventsData()` | 6043 | Extract events/flags data |
| `renderSaveEvents()` | 6158 | Events & Flags tab |
| `renderSaveEventsCurrentView()` | 6192 | Route to sub-view |
| `renderSaveEventChains()` | 6211 | Event chains cards |
| `renderSaveGlobalFlags()` | 6261 | Global flags table |
| `renderSaveCountryFlags()` | 6279 | Country flags (dual mode) |
| `renderSaveVariables()` | 6334 | Variables (dual mode) |
| `renderSaveFiredEvents()` | 6392 | Fired events table |
| `renderSavePlayerEvents()` | 6410 | Player events table |
| `exportSaveEventsCSV()` | 6428 | Events CSV export |

### Mode & Navigation
| Function | Line | Purpose |
|----------|------|---------|
| `updateModeSwitch()` | 6505 | Update mode switch UI |
| `switchMode(mode)` | 6518 | Switch log ↔ save |
| `resetToUpload()` | 6569 | Reset ALL state, return to upload |

### IndexedDB
| Function | Line | Purpose |
|----------|------|---------|
| `openDB()` | 6889 | Open/create IndexedDB |

### File Management
| Function | Line | Purpose |
|----------|------|---------|
| `readFileAsText(file)` | 6956 | Async file reader |
| `renderPendingFiles()` | 7020 | Show staged files |
| `updateLibraryUI()` | 7104 | Refresh log library |
| `updateAnalyzeState()` | 7111 | Enable/disable analyze button |

## Global Variables (62 top-level)

### Mode State
```
currentMode, hasLogData, hasSaveData
```

### Data Constants
```
RESOURCES (13 since v2.6.0 — added 'influence'),
STATS (9 since v2.6.0 — added 'fleet power', 'unemployed pops estimate', 'unemployment %'),
STAT_SHORT, RESOURCE_SHORT,
MAJOR_INCOME (9), RARE_INCOME (18 incl. 'ketracel white'),
INCOME_KEYS, INCOME_SHORT,           ← new in v2.6–v2.7
FACTION_CATEGORIES, CATEGORY_PRIORITY, CATEGORY_SHORT, CATEGORY_COLOR,
SAVE_RESOURCES (12), SAVE_RES_LABELS
```

### Log State
```
data (main object — has .income bucket since v2.6.0),
revSortCol, revSortAsc, revSelected,
incomeSortCol, incomeSortAsc, incomeSelected,
incomeSubView ('major'|'rare'|'all'), incomeDatePickerCtrl,   ← new in v2.7.0
chartState (mode: 'revenue'|'income'|'stats'),
statsSortCol, statsSortAsc, statsSelected,
WAR_PAGE_SIZE, warPage, warNetworkActive, warNetworkAnim,
compareFactions, TL_PAGE_SIZE, tlPage,
globalDateFrom, globalDateTo, globalFileFilter,
sheetScrollHandler, sheetExportCache,
runs[]   ← multi-run (v2.5.0+)
```

### Save State
```
SAVE_WORKER_SOURCE (embedded worker code),
saveWorker, loadedSaveData, saveEmpires, saveCountryMap,
saveEmpireSortKey, saveEmpireSortDir, saveEmpireFilter,
saveEconSortKey, saveEconSortDir, saveEconFilter, saveEconData,
saveTechSortKey, saveTechSortDir, saveTechFilter, saveTechData,
saveDiploEmpire, saveDiploView,
saveWarFilter, saveWarData,
saveEventsView, saveEventsFilter, saveEventsData,
saveEventsSortKey, saveEventsSortDir,
saveCountryFlagMode, saveCountryFlagEmpireId,
saveVarMode, saveVarEmpireId
```

### Library State
```
DB_NAME, DB_VERSION, STORE_NAME,
pendingFileData, librarySelected
```

## HTML ID Map (key IDs)

### Log Navigation
`tabNav`, `newAnalysisBtn`, `logModeUploadBtn`, `helpBtn`

### Save Navigation
`saveTabNav`, `saveNewAnalysisBtn`, `saveModeUploadBtn`

### Mode Switch
`modeSwitch`, `modeSwitchLog`, `modeSwitchSave`

### Tab Panels (Log)
`tab-dashboard`, `tab-revenue`, `tab-stats`, `tab-rankings`, `tab-wars`, `tab-spreadsheet`, `tab-faction`, `tab-compare`, `tab-timeline`, `tab-insights`, `tab-meta`

### Tab Panels (Save)
`tab-save-overview`, `tab-save-empires`, `tab-save-economy`, `tab-save-tech`, `tab-save-diplomacy`, `tab-save-wars`, `tab-save-events`, `tab-save-raw`

### Events & Flags Sub-Views
`saveEventsViewSelect`, `saveEventsSearch`, `saveEventsExportBtn`, `saveEventsCount`, `saveEventsStatGrid`, `saveEventsChains`, `saveEventsGlobalFlags`, `saveGlobalFlagsTbody`, `saveEventsCountryFlags`, `saveCFlagModeToggle`, `saveCFlagEmpireSelect`, `saveCFlagThead`, `saveCFlagTbody`, `saveEventsVariables`, `saveVarModeToggle`, `saveVarEmpireSelect`, `saveVarThead`, `saveVarTbody`, `saveEventsFiredEvents`, `saveFiredEventsTbody`, `saveEventsPlayerEvents`, `savePlayerEventsTbody`

## Data Structures (Save Mode)

### `loadedSaveData.parsed` (from Worker)
```
{
  player:           { __values: [{country: N, name: "..."}] },
  species_db:       { "0": {name: {key: "NAME_..."}, ...}, ... },
  country:          { "0": { name, flags, variables, events, budget, tech_status, ... }, ... },
  war:              { "0": { name, start_date, attackers, defenders, battles, ... }, ... },
  federation:       { "0": { name, type, members, leader, ... }, ... },
  flags:            { "flag_name": { set_date: "...", ... }, ... },
  fired_event_ids:  { __values: ["event.1", "event.2", ...] },
  player_event:     { __values: [{ id, event, country, date }, ...] }
}
```

### `saveCountryMap[id]` (raw country object)
```
{
  name: {key: "NAME_...", variables: [{key: "...", value: "..."}]},
  flags: { "flag_name": 43816536 | {set_date: "...", timed: true}, ... },
  variables: { __values: [{which: "var_name", value: 42.0}, ...] },
  events: {
    event_chain: { __values: [{event_chain: "chain_name", scope: {...}}, ...] },
    completed_event_chain: { __values: ["chain1", "chain2", ...] },
    special_project: { __values: [{type: "project_name", progress: 0, days_left: 30}, ...] }
  },
  budget: { current_month: { income: {...}, expenses: {...} } },
  tech_status: { technology: { __values: [{technology: "tech_name"}, ...] } },
  traditions: { __values: ["tradition_name", ...] },
  government: { type, authority, origin },
  ethos: { ethic: { __values: ["ethic_name", ...] } },
  military_power, economy_power, tech_power, fleet_size, empire_size,
  owned_planets, num_sapient_pops, used_naval_capacity, founder_species_ref
}
```

### Clausewitz Parser Notes
- `__values` array: when a key appears multiple times (Clausewitz allows duplicate keys)
- `__dup` flag: true on arrays that represent duplicate keys
- Tick timestamps: `(ticks - 43791360) / 24` = days from 2200.01.01 (360-day year, 12×30)
- Names are often `{key: "NAME_...", variables: [...]}` objects, not plain strings

## Patterns to Follow

### Adding a save tab with sort/filter
```js
// State
let myTabSortKey = 'name';
let myTabSortDir = 1;
let myTabFilter = '';
let myTabData = [];

// Main render (called from renderSaveAll)
function renderMyTab() {
  // Extract data from loadedSaveData.parsed / saveCountryMap
  myTabData = [...];
  renderMyTabTable();
}

// Table render (called on sort/filter change)
function renderMyTabTable() {
  let rows = myTabData;
  const f = myTabFilter.toLowerCase();
  if (f) rows = rows.filter(r => r.name.toLowerCase().includes(f));
  rows.sort((a, b) => {
    const va = a[myTabSortKey] ?? '', vb = b[myTabSortKey] ?? '';
    return (typeof va === 'string' ? va.localeCompare(vb) : va - vb) * myTabSortDir;
  });
  document.getElementById('myTabCount').textContent = `${rows.length} of ${myTabData.length}`;
  document.getElementById('myTabTbody').innerHTML = rows.map(r => `<tr>...</tr>`).join('');
}

// In renderSaveAll(): add renderMyTab()
// In resetToUpload(): reset myTabSortKey, myTabSortDir, myTabFilter, myTabData
// Event listeners: search input, sort header clicks, export button
```

### Adding a save worker section
```js
// In SAVE_WORKER_SOURCE criticalSections array (~line 5355):
const criticalSections = ['player', 'species_db', 'country', 'war', 'federation',
  'flags', 'fired_event_ids', 'player_event', 'YOUR_NEW_SECTION'];
// Access via: loadedSaveData.parsed.YOUR_NEW_SECTION
```

### Using the reusable date picker (v2.7.0+)
```js
// Scales from 3 to thousands of snapshots without flooding a <select>.
// HTML: <div class="date-picker-control" id="myDatePicker"></div>
const ctrl = createDatePicker(document.getElementById('myDatePicker'), {
  dates: Object.keys(data.income).sort(compareDates),
  initial: latestDate,
  onChange: (newDate) => renderMyTab()
});
ctrl.getValue();                // current date string
ctrl.setValue('2180.1.1');      // programmatic set (no onChange fire)
ctrl.setDates(newDates, init);  // replace the dataset (e.g. after filter change)
```
Sub-controls (all in sync): `<input list>` combobox + ◀/▶ nav buttons + slider (auto-hidden if <10 dates) + span label.

### Adding a new log-parse target
```js
// For new "<faction> <value> <suffix>" style events:
// 1. Add the suffix to RESOURCES / STATS / INCOME_KEYS and a short label to its *_SHORT map.
// 2. If the semantics don't fit those buckets, add a new top-level bucket to `data`
//    (plus freshDataObj(), loadIntoGlobalData(), counts, and resetToUpload()).
// 3. The existing parser loop uses longest-first suffix matching — sort your key list
//    before building the suffix array so multi-word keys (e.g. 'ketracel white') match
//    before their tail tokens.
// 4. Dedupe if the upstream mod double-logs (see the v2.6.0 income branch for the pattern).
```

## CSS Variables (from :root)
```css
--bg-primary: #0f0f23;     --bg-secondary: #1a1a3e;
--bg-card: #16213e;         --bg-tertiary: #1e2a4a;
--text-primary: #e0e0e0;    --text-secondary: #a0a0b0;
--text-muted: #606080;      --border: #2a2a4a;
--accent: #e94560;           --success: #4ecdc4;
```
