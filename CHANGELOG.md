# Changelog

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
