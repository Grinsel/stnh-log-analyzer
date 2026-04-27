# Changelog

## v2.10.0 (2026-04-27)

### Added
- **Validation tab — Faction-targeted full check.** New "Target faction" dropdown next to the spot-check sample size. When set, *every* parsed value for that empire is verified back against the raw log (not just a random sample of 20). Lets users hand-verify a specific faction's data exhaustively without manually grepping the log file. The spot-check card header changes to "Spot checks — full coverage for &lt;Faction&gt;" so the mode is obvious.
- **Validation tab — Conflict detection card.** Surfaces every `(date, faction, metric)` tuple where the raw log contains the same entry multiple times with *different values*. This is the silent kind of mismatch users notice when hand-comparing numbers (e.g. "your tool shows 16,749 pops for UFP but the log says 16,649 somewhere"). Shows all distinct values seen and the value the parser actually kept (last-wins for stats/revenue, first-wins for income). Hidden when there are no conflicts.

### Why this release
Reported by Orion1988 (2026-04-27): noticed a ~100 pop mismatch for one faction when comparing analyzer output to the raw log by hand. The existing random spot-check has a small probability of hitting that exact tuple, and even if it did, "first-matching-line" verification can't see that *another* line has a different value. Both new tools target this directly:
- Faction-targeted mode = "verify every single value for UFP".
- Conflict detection = "show me everywhere the upstream mod logged the same thing twice with different numbers, regardless of where I look in the UI".

Tested on Orion's game(1).log: detection finds 27 stats conflicts, including `owned pops` for Hirogen Hunters (5154 vs 0) and `used naval cap percentage` for Klingon Empire (0.75167 vs 0.74172) — likely the source of the mismatch he saw.

## v2.9.3 (2026-04-27)

### Fixed
- **Spreadsheet "Compare runs (A vs B)" option missing after re-analyze** (regression introduced by v2.7.2). The compare option's visibility was only updated in `initSpreadsheet()`, which runs once per page lifetime since the v2.7.2 init-once refactor. So uploading 1 log first, then doing "New Analysis" with 2 logs (or "+ File" to add a 2nd log) left the option hidden — the v2.5.0 feature was effectively unreachable in those workflows. Extracted the visibility logic into `refreshSheetCompareOptVisibility()` and call it from the re-run path and from `resetToUpload()` too. Also: if `runs.length` drops below 2 while the timeframe is set to `compare`, the timeframe falls back to `alltime` so the spreadsheet doesn't try to render an invalid compare view.

## v2.9.2 (2026-04-27)

### Fixed
- **Global filter bar listener-stacking on re-analyze.** `initGlobalFilters()` was called both on first analyze and again on every "New Analysis"-then-re-upload (it has to refill the date and file dropdowns), but its event listeners on the static date/file selects and reset button were being re-bound each time. After three uploads, changing the global filter fired `applyGlobalFilters()` three times. Guarded with a `globalFiltersListenersBound` flag.
- **Income tab stays hidden after re-uploading a log with income data**, when the previous log had none. `initIncome()` set `display: none` on five Income-tab sub-elements when `data.counts.income === 0`, but never set them back to `display: ''` when subsequent runs had income. Fixed: explicit visibility reset on the has-income path.
- **DLC reducer "removed" count broken when `required_dlcs` is a plain string.** The success message used `(meta.required_dlcs?.__values || meta.required_dlcs || []).length` which returned the string's character length for the bare-string case (rare but possible per the parser's fallback shapes). Now mirrors the same shape-tolerant logic used by `renderSaveOverview()`.
- **DLC names with embedded `"` characters** would have produced a malformed Clausewitz block in the exported save. Real Stellaris DLC names don't contain quotes today, but `rewriteRequiredDlcs()` now defensively escapes them so a future weird name doesn't silently corrupt the export.

### Why this release
A precautionary deep-scan after the v2.9.1 Meta fix found four real regressions and two architectural fragilities. False positives (e.g. resetting the various `*ListenersBound` flags on new analysis) were verified and discarded — those flags are intentionally persistent because their listeners are bound to static DOM elements that survive resets.

## v2.9.1 (2026-04-27)

### Fixed
- **Meta Analysis tab showed "1/1" for every faction with 2+ logs loaded.** In multi-run mode (the default since v2.5), each uploaded log goes into its own run bucket and the global `data` only mirrors `runs[0]`. The Meta tab read `data.files` directly, so it only ever saw the first file even when many were loaded. Now reads across `runs[*].data.files` (and `allFactions` / `humanPlayers` likewise) so the presence matrix shows accurate per-file counts (`2/2`, `1/2`, etc.) and the systemic observations actually fire.
- **Single-file observation hint** now also explains the merge-vs-multi-run upload checkbox so users don't repeat the same confusion.

## v2.9.0 (2026-04-26)

### Added
- **Save DLC Reducer & Export** (Save mode) — load any Stellaris `.sav`, uncheck the DLCs you don't own on the Overview tab's new "DLC Reducer & Export" card, and click **Export modified .sav** to download a save that no longer requires those DLCs. Solves the common "I got a save from a friend but it requires DLCs I don't have" problem.
  - Surgically rewrites only the `required_dlcs={ ... }` block in the save's `meta` file. `gamestate` is byte-identical to the original.
  - Repacks as a STORED (uncompressed) ZIP that Stellaris accepts.
  - Filename: `<original>_reduced_dlcs.sav`.
  - Caveat: gamestate isn't touched, so if the save uses DLC-exclusive content (Federations, megastructures, ascension perks added by a removed DLC), the game may still complain. The reducer warns about this in-line and in the help.

### Internal
- Save worker now ships back the original raw `metaText` and `gamestateText` bytes so the main thread can re-pack a modified save without re-reading the file.
- New `buildStoredZip(entries)` helper — minimal CRC-32 + STORED-method ZIP writer (~80 lines, no external library).
- `loadedSaveData.fileName` now stores the original filename for export naming.

## v2.8.0 (2026-04-26)

### Added
- **Validation tab** — answers the dev question *"how do I know the numbers in the analyzer match what's actually in my log?"*. Three independent checks run on demand:
  1. **Counter cross-check** — parser-independent regex counters per pattern (Revenue, Income, Stats, War declarations, Invasions, War status, Human players, Difficulty, Faction categories) compared against the parser's internal counters. Identical = ✓; deviation = ✗ with explicit delta. Income gets special treatment because upstream STH_test_events double-logs each entry — the validator expects raw count ≈ 2× parser count and reports that as "✓ Dedup".
  2. **Spot checks** — picks N random `(date, faction, key, value)` tuples from the parsed dataset and locates each one back in the raw log text. Each ✓ proves the displayed number came from a real source line. N is configurable (5–200, default 20).
  3. **Parse coverage** — fraction of raw log lines recognized as STNH events. A sudden drop between game versions hints at a new event format the parser doesn't handle yet.
- **Source-line modal** — every spot-check row is clickable. Opens a modal showing the exact log line behind the value. The strongest proof you can give a skeptic that the displayed number is correct.
- **Raw log text retention** — `parseLog()` now keeps the original text in `fileRec.rawText` after parsing so the validator has something independent to scan. Memory cost: roughly the size of the loaded log files (4 MB per typical log). Reset on new analysis.

### Why this release
A dev asked: *"How do I know whether the numbers from the log were processed correctly by the analyzer?"* — a fair question with no good answer until now. The Validation tab gives a one-click "trust stamp" that anyone can run themselves, no code reading required. Parser bugs, key-list mismatches, and silent dedup behavior all become visible.

## v2.7.2 (2026-04-26)

### Fixed
- **Listener-stacking on re-analyze**: every time a user clicked "New Analysis" and re-uploaded, `initUI()` re-bound listeners onto static controls (search inputs, date selects, graph buttons, the Income sub-toggle, etc.). After N uploads, every click fired N times. `initUI()` now wires static listeners exactly once per page lifetime via a `uiInitialized` guard; subsequent runs only refresh data-driven UI (date dropdowns, faction dropdowns, derived insights/timeline events, table renders).
- **`initIncome()` is now idempotent**: defensive `incomeListenersBound` flag inside it, so even direct re-calls don't stack the sub-view-toggle listeners.
- **`chartState.resizeObs` leaked across resets**: ResizeObserver was never disconnected when starting a new analysis, leaving an observer attached to a stale chart wrap. `resetToUpload()` now disconnects it and reassigns `chartState` to a fresh object so the chart can't render with cached series/zoom from a prior session.
- **Spreadsheet scroll listener leaked**: `sheetScrollHandler` was nulled out without `removeEventListener`, leaving a dangling listener on `#sheetContainer` after reset. Now removed explicitly. `sheetExportCache` also cleared.
- **Save-mode sort directions persisted across sessions**: `saveEmpireSortDir`, `saveEconSortDir`, `saveTechSortDir` were never reset, so a fresh save-file load could open with the previous file's sort direction inverted. Now reset alongside their key counterparts in `resetToUpload()`.
- **Stale text in search inputs after reset**: `revSearch`, `incomeSearch`, `statsSearch`, `warSearch`, `rankSearch`, `sheetSearch`, `factionSearch` are now cleared when starting a new analysis so a fresh upload doesn't silently filter to zero rows.

### Added
- **Income parser warns on conflicting double-logs**: when STH_test_events emits the same `(date, faction, key)` twice with different values, the parser keeps the first value (existing behavior) but now logs a `console.warn` so divergent values don't get silently dropped.

### Why this release
A deep audit (state-reset coverage, parser edge cases, XSS exposure, listener leaks) found one high-severity bug — listener stacking after multiple re-analyzes — and several minor leaks. Functionally the app worked, but per-action multipliers grew with each re-upload, eventually causing visible misbehavior (sorts firing 5×, sub-toggle bouncing). All fixes are internal — no UI changes.

## v2.7.1 (2026-04-24)

### Documentation
- **In-app patch notes**: new "What's New" section at the top of the `?` help overlay, covering v2.4.0 through v2.7.0 highlights.
- **In-app Income docs**: new `help-income` section with gross-vs-net explanation (Klingons 2180 example), sub-toggle reference, date-picker usage, and requirements note.
- **Updated Revenue / Stats / Log-Format help**: resource list now 13 keys (adds influence), stats list now includes Fleet Power, Unemployed, Unemployment % (derived), and Income (26 keys, major + rare).
- **Fixed** Income-tab "Full docs" link that mistakenly pointed at `help-revenue`.
- **README.md added** for GitHub visitors — project overview, features across both modes, getting-started steps, log-format summary, credits.

## v2.7.0 (2026-04-24)

### Added
- **Income tab** — new tab between Revenue and Stats that surfaces the net-income data parsed in v2.6.0. Mirrors the Revenue tab's layout (search, category quick-select, checkbox-to-graph, CSV export) but reads from `data.income`.
  - **Sub-view toggle** Major (9 cols) / Rare (17 cols) / All (26 cols). Switching columns resets sort.
  - **Chart integration**: selecting empires + "Show Selected Empires Graph" opens the chart overlay in income mode (`chartState.mode = 'income'`), with the metric dropdown filled from the current sub-view. Chart title reads "… Income (net)".
  - **Empty-state**: loading a pre-April-2026 log (no income events) shows a message pointing to the mod version required, instead of a blank table.
- **Reusable date picker control** (`createDatePicker`) that scales from 3 to thousands of snapshots. Replaces the fixed `<select>` dropdown in the Income tab with:
  - searchable type-ahead combobox (`<input list="…">` + `<datalist>`),
  - prev/next navigation buttons (◀ ▶),
  - a range slider (hidden for < 10 snapshots),
  - a span label `2150 — 2180 · N snapshots` for at-a-glance scale.
  - Defensive: invalid combobox input falls back to the last valid value. API: `createDatePicker(container, {dates, onChange, initial})` returns `{getValue, setValue, setDates}`.

### Fixed
- `resetToUpload()` now also resets `data.income` and `data.counts.income` (was missed when the income stream was added in v2.6.0).

### Why this release
v2.6.0 parsed and stored income but never surfaced the numbers anywhere visible — only counts showed up on the Dashboard and Meta tab. The Income tab makes the gross-vs-net delta (e.g. Klingons 2180: +365 energy revenue vs. -67 energy income) inspectable in the UI. The date-picker refactor preempts a usability collapse if/when Russ changes the logging interval from decadal to annual/monthly (which would flood a plain `<select>` with 200–2400 entries).

## v2.6.0 (2026-04-24)

### Added
- **New log metrics** (from Orion1988's latest STH_test_events updates):
  - `fleet power` — overall country military strength as shown in the observer outliner. Parsed as a stat; appears in Stats, Rankings, Chart, Spreadsheet, Faction Detail, Compare, Timeline.
  - `unemployed pops estimate` — AI unemployment figure (ingame-UI differs by tens/hundreds).
  - `unemployment %` — derived in `finalizeData()` as `unemployed / owned pops * 100` per date/faction. Only set when both source values exist and pops > 0.
  - `influence revenue` — added to the RESOURCES family (parses via the existing ` revenue` suffix pipeline).
- **Income stream** (`data.income`, separate from `data.revenue`): parses ` <resource> income` and ` <resource> income AI only` suffixes. Income is NET (after upkeep), revenue is GROSS production — the two diverge meaningfully (e.g. Klingons 2180: +365 energy revenue vs. -67 energy income).
  - Major income keys: `energy`, `mineral`, `food`, `alloy`, `unity`, `engineering`, `physics`, `society`, `dilithium`.
  - Rare income keys: `latinum`, `water`, `brizeen`, `deuterium`, `luxuries`, `cordrazine`, `duranium`, `tallonian`, `kemocite`, `topaline`, `magnesite`, `boronite`, `time_crystal`, `pergium`, `trellium`, `holographic`, `sr_new_horizons`.
  - Duplicate income lines per `(date, faction, key)` are deduplicated — first value wins (upstream STH_test_events currently double-logs some keys).
- **Dashboard** now shows an `Income Entries` card alongside `Revenue Entries`.
- **Meta tab** per-file cards show an `Income: N` counter. Meta observations (empty-file detection, volume variance) now include income in the totals.

### Changed
- `data`, `freshDataObj()`, `loadIntoGlobalData()`, and per-file `counts` extended with an `income` bucket.

## v2.5.0 (2026-04-19)

### Added
- **Run Comparison (A vs B)** in the Spreadsheet tab: when 2+ logs are uploaded without merging, each file becomes a separate "run". The Spreadsheet tab gets a new timeframe option **"Compare runs (A vs B)"** that picks any two runs + a date present in both, then shows per-empire `Run A | Run B | Δ | Δ%` columns plus per-category average rows (Major / Significant / Medium / Minor / etc.). Existing category filter, search, heatmap, and CSV export all work in this mode. Δ heatmap uses red (negative) ↔ green (positive). Helps modders distinguish a real balance change from run-to-run variance.
- **Upload checkbox** "Merge all files into a single run" (appears with 2+ files). Default OFF — files become separate runs. Check it to keep the legacy merge-everything behavior.

### Changed
- `parseLog()` and `finalizeData()` now accept an optional target data bucket so multiple runs can be parsed in parallel without colliding on the global `data` object.

## v2.4.0 (2026-04-19)

### Added
- **Chart**: Optional dashed white **Average line** showing the mathematical mean of all selected empires per date (toggle in the chart toolbar; only appears when 2+ empires are selected). Y-axis range automatically includes the average so the line stays visible.
- **Spreadsheet**: Category filter bar with tri-state buttons for `Major`, `Significant`, `Fallen Empire`, `Static`, `Medium`, `Minor`, and `Human` — click to cycle through include (+) → exclude (−) → off. Multiple categories can be combined; "Clear Filters" resets all. Category badges now also render in the empire-name column for visibility.

## v2.3.2 (2026-04-14)

### Fixed
- **Wiki links**: Tradition and perk links now use `?select=` parameter for exact ID matching instead of `?search=`, which often failed to find the correct entry
- **Wiki**: Tradition `?select=` handler now correctly switches to Traditions tab and opens the tree overlay instead of the generic detail panel

## v2.3.1 (2026-04-13)

### Added
- **Wiki links** (Save): Clickable ↗ icons throughout save analysis that open the STNH Wiki for quick lookup
  - Events & Flags tab: wiki links on fired events, player events, event chains, global flags, country flags, and variables
  - Tech & Traditions tab: wiki links on tradition tree badges and ascension perk badges
  - Links open in a new tab, styled as subtle superscript icons that brighten on hover

## v2.3.0 (2026-04-13)

### Added
- **Events & Flags tab** (Save): New tab showing all active events, event chains, flags, and variables across empires
  - Event Chains view: card layout grouped by chain name with status badges (completed/active/project) and empire badges
  - Global Flags view: sortable table with flag name, type (permanent/timed), set date, expiry
  - Country Flags view: dual mode — flag-centric (cross-empire, showing # empires and badges) or empire-centric (select empire → individual flags)
  - Variables view: dual mode — variable-centric (min/max/avg across empires) or empire-centric (per-empire values)
  - Fired Events view: searchable table of all fired event IDs with prefix extraction
  - Player Events view: pending player events with country names and dates
  - 9 overview stat cards, search filtering, CSV export for all views
- **`cwTicksToDate()` utility**: converts Clausewitz tick timestamps to game dates
- Worker now parses `flags`, `fired_event_ids`, `player_event` sections from save files

## v2.2.0 (2026-04-11)

### Fixed
- **Empire names**: Fixed `[object Object]` bug in Empires tab — Clausewitz save format stores names as localized objects (`{key: "NAME_..."}`) which are now resolved via `resolveLocalizedName()`
- Species names also resolved correctly

### Added
- **Economy tab** (Save): Per-empire resource income breakdown for 12 resources (energy, minerals, food, alloys, consumer goods, unity, 3 research types, influence, dilithium, crew) with net total. Sortable, filterable, CSV export
- **Tech & Traditions tab** (Save): Tech count, tradition count, completed tradition trees, ascension perks per empire with detail badges. Sortable, filterable, CSV export
- **Diplomacy tab** (Save): Relations table (opinion, trust per target empire), heatmap matrix (top 30 empires by power), federation cards (name, type, members, cohesion, leader)
- **Wars tab** (Save): War cards showing name, date, war goal, attacker/defender badges with primary markers, battle count
- **Expanded Empires tab** (Save): New columns — Ethos, Authority, Origin, Pops, Techs, Traditions, Naval Capacity

### Improved
- Worker now parses `war` and `federation` sections for save analysis
- Save mode navigation expanded from 3 to 7 tabs

## v2.0.0 (2026-04-07)

### Added
- **`serve.py`**: Python local development server (stdlib only, default port 8080, auto-opens browser)
- **CSV/JSON Export**: Export buttons on Revenue, Stats, Rankings, Wars, Spreadsheet, Faction Detail, Compare, and Timeline tabs
- **Compare tab**: Side-by-side comparison of 2–4 factions with revenue/stats tables and sparkline overlays
- **Timeline tab**: Chronological unified event stream combining wars and detected resource spikes/drops (>30% change), paginated and filterable
- **Insights tab**: Auto-detected trends — resource crashes (>50% drop), naval cap spikes, vanished factions, war frequency spikes, economic dominance shifts — with severity-coded cards
- **Global date filter**: "From" / "To" date selects in a persistent bar below tab navigation, affecting all tabs
- **File-level filter**: Dropdown to scope all tabs to data from a single loaded file
- **Unified search**: Search box added to Rankings and Faction Detail (dropdown filter) tabs
- **War Network visualization**: Force-directed graph in Wars tab showing faction war relationships (node size = involvement, edge thickness = war count, click to filter)
- **Log Library rename**: Click the edit icon next to any saved log name to rename it inline
- **Storage quota warning**: Log Library shows storage usage and warns when approaching browser limits

### Improved
- **Spreadsheet virtual scrolling**: Datasets with 100+ factions now render only visible rows for smooth performance
- **Chart zoom/pan**: Mouse wheel zooms the date range in the chart overlay
- **Chart vertical hover line**: Hovering shows all faction values at the nearest date with a dashed vertical guide
- **Chart Y-axis**: Smart tick calculation using "nice numbers" instead of linear division
- **Chart X-axis**: Label thinning when many dates to prevent overlap
- **Tab help texts**: All descriptions updated with info about new features (export, compare, timeline, insights, filters, network, zoom)

## v1.2.0 (2026-04-07)

### Added
- **Usage hints on every tab**: Each tab now shows a collapsible "How to use" help text explaining controls, features, and multi-log behavior
- **Log Manager help texts**: Upload area and Log Library both have inline usage instructions
- All help texts are collapsible via `<details>` — click to hide once you know the tool

## v1.1.0 (2026-04-07)

### Added
- **Log Library**: Uploaded logs are now persistently stored in the browser (IndexedDB) and can be re-used across sessions
- Each upload is saved with timestamp and an optional custom name
- Saved logs are listed in a library panel alongside the upload area
- Users can select one or multiple saved logs for analysis without re-uploading
- Logs can be individually or bulk-deleted from the library
- **Online hosting**: App is now available via GitHub Pages — no need to download the HTML file
- **New Analysis** button in tab navigation to return to log selection and start fresh
- `index.html` entry point for GitHub Pages deployment

### Changed
- Upload flow redesigned: files are staged first, then analyzed via explicit "Analyze" button
- Upload area now side-by-side with log library on wider screens
- Version bumped to 1.1.0

## v1.0.0 (2026-04-07)

- Initial release
- Dashboard, Revenue, Stats, Rankings, Wars, Spreadsheet, Faction Detail, Meta Analysis tabs
- Multi-file support with cross-file comparison
- Human player detection and highlighting
- Custom canvas chart for revenue/stats comparison
