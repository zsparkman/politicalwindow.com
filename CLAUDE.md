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
| `hideDistricts()` | Removes district layers from map |

## Design System
- **Aesthetic:** Bloomberg Terminal — dark navy, monospace data, amber accents
- **Fonts:** Bebas Neue (display), IBM Plex Mono (data), DM Sans (body)
- **CSS variables only** — do not introduce hardcoded color values
- **Key colors:** `--navy`, `--blue`, `--amber`, `--red`, `--green`, `--slate`
- **Mobile breakpoint:** 640px — detail panel replaced by bottom drawer

## Data Structures (populated at runtime from API)
- `SM` — state metadata map keyed by state abbr `{ TX: {...}, CA: {...} }`
- `CANDS` — candidates by state `{ TX: [{race, list:[{n,p,s,note}]}] }`
- `BALLOT` — ballot measures by state (loaded from API via `loadLiveData()`)
- `POLLS` — polls by state → race (loaded from API via `loadLiveData()`)
- `glMap` — MapLibre GL JS map instance (null if CDN failed)
- `statesGeoJSON` — US states GeoJSON with injected `abbr`/`status` properties
- `DISTRICT_CACHE` — cached congressional district GeoJSON per state abbr

## Rules
1. `index.html` stays a single self-contained file — no external JS or CSS files
2. All API calls must use try/catch and degrade gracefully (show empty state, not crash)
3. Do not add localStorage, sessionStorage, cookies, or any client-side persistence
4. Both mobile drawer and desktop panel must render any new UI sections added
5. Do not change API_BASE or RATES_API values

## Current State (Mar 2026)
- Rate Intel section live for TX and CA tiles
- Ballot measures populated from API (empty until seeded)
- FCC General Window: Sep 4–Nov 3, 2026
- Data badge shows LIVE or CACHED SNAPSHOT depending on API availability
- MapLibre geographic map replaces tile-grid cartogram (with tile-grid fallback)
- State zoom + congressional district boundaries on state selection
- Polling data displayed in detail panel tabs
