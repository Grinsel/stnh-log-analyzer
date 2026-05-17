# STNH Log Pattern Inventory

> What information the STNH game.log emits, and how much of it the analyzer
> currently extracts.

**Source log analyzed:** `ai_without_new_horizonsgame.log` (62 MB, 424 028 lines, ~50 game-years of an AI-vs-AI playthrough — the largest real test log we have).

**Extraction summary:**

| Metric | Value |
|---|---|
| Total log lines | 424 028 |
| Structured `Log effect/trigger` lines (STNH events) | 411 085 |
| Engine-level / unstructured lines | 12 943 |
| **Lines parsed by the analyzer** | **405 039 (98.5 % of structured events)** |
| Unparsed structured events | 6 046 (1.5 %) |

The 12.9 k unstructured lines are pure Stellaris engine output (galaxy generation, save-load, mod-load banner, system inits, etc.) and contain no game-state information. They are intentionally ignored.

---

## ✅ PARSED — fully consumed by the analyzer

### Resource revenue (gross production, 13 resources)

`<faction> <number> <resource> revenue` — emitted from `events/STH_test_events.txt` lines 1596–1629, 1667.

| Resource | Lines | Where surfaced |
|---|---:|---|
| `unity` | 19 525 | Revenue, Stats (chart), Spreadsheet, Compare, Faction Detail, Insights, Galaxy (size mode) |
| `alloy` | 19 514 | same |
| `mineral` | 19 513 | same |
| `crew` | 19 512 | same |
| `dilithium` | 19 502 | same |
| `energy` | 19 479 | same |
| `food` | 19 479 | same |
| `consumer goods` | 19 478 | same |
| `physics_research` | 19 478 | same |
| `influence` | 19 478 | same |
| `engineering research` | 19 477 | same |
| `society research` | 19 477 | same |
| `ketracel white` | 240 | same |

**Total: ~253 000 revenue events parsed.**

### Stats (per-empire snapshots, 8 metrics)

`<faction> <number> <stat>` — emitted from `events/STH_test_events.txt` lines 1632–1653.

| Stat | Lines | Surfaced in |
|---|---:|---|
| `fleet power` | 19 813 | Stats, Rankings, Galaxy (Military mode), Faction Detail, Compare, Insights |
| `researched technologies` | 19 690 | same |
| `used naval cap percentage (0.0-1.0)` | 19 595 | same |
| `used naval cap` | 19 568 | same |
| `owned pops` | 19 479 | Stats, Rankings, Galaxy (Population mode), … |
| `owned planets` | 19 477 | same |
| `tradition categories picked` | 19 476 | same |
| `unemployed pops estimate` | 19 410 | Stats, … (also derives `unemployment %`) |

**Plus derived metric:** `unemployment %` = `unemployed / pops × 100` — computed in `finalizeData()`.

### Income (net, after upkeep — 26 keys)

`<faction> <number> <key> income` (with optional ` AI only` suffix) — Russ' addition from April 2026; **the test log analyzed here predates that and contains zero income lines**. The parser supports 26 income keys when present (9 major + 17 rare resources).

### Wars

| Pattern | Lines | Surfaced in |
|---|---:|---|
| `<faction> entered war against <faction>` | 909 | Wars, Timeline, Insights, War Network |
| `<faction> Invade Win\|Fail On <date>: <faction> - <planet>` | 510 | Wars, Timeline |
| `<faction> is at war` | 2 975 | Wars |

### Faction categories

`<faction> is <category>` — `STH_test_events.txt` lines 1676, 1685, 1689, 1699, 1708.

| Category | Lines |
|---|---:|
| `significant_power` | 2 823 |
| `sth_medium_country` | 2 812 |
| `sth_minor_country` | 2 009 |
| `major_faction` | 1 261 |
| `sth_fallen_empire` | 960 |

The analyzer's `CATEGORY_OVERRIDES` table pins "Major" to the canonical 8 prescripted empires; the mod's runtime `is major_faction` tagging on other empires is demoted to `significant_power` (v2.13.1).

### Other

| Pattern | Lines | Surfaced in |
|---|---:|---|
| `Game difficulty is <level>` | 120 | Dashboard banner |
| `<faction> is human player` | 0 (this log is AI-only) | Dashboard banner, HUMAN badge across all tabs |
| `Game Version: <name>` (engine, not STNH event) | 1 | Dashboard banner |

### Diagnostic logging — added in v2.16.0 (Orion's STH_test_events.txt extensions)

These four pattern families are *not* present in the 62 MB analyzed log (that log predates the extension) but are exercised by the two newer test logs in `input_code/`:
- `improvedgameloggin2.log` (39k lines, Cetus v4.3.5, Game id `273781`, map `STH Galaxy 1400 lore map`)
- `game(3).log` (81k lines, Cetus v4.3.7, Game id `452256`, map `STH huge lore map 3000 stars`)

| Pattern | Mod line | Surfaced in |
|---|---|---|
| `Game id number is <N>` | STH_test_events.txt ~1533, 1892, 1919 | Dashboard banner; Validation card flags multi-game logs |
| `STH …  map …` (23 known variants) | STH_test_events.txt ~1552, 1599 | Dashboard banner + auto-switches the Galaxy tab to the matching one of 16 wiki layouts |
| `<faction> N subjects` | STH_test_events.txt ~1854 | New `subjects` stat — Stats tab, Spreadsheet, Faction Detail, Compare, Galaxy advanced-mode picker |
| `Ethics and Factions OFF` / `Ethics and Factions OFF for AI` | STH_test_events.txt ~1707 | Dashboard Game-Settings banner |
| `Specified Seed: <N>` (engine line) | galaxy_generator.cpp:4075 | Dashboard Game-Settings banner |

#### Known mod-side issue (surfaced in Validation)

`improvedgameloggin2.log` shows a Stellaris color-code leakage bug — events like `^Snavy_size^S ^QUAntican Packs^Q! 0 subjects` (control bytes 0x11/0x13 wrap the empire name, plus the previous `navy_size` payload glues onto the next event). The parser strips these defensively (the cleaned `subjects` data is correct), and the new **"Malformed log lines"** validation card lists every affected raw line for upstream reporting.

---

## ❌ UNPARSED — present in log but currently ignored

6 046 structured event lines that match the `Log effect/trigger` shape but don't fit any pattern the analyzer extracts. Grouped by source mod-file in descending volume.

### `events/STH_klingon_mechanics.txt` — 1 779 lines

Klingon house loyalty/disloyalty bookkeeping. Per house, per game year, the mod logs whether each Great House is currently loyal or disloyal to the High Council, and which event fired.

| Count | Mod-line | Pattern |
|---:|---:|---|
| 825 | 1429 | `House of <X> has no loyalty/disloyalty` (across all houses) |
| ~800 | 1743 | `House of <X> is Loyal` |
| 120 | 1714 | `STH_klingon_mechanics.<id> - fired` |

**Could become:** Klingon politics mini-dashboard (loyalty over time per house). Niche but flavourful for KDF playthroughs. **Not on the roadmap.**

### `events/STH_klingon_houses.txt` — 775 lines

House dominance hierarchy. Pattern: `House of <X> ruled by Rival: House of <Y>` (line 3955) — 690+ occurrences across all combos.

**Could become:** "Who rules the Klingon Empire over time" timeline. **Not on the roadmap.**

### `events/STH_first_contacts.txt` — 647 lines

Random pirate / wandering encounter first-contact events.

Pattern: `First Contact Fired in <System> between <X> Aliens and <Y> Aliens` — line 23 — 554 distinct combos.

**Could become:** Map overlay of where wandering events fire. Data is one-off (fires once, never again). **Not on the roadmap.**

### `events/STH_pulse_events.txt` — 619 lines

Mostly bookkeeping. **Most useful single pattern:**

| Count | Mod-line | Pattern |
|---:|---:|---|
| 121 | 9, 20 | `Year Marker: <YYYY.MM.DD>` |
| ~200 | 2875, 2893 | `<leader-name> assigned to <fleet/structure>` |

**Year Marker** could provide a more authoritative "fresh game year" timestamp than the per-line `[date]` prefix the parser already uses. Marginal win; current parser doesn't need it.

Leader-assignment chatter is per-leader and one-off (no aggregation possible).

### `events/STH_start.txt` — 444 lines

Game-startup bookkeeping. `sth_start.<N>` events covering: game_start.1, sth_start.9 (per-empire starting-buildings placement), sth_start.16/27/51/91/92, etc.

**Could become:** "Campaign setup" summary in Dashboard (which homeworlds got which starting buildings). Nobody's asked for it. **Not on the roadmap.**

### `events/STH_onaction_events.txt` — 427 lines

OnAction-triggered effects, mostly building placement on Borg unicomplexes and special systems:

`STH_onaction.<id> <system> added <building>` — lines 1515, 1531, 1559, 1567 — 180+ distinct combos.

**Better alternative:** Save mode already parses building inventories from the `country.<id>.controlled_buildings` field with full coverage. Log-mode equivalent would always be incomplete.

### `events/STH_operation_events.txt` — 316 lines

Espionage system targets. Internal mod variable assignments showing what spy-fleet targeted what ship.

**Could become:** "Espionage activity" map between empire pairs. Niche. **Not on the roadmap.**

### `events/STH_component_events.txt` — 224 lines

Boarding-action bookkeeping during invasions: `<fleet> - boarded fleet <id>`, `<ship> - invading ship2`.

**Could become:** Tactical detail in the Wars tab's invasion records. Low priority.

### `common/megastructures/STH_science.txt` — 218 lines

`<empire> - science mega` — emitted on every science-megastructure tick.

| Count | Empire |
|---:|---|
| 62 | Tellar Prime Sector |
| 24 | Dominion |
| 23 | United Federation of Planets |
| 17 | Breen Confederacy / Klingon Empire |
| 14 | Sheliak Corporate |
| 13 | Yaderan Union |
| 10 | Tholian Assembly / Rakhari Third Imperium |
| 8 | Son'a Command |
| 6 | Husnock Ascendency |

**Could become:** "Megastructure activity" Stats column. Save mode already shows the megastructures themselves. Log-mode tick-counter is a poor substitute.

### `events/STH_crime_events.txt` — 199 lines

Crime / police-operation events: `<system> Police Operation` (104 distinct system names, mostly one-off).

**Could become:** Crime hotspot map overlay. Niche. **Not on the roadmap.**

### `events/STH_fixes.txt` — 120 lines

Mod-internal fixup events that fire to repair save-state inconsistencies. Pattern: `STH_fix.<id> <description>` (e.g. `STH_fix.192 Fed graphical culture fix`).

Useful for Russ debugging the mod, not for analytics. **Will not be parsed.**

### Smaller files

| File | Lines | Content |
|---|---:|---|
| `events/STH_dominion_flavour_events.txt` | 76 | Genetic templates, Vorta loyalty markers |
| `events/STH_war_events.txt` | 45 | War warning/exhaustion bookkeeping |
| `common/war_goals/STH_war_goals.txt` | 36 | War goal definitions on declaration |
| `events/STH_admonition_events.txt` | 21 | Admonition events |
| `events/STH_expedition_situation_events.txt` | 10 | Expedition-fleet situations |
| `events/situation_deficit_events.txt` | 10 | Resource-deficit situations |
| `events/STH_federation_mechanics.txt` | 7 | Federation internal mechanics |
| `events/STH_borg_mechanics_events.txt` | 6 | Borg assimilation cycles |
| `events/STH_cheronite_story_events.txt` | 3 | Cheronite story milestones |
| `events/STH_technology_events.txt` | 2 | Tech-tree events |
| `common/situations/STH_story_situations.txt` | 2 | Story situations (e.g. Bajoran occupation) |
| `events/STH_undine_mechanics_events.txt` | 1 | Undine fluidic-space isolation |
| `events/STH_species_mechanics.txt` | 1 | Pop-fix events |
| `events/STH_hero_leader_events.txt` | 1 | Hero leader reveals |
| `events/STH_federation_babel_events.txt` | 1 | Babel Conference events |
| `events/STH_cardassian_story_events.txt` | 1 | Cardassian story events |
| `events/STH_borg_situation_events.txt` | 1 | Borg situation events |
| `events/STH_anomaly_emanations.txt` | 1 | Anomaly emanations |
| `common/situations/STH_expedition_situations.txt` | 1 | Expedition situation kickoff |
| Several `solar_system_initializers/*.txt` | 1–2 each | One-off startup markers |

All of these are debug / flavour markers used by Russ during mod development. Not data-grade events. **Not on the roadmap.**

### War-Decl noise

The 909 parsed `war declaration` lines include some malformed entries like:
- `Cardassian Union entered war against navy_size UAntican Packs!`

These are mod-internal placeholder strings that escaped into the log. The parser currently captures them as-is (they end up in `data.wars` with a garbage `target` string). **Not worth filtering** — they're rare and obvious in the UI.

---

## What is **not** in the log (and would be worth requesting from Russ)

| Information | Why missing | Workaround | Status |
|---|---|---|---|
| **Galaxy map ID** (which `galaxy_size` is in use) | The mod resolves the map at game start but doesn't emit which one | Galaxy tab assumes default map; mismatches go silent | Discord request to Russ pending — see `docs/community-messages.md` |
| **Federation founding** event | The Earth → UFP rename happens via mod event but isn't logged as a discrete event | Analyzer auto-detects from "founder ends, UFP begins" timing of revenue/stats lines | Working (v2.13.3) |
| **Empire defeat / vassalization / annexation** | Empires can disappear from the log silently when conquered | No special handling — empire just stops appearing in revenue/stats | Could be inferred from "empire had data, no longer does" |
| **System ownership transfers** | Who controls which system over time | Save mode has it; log mode doesn't | Save mode is the right place |
| **Tech-tree milestones** (each empire researching each tech) | `researched technologies` count is logged but not which techs | Save mode shows tech list per empire | Adding to log would require per-tech log line |
| **Diplomatic events** (federation joins, alliance forms, vassalage) | Wars are logged, peace agreements & alliances aren't | Save mode has full diplomacy state | Acceptable as-is |

---

## Coverage at a glance

```
Structured STNH events:          411 085   (96.9 % of all log lines)
├── Parsed by analyzer:          405 039   (98.5 % of structured)
│   ├── Resource revenue (13)    253 522
│   ├── Stats (8)                155 508
│   ├── Faction categories (6)     9 865
│   ├── Wars (3 kinds)             4 394
│   ├── Difficulty                   120
│   └── Income (26 keys)               0   (this log predates Russ' income logging)
└── Unparsed:                      6 046   (1.5 %)
    └── Mostly Klingon politics, megastructure ticks, mod-internal fixups,
        random first contacts, leader assignments, and one-off startup
        markers. None of it is on the analyzer roadmap because none of it
        answers a user question that's been asked.

Engine messages (not STNH events): 12 943
└── Galaxy generation, mod-load, save-load, system inits — intentionally ignored.
```

---

## Generation

Generated by analyzing `ai_without_new_horizonsgame.log` against the analyzer's known patterns. To regenerate after a parser change or with a different test log, the one-off analysis script is at `C:\Users\marcj\AppData\Local\Temp\analyze_log_v2.mjs`.
