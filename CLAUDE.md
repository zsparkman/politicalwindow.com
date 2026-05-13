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

## Analytics
All HTML pages load two analytics tags inside `<head>`, in this order:

1. **Google Analytics 4** — `gtag.js`, property `G-JGHV19ZYEQ`. Present
   on `index.html`, `candidate-tracker.html`, `explorer.html`,
   `public-file.html`, `rates.html`. Not present on `admin.html` or
   `lur.html` (internal-only pages — kept off GA4 by design).
2. **Microsoft Clarity** — project `wnbzwo190n` (added 2026-05-07).
   Present on **all 7** HTML pages, including `admin.html` and
   `lur.html`. Inserted immediately after the gtag block, or right
   after `<head>` on the two pages without gtag. Provides session
   recordings, heatmaps, and rage-click signal that complements GA4.

When adding a new HTML page, copy both blocks from `index.html` (or
just the Clarity block if it's an internal/admin page that should stay
off GA4). Project ID `wnbzwo190n` is hard-coded in the loader; no env
swap needed.

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
| `renderPartyGroup(abbr,office,p,list,primComplete,expanded,runoffPhase)` | Renders one party group (D/R/I) with top-3 + no-spend fallback |
| `renderCuratedFallback(abbr,office)` | Shows curated CANDS before tracker data loads |
| `renderSeeAllLink(abbr,office,district?)` | Small gray underlined "See All Candidates" bottom link |
| `mergeCuratedIntoEntities(abbr,office,entities)` | Union of tracker entities + curated CANDS (used as no-spend fallback source) |
| `wireCandidatesPane(c,abbr)` | Delegated event wiring for slicer/See More/CD clicks |
| `rerenderCandidatesPane(c,abbr)` | Replaces pane innerHTML after state change or data load |
| `getRacesForState(s)` | Returns \['Senate','Governor','House'\] present for a state |
| `isPrimaryComplete(s)` | True if general-only OR primary past AND NOT in a runoff phase |
| `isRunoffPhase(s)` | True when status is runoff-window or runoff-upcoming |
| `getCandidatesFor(abbr,office,district?)` | Tracker entities for a given slice |
| `normalizeEntity(e)` | Shapes a tracker record → {name,party,office,district,spend,...} |
| `groupByPartySorted(list)` | Groups candidates by party + sorts each by spend desc |
| `lookupCuratedMeta(abbr,name)` | Gets incumbent/note from CANDS by name (fuzzy last-name) |
| `getDistrictGlowColor(abbr)` | Picks glow color from current overlay: heatmap cache if set, else STATUS_COLORS for the state |
| `getHouseDistrictInfo(abbr,num)` | Returns `{incumbent, challengers, all}` for one CD, read from CANDS (FEC-sourced). Drives the district-hover tooltip; handles at-large states (AK/DE/ND/SD/VT/WY) by matching any district |
| `highlightActiveDistricts(abbr)` | Adds/updates bright fill + faint outer halo + crisp hover border (3 layers) on active CDs; colors track the active map overlay, hover delta matches the state-fill hover treatment. **4/15: "active" now means every district in the loaded GeoJSON, not just tracker-backed — every House seat is up every 2 years** |
| `highlightSelectedDistrict(abbr,cd)` | Adds blue fill to selected CD + fits bounds |
| `clearCandDistrictHighlight()` | Removes active glow layers + selected fill |
| `applyCandMapHighlight(abbr)` | Reapplies highlights after districts load/reload |
| `enrichCandidateFinance(abbr,c)` | Async: caches tracker entities, triggers re-render |
| `buildDistrictLabelPoints(geo)` | Collapses districts GeoJSON → one centroid Point per district_number |
| `collectOuterRings(geometry)` | Returns outer rings of Polygon or MultiPolygon |
| `ringBBox(ring)` | Bounding box of a ring (used to pick the largest for label placement) |

## Design System
- **Aesthetic:** Bloomberg Terminal — dark navy, monospace data, amber accents
- **Fonts:** Bebas Neue (display), IBM Plex Mono (data), DM Sans (body)
- **CSS variables only** — do not introduce hardcoded color values
- **Key colors:** `--navy`, `--blue`, `--amber`, `--red`, `--green`, `--slate`
- **Mobile breakpoint:** 640px — detail panel replaced by bottom drawer

## Data Structures (populated at runtime from API)
- `SM` — state metadata map keyed by state abbr `{ TX: {...}, CA: {...} }`.
  Each value now carries `pWindow`, `rWindow`, `dWin`, `dPrim`, `dRun`, `dRunW`, `status`.
  Possible status values: `window-open`, `window-soon`, `upcoming`,
  `runoff-window`, `runoff-upcoming`, `general-only`, `inactive`.
- `CANDS` — candidates by state `{ TX: [{race, list:[{n,p,s,note,d}]}] }` — sourced from `/api/candidates` which is populated by the FEC scraper (daily cron) + curated manual rows. The `d` field holds the House district number (undefined for Senate/Governor); preserved 4/15 so `getHouseDistrictInfo` and `renderHouseOffice` can bucket per-CD without waiting on tracker-API enrichment.
- `BALLOT` — ballot measures by state (loaded from API via `loadLiveData()`)
- `POLLS` — polls by state → race (loaded from API via `loadLiveData()`)
- `glMap` — MapLibre GL JS map instance (null if CDN failed)
- `statesGeoJSON` — US states GeoJSON with injected `abbr`/`status` properties
- `DISTRICT_CACHE` — cached congressional district GeoJSON per state abbr
- `_financeCache` — cached tracker-API entities per state abbr (name/party/office/district/total_spend); populated by `enrichCandidateFinance`, consumed by `renderCandidatesPane`
- `candTabState` — per-state UI state for the Candidates tab: `{ office, expanded:{D,R,I}, district }` — preserved across re-entry into a state
- `_currentStateAbbr` — abbr of the zoomed-to state (null when national view); used by the once-wired district click handler
- `_hoveredDistrictId` — feature id of the district currently under the cursor; drives the `cand-hover` feature-state that brightens the active-district glow
- `DISTRICT_COUNTS` — static map of state abbr → number of congressional districts; the only source of truth for "is this state at-large?" (AK/DE/ND/SD/VT/WY)

## Rules
1. `index.html` stays a single self-contained file — no external JS or CSS files
2. All API calls must use try/catch and degrade gracefully (show empty state, not crash)
3. Do not add localStorage, sessionStorage, cookies, or any client-side persistence
4. Both mobile drawer and desktop panel must render any new UI sections added
5. Do not change API_BASE or RATES_API values

## Station display convention (cable PSID → retail brand)
Cable systems are ingested by ratewindow-api against their FCC PSID
(a numeric string like `003777`), not a broadcast callsign. To keep the
UI human-readable, every page that renders a station column uses the
same three-piece pattern (see `rates.html`, `public-file.html`,
`lur.html`, `explorer.html`; also `candidate-tracker.html` for a
simplified inline version):

1. **`stationLabel(row)`** — preferred on every render site. Returns
   `row.station_display` when the backend provided it (all the relevant
   ratewindow-api endpoints now select
   `station_display_name(...) AS station_display`), otherwise falls back
   to the PSID → label map (`STATION_DISPLAY`) populated from
   `/api/cable-systems`, and finally to the raw `row.callsign` /
   `row.station`. Broadcast callsigns (`KTLA`, `KXAS`, `WPVI`) pass
   through unchanged.
2. **`stationLabelFromCode(code)`** — same lookup without a row,
   used for dropdown option labels (where only the callsign is known).
3. **`STATION_DISPLAY` map + `loadStationDisplay()` IIFE** — runs at
   page load, calls `/api/cable-systems?limit=500`, and populates the
   map. If the deployed API hasn't yet returned a `station_display`
   field, the loader builds it client-side from `display_brand + dma`
   (title-cased) so the rollout works independently of Railway deploy
   timing.

**Filter-dropdown convention:** the option's `value` stays the raw
callsign/PSID (so the existing filter compare keeps matching), while
the option's `textContent` uses `stationLabelFromCode(v)`. Don't change
the field function in `updateOptionStates` — it intentionally returns
the raw code so the dropdown value and the filter field agree.

**Display format:** `"<Brand> (<DMA>)"` — e.g. `Spectrum (Los Angeles)`,
`Comcast (Houston)`. This was Option B from the 2026-04-15 rollout;
multiple PSIDs in the same DMA intentionally collapse to the same
label. Comcast uses its legal name, not Xfinity.

## Current State (Apr 2026)
- **Coastal Owner Dashboard follow-up (5/12/26 PM)** — three fixes
  after first review of `coastal.html`: (1) every strategic-
  recommendation phrase that wasn't deducible from the source data
  was removed (consolidation panel no longer says "stop competing
  against yourself" / "coordinated pitch"; hero gap card no longer
  says "money that flowed past your stations to rivals"; PPTX tile
  no longer says "GSM handoff" — audit grep confirms zero remaining
  instances of those + similar strategic verbs). (2) Two date
  helpers (`fmtWeek` → `3/9`, `fmtDate` → `Mar 9`) replace the
  v1 `.slice(5)` that left ISO-timestamp T-suffix garbage in the
  weekly chart x-axis, daypart heatmap top labels, daypart cell
  tooltips, the worst-week annotation, and the LUR violations
  table week column. (3) Visual legends added to the two charts
  that lacked them — `.chart-legend` HTML bar above the weekly
  trend (blue/red swatches + axis note), `.heat-legend` HTML bar
  below the daypart heatmap (horizontal slate→blue→red gradient
  strip with min/max dollar values bracketing it). Daypart codes
  also get SVG `<title>` tooltips resolving to full daypart names
  (Early Morning / Daytime / etc.) so the two-letter codes decode
  on hover. See `CHANGES.md` 2026-05-12 follow-up entry.
- **Coastal Owner Dashboard (`coastal.html`, 5/12/26 — supersedes
  yesterday's `groups.html`)** — dedicated per-owner page reachable
  at `politicalwindow.com/coastal` (extension-stripped via GH Pages,
  matching `/lur`, `/admin`, `/explorer`). Full authentication
  required — the page renders nothing data-related until
  `/auth/me` returns 200; login overlay matches `lur.html` exactly
  (JWT in `localStorage.pw_token`). Pure read-only viewport over
  existing `ratewindow-api` endpoints — no schema changes, no new
  backend, no edits to any other file. **Visual rework vs the v1**:
  hero is a two-card split with the captured number AND the gap
  number both at 5.5rem Bebas Neue (gap is no longer secondary);
  market scoreboard with share-of-voice stacked bars sized
  proportionally to total market $ replaces the table as primary
  visual; market-detail view gets a 220×220 SVG share donut + peer
  benchmark dot; weekly trend has gradient area fills; compliance
  uses a "station wall" of red/green color-coded tiles; competition
  uses a 25-card advertiser leaderboard. Per-owner onboarding model:
  copy `coastal.html` to `<owner-slug>.html` and edit the `GROUP`
  constant at the top (slug, name, station list, peer benchmark).
  No other code change. The multi-tenant `?group=<slug>` engine from
  yesterday is gone — per-owner files are the production pattern.
  Routes: `/coastal?view=<hero|markets|market|compliance|competition|
  exports>&dma=<DMA>&demo=true`. See `CHANGES.md` 2026-05-12 for the
  full feature index, visual decisions, and v2 deferrals. Page index
  in `politicalwindow.architecture.md` §7.
- **LUR Invoice Review modal (4/24)** — the pop-out on `lur.html` that
  opens from the red "COMPARE INVOICES" button on a variance row now uses
  independent per-panel invoice selection. Each panel has its own
  `.inv-panel-selector` row above its PDF viewport listing every invoice
  in the group (Invoice A / B / C / …); left and right selections are
  independent (any pair viewable, including same-on-both). Initial view
  preserves the legacy default: highest-rate left, LUR right. The
  green "LUR Rate" footer badge now follows the row whose rate equals
  the group minimum, not the panel position. Driven by
  `openInvoiceReview` → `renderInvoicePanels` → `buildPanelHTML(side,
  sorted, idx, lurRate)` and `selectInvoice(side, idx)` →
  `renderInvoicePanelSide`. The old top-of-modal `#inv-tabs` strip,
  `renderInvoicePair`, and `switchInvoiceTab` are gone.
- **LUR matched-line indicator removed (4/24)** — `renderPanelPages` no
  longer scans PDF text content, no longer draws the amber highlight
  bar, no longer renders the "▶ MATCHED LINE" tag, and no longer
  prepends the "Locate Order #… in the document below" banner on miss.
  The `.inv-pdf-highlight-row` / `.inv-pdf-highlight-tag` /
  `.inv-pdf-banner` / `.inv-pdf-highlight` CSS rules were deleted. The
  selector UI above each panel already makes clear which invoice is
  which; the highlight was judged intrusive.
- Rate Intel section live for TX and CA tiles
- Ballot measures populated from API (empty until seeded)
- FCC General Window: Sep 4–Nov 3, 2026
- Data badge shows LIVE or CACHED SNAPSHOT depending on API availability
- MapLibre geographic map replaces tile-grid cartogram (with tile-grid fallback)
- State zoom + congressional district boundaries on state selection
- Polling data displayed in detail panel tabs
- **Candidates tab (4.14.26, incl. Follow-up #1 + Follow-up #2)** — slicer-based layout:
  - Detail panel: `.detail-header` (status line + state name + abbr) is
    followed by a sticky `.detail-tabs` row. **Overview** is tab 1 (default)
    and contains the race tags, primary/runoff dates, and FCC window bars
    (primary + runoff + general). Candidates/Issues/Polling are tabs 2/3/4.
  - Color-coded office slicers (Senate/Governor/House) filter the pane
  - Senate/Governor: candidates grouped by party, top-3-with-spend + "See More"
    → single small gray underlined "See All Candidates" link at the bottom
    of the candidates list (right aligned) linking to
    `/candidate-tracker?office=X&party=Y&state=Z`
  - When tracker spend is missing for a race, the renderer falls back to
    `mergeCuratedIntoEntities` so each party still shows ≥2 curated
    candidates in primary/runoff phases (or one nominee post-primary).
  - **Min-1-per-party-per-race floor (Follow-up #2).** `renderPartyGroup`
    in-primary/runoff branch now pads `withSpend.slice(0,3)` from the
    no-spend pool up to `MIN_FLOOR = 2` per party, so each declared party
    shows at least 2 candidates regardless of spend (unless truly none are
    declared). See More (+N) now counts unshown entries across the full
    spend+no-spend list. Post-primary branch unchanged.
  - **District-row label fix (Follow-up #2).** House district-list rows
    now show `N candidate(s)` instead of the misleading `N candidates
    with spend`, and render `no spend reported` (faint) instead of `—`
    when `totalSpend === 0`. Empty-state copy corrected from "No House
    districts with reported spend for X" → "No active House races on
    file for X".
  - House: district list (tap to drill in) with a **bright overlay-matched
    surface fill + faint outer halo** on active CDs (Follow-up #2 — two
    layers, simplified from the previous 3-layer outline-glow). The glow
    color now tracks the **active map overlay** via `getDistrictGlowColor`:
    under Window Timing it picks `STATUS_COLORS[s.status]`, under Density /
    Spend / Cash it picks `_lastHeatmapColors[abbr]`. Layers:
    `districts-active-fill` (0.55 → 0.72 opacity on hover) and
    `districts-active-glow-outer` (blurred line, 0.28 → 0.45 on hover).
    Hovering brightens both via `cand-hover` feature-state.
    `setMapLayer` calls `applyCandMapHighlight(_currentStateAbbr)` after
    any overlay switch so the glow recolors in place (paint-property
    update, no layer rebuild). Clicking an active district anywhere on
    the map drills into that CD (same effect as clicking its row), auto-
    switching the Candidates tab to House. The `states-fill` click handler
    early-exits when the same point hits `districts-active-fill` so MapLibre
    doesn't double-fire and toggle the state off. Selected CD gets a blue
    fill overlay.
  - **Districts illuminate on state selection** (Follow-up #2). Active
    House districts light up from the moment a state with House races is
    selected — no longer gated on the user picking the House slicer.
    `applyCandMapHighlight` checks `getRacesForState(s).includes('House')`
    instead of `st.office === 'House'`; the selected-district blue overlay
    is the only behavior still gated on drill-in.
  - **Runoff-phase states** (`status='runoff-window'` or `'runoff-upcoming'`)
    are treated as still in-primary for candidate-list purposes and show
    the runoff finalists per party. Status chip + map color + FCC Runoff
    Window bar reflect the LUR status for the 45 days before the runoff.
  - Post-primary states (`status='general-only'`) show nominees + any
    indies-with-spend; in-primary/runoff states show top 3 per party.
  - **At-large fix:** only the six true single-district states
    (AK/DE/ND/SD/VT/WY) are labeled `AL`. Multi-district states no longer
    fall back to `district_number = 0`.
  - **Labels:** Alaska suppresses CD labels entirely. All other states use
    a dedicated point-feature source (`districts-labels-src`) with one
    centroid per unique `district_number`, so Hawaii shows a single "1"/"2"
    label on the largest island instead of duplicating across every atoll.
- **Window Timing palette refined (Follow-up #2, 4/15) — two rounds of iteration.**
  Started the day deep (red/amber/violet/navy/slate) to replace the earlier
  "box of crayons" set, then bumped brightness by one step after user
  feedback that the new set read as too dull. Current production values:
  - `--status-open:     #B8221E` (window-open / runoff-window — bumped from #991B1B)
  - `--status-soon:     #D1680F` (window-soon / runoff-upcoming — bumped from #B45309)
  - `--status-upcoming: #2563EB` (unified blue — bumped from #1D4ED8; earlier same day merged "general-only" into this, see "Upcoming + General Only merge" below)
  - `--status-general:  #2563EB` (same unified blue; var name retained so status-line branches compile)
  - `--status-none:     #8B95A5` (inactive — slightly warmer slate, bumped from #94A3B8)
  `STATUS_COLORS`, `WC`, `SL`, `SPILL`, map-tooltip `statusLabel`, insights
  `statusDot`, `.tile-*`, `.dot-*` legend dots, and the new
  `.spill-status-open/soon/upcoming/general/none` classes all reference
  these vars. The governor pill (`.tag-governor` / `.slicer-governor` /
  `.spill-purple`) stays on its own `#7C3AED` — "Governor" is a race type,
  not a window-timing status.
- **Upcoming + General Only merged into single "Upcoming" category (4/15).**
  After iterating on the `--status-upcoming` hue (violet → teal → blue), the
  user requested the two primary-complete-but-waiting phases be visually
  collapsed into one category called "Upcoming" in unified blue `#1D4ED8`.
  Both `--status-upcoming` and `--status-general` CSS vars now point at the
  same blue (the vars are retained so internal status-value branches still
  compile). Both `STATUS_COLORS['upcoming']` and `STATUS_COLORS['general-only']`
  also resolve to `#1D4ED8`, so the map fill/border match expressions,
  `WC`, and `statusDot` insights all render the same color for either
  status. User-facing labels were also unified: `SL` shows `○ UPCOMING` for
  both statuses, the hover tooltip `statusLabel` reads `UPCOMING`, the
  `SPILL` map reads `○ UPCOMING` for both, and the state-detail status
  line for post-primary states reads `✓ PRIMARY COMPLETE · UPCOMING`. The
  legend row for "General Only" was removed entirely — only the single
  "Upcoming" (`dot-purple`) row remains. `SO` sort now gives both statuses
  rank 2 (inactive shifted from 4→3). The internal `general-only` status
  value is still produced by `classifyStatus` for states past their primary
  date and is still consumed by `isPrimaryComplete()` + the `✓ COMPLETE`
  spill-green chip rendering — that chip is a functional completion
  indicator and deliberately stays green (not blue).
- Candidate tracker page sorts by `total_spend` DESC by default (matches above)
- **Back to Map button (Follow-up #2)** is now absolutely positioned inside
  `#mapContainer` in the top-left corner (previously inline in
  `.map-layer-toggles` alongside Window Timing / Candidate Density / Total
  Spend / Cash on Hand). Renders as a translucent white pill with a drop
  shadow + backdrop blur so it stays legible over the dark basemap and any
  glowing district fills. `id="backToUSA"` unchanged; `mapSelectState` /
  `mapDeselectState` still toggle its `style.display`.
- **House coverage = all 435 districts (4/15).** Every House seat is up
  every 2 years, so every district should have a hover tooltip and show at
  least an incumbent. Previously the frontend treated "active districts"
  as the subset backed by tracker-API spend records — which hid ~60% of
  districts because tracker data is sparse. Now:
  - `highlightActiveDistricts(abbr)` builds its "active" set from the
    full loaded GeoJSON (`DISTRICT_CACHE[abbr].features`) instead of
    `getCandidatesFor(abbr, 'House')`. Every CD in the selected state
    lights up on selection.
  - `renderHouseOffice` now runs tracker entities through
    `mergeCuratedIntoEntities('House', ...)` so the district list reflects
    the full FEC-sourced candidate set, not just tracker-backed spend
    rows. Districts with no tracker spend still appear with `no spend reported`.
  - `mergeCuratedIntoEntities` now preserves the House district number
    (was hardcoded `district: null`); reads `c.d` from CANDS and resolves
    at-large states (`DISTRICT_COUNTS[abbr] === 1`) to `'AL'`.
  - `loadLiveData` CANDS transformation now carries `d: c.district`
    through from the `/api/candidates` response (was dropped before).
  - New helper `getHouseDistrictInfo(abbr, num)` → `{incumbent, challengers, all}`
    for one CD. Handles at-large states by matching anything when
    `DISTRICT_COUNTS[abbr] === 1`.
  - New district-level `mousemove` tooltip on `districts-active-fill`.
    Shows `STATE-CD` label, incumbent (name + party + INCUMBENT tag),
    and up to 3 challengers with a `+N more` tail. Reuses `#mapTooltip`
    and picks a border color from `STATUS_COLORS[state.status]`.
  Data source: FEC scraper in `../politicalwindow-api/scraper.js` runs
  daily at 6 AM ET and populates the `candidates` table with ~1,688 House
  rows across all 51 states (50 + DC), 426 marked as incumbents. See
  `CHANGES.md` for the audit (total House, by party, at-large coverage).
