# CLAUDE.md — politicalwindow.com (Frontend)

## Project Overview
PoliticalWindow.com is a political advertising intelligence platform. This repo is the
entire frontend — a single-file static SPA hosted on GitHub Pages.

## Related Repos
- `../politicalwindow-api` — candidate, ballot measure, and state window data API
- `../ratewindow-api` — FCC political ad contract rate intelligence API

## Architecture
- **Single file:** `index.html` contains all HTML, CSS, and JavaScript inline
- **No build step, no bundler, no npm** — do not introduce any
- **Host:** GitHub Pages (auto-deploys on push to main)

## External CDN Dependencies
- **MapLibre GL JS 4.7.1** — geographic map rendering (loaded from unpkg CDN)
- **CARTO Dark Matter basemap** — free dark basemap tiles (no API key needed)
- Falls back to tile-grid cartogram if MapLibre CDN fails to load

## API Endpoints Consumed
| API | Base URL | Purpose |
|---|---|---|
| politicalwindow-api | https://api.politicalwindow.com | States, candidates, ballot measures |
| ratewindow-api | https://rates.politicalwindow.com | FCC ad contract rate intel |

### Response shapes expected
- `GET /api/states` → state metadata (window dates, status, races)
- `GET /api/candidates` → declared candidates grouped by state
- `GET /api/ballot-measures` → ballot measures grouped by state `{ [state_abbr]: [...] }`
- `GET /api/polls` → polls grouped by state → race `{ [state_abbr]: { [race]: [{pollster, date, sample, moe, results}] } }`
- `GET /api/rates?state=TX&limit=20` → rate intel records array

## Key Constants (do not change)
```javascript
const API_BASE  = 'https://api.politicalwindow.com';
const RATES_API = 'https://rates.politicalwindow.com';
const TODAY     = new Date();             // live date — auto-updates state map colors
const GENERAL   = new Date(2026,10,3);   // Nov 3, 2026
```

## Key Functions (do not rename or remove)
| Function | Purpose |
|---|---|
| `loadLiveData()` | Fetches states, candidates, ballot measures, polls on page load |
| `buildDetailHTML(abbr)` | Renders full state detail panel HTML |
| `buildPollContent(abbr)` | Renders polling content for state detail tab |
| `wireDetailTabs(container)` | Wires click handlers for Candidates/Issues/Polling tabs |
| `loadRatesForState(abbr)` | Async fetch + render Rate Intel section |
| `pick(abbr)` | State tile click handler |
| `drawTable()` | Renders sortable candidate/state table |
| `buildMap()` | Renders the 12-column tile grid US map (fallback) |
| `initMap()` | Async — loads MapLibre geographic map, replaces tile grid |
| `addStatesLayer()` | Adds GeoJSON state fill + border layers to MapLibre map |
| `addStateLabels()` | Adds state abbreviation labels as symbol layer |
| `wireMapInteractions()` | Sets up hover tooltips and click handlers on map |
| `mapSelectState(abbr)` | Zooms map to state, dims others, loads districts |
| `mapDeselectState()` | Returns map to national view, removes districts |
| `loadDistricts(abbr)` | Async — fetches congressional district GeoJSON per state |
| `showDistricts(abbr)` | Renders district boundaries on map |
| `hideDistricts()` | Removes district layers from map (incl. cand-tab highlights) |
| `renderCandidatesPane(abbr)` | Renders the Candidates-tab slicer layout for a state |
| `renderStatewideOffice(abbr,s,st)` | Renders Senate/Governor party-grouped candidates |
| `renderHouseOffice(abbr,s,st)` | Renders House district list / drilled-in CD view |
| `renderPartyGroup(...)` | Renders one party group (D/R/I) with top-3 or expanded |
| `renderCuratedFallback(abbr,office)` | Shows curated CANDS before tracker data loads |
| `wireCandidatesPane(c,abbr)` | Delegated event wiring for slicer/See More/CD clicks |
| `rerenderCandidatesPane(c,abbr)` | Replaces pane innerHTML after state change or data load |
| `getRacesForState(s)` | Returns \['Senate','Governor','House'\] present for a state |
| `isPrimaryComplete(s)` | True if status='general-only' or primary date in the past |
| `getCandidatesFor(abbr,office,district?)` | Tracker entities for a given slice |
| `normalizeEntity(e)` | Shapes a tracker record → {name,party,office,district,spend,...} |
| `groupByPartySorted(list)` | Groups candidates by party + sorts each by spend desc |
| `lookupCuratedMeta(abbr,name)` | Gets incumbent/note from CANDS by name (fuzzy last-name) |
| `highlightActiveDistricts(abbr)` | Adds green fill layer to active-race CDs on map |
| `highlightSelectedDistrict(abbr,cd)` | Adds blue fill to selected CD + fits bounds |
| `clearCandDistrictHighlight()` | Removes active/selected fill layers |
| `applyCandMapHighlight(abbr)` | Reapplies highlights after districts load/reload |
| `enrichCandidateFinance(abbr,c)` | Async: caches tracker entities, triggers re-render |

## Design System
- **Aesthetic:** Bloomberg Terminal — dark navy, monospace data, amber accents
- **Fonts:** Bebas Neue (display), IBM Plex Mono (data), DM Sans (body)
- **CSS variables only** — do not introduce hardcoded color values
- **Key colors:** `--navy`, `--blue`, `--amber`, `--red`, `--green`, `--slate`
- **Mobile breakpoint:** 640px — detail panel replaced by bottom drawer

## Data Structures (populated at runtime from API)
- `SM` — state metadata map keyed by state abbr `{ TX: {...}, CA: {...} }`
- `CANDS` — curated candidates by state `{ TX: [{race, list:[{n,p,s,note}]}] }` — used for incumbent flag + notes
- `BALLOT` — ballot measures by state (loaded from API via `loadLiveData()`)
- `POLLS` — polls by state → race (loaded from API via `loadLiveData()`)
- `glMap` — MapLibre GL JS map instance (null if CDN failed)
- `statesGeoJSON` — US states GeoJSON with injected `abbr`/`status` properties
- `DISTRICT_CACHE` — cached congressional district GeoJSON per state abbr
- `_financeCache` — cached tracker-API entities per state abbr (name/party/office/district/total_spend); populated by `enrichCandidateFinance`, consumed by `renderCandidatesPane`
- `candTabState` — per-state UI state for the Candidates tab: `{ office, expanded:{D,R,I}, district }` — preserved across re-entry into a state

## Rules
1. `index.html` stays a single self-contained file — no external JS or CSS files
2. All API calls must use try/catch and degrade gracefully (show empty state, not crash)
3. Do not add localStorage, sessionStorage, cookies, or any client-side persistence
4. Both mobile drawer and desktop panel must render any new UI sections added
5. Do not change API_BASE or RATES_API values

## Current State (Apr 2026)
- Rate Intel section live for TX and CA tiles
- Ballot measures populated from API (empty until seeded)
- FCC General Window: Sep 4–Nov 3, 2026
- Data badge shows LIVE or CACHED SNAPSHOT depending on API availability
- MapLibre geographic map replaces tile-grid cartogram (with tile-grid fallback)
- State zoom + congressional district boundaries on state selection
- Polling data displayed in detail panel tabs
- **Candidates tab (4.14.26)** — slicer-based layout:
  - Color-coded office slicers (Senate/Governor/House) filter the pane
  - Senate/Governor: candidates grouped by party, top-3-with-spend + "See More"
    → "See All Candidates" links to `/candidate-tracker?office=X&party=Y&state=Z`
  - House: district list (tap to drill in) with green highlight on active CDs
    and blue fill on the selected CD via map feature-state layers
  - Post-primary states (`status='general-only'` or `dPrim<=0`) show nominees
    + any indies-with-spend; in-primary states show top 3 per party
  - Tracker-API entities (via `_financeCache`) are merged with curated `CANDS`
    for incumbent/note metadata. Curated-fallback renders while data loads.
- Candidate tracker page sorts by `total_spend` DESC by default (matches above)
