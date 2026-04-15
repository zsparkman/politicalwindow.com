# CHANGES — politicalwindow.com

Frontend changelog. Newest entries first. Document all non-trivial edits here.

---

## 2026-04-14 — Candidates tab · Follow-up #2 (spec 4/11/26 11:20pm)

**Spec:** `../Candidates Tab Improved Layout 4.14.26.rtf` (Follow-up #2 section)

Five-item punch-list from the user's 11:20pm review. Scopes the district-level
illumination redesign, surfaces House districts on state selection without
requiring the House slicer, guarantees a min-1-per-party floor on active-race
district views, and moves the Back to Map affordance onto the map panel itself.

### Min-1-candidate-per-party-per-race floor

1. **`renderPartyGroup` rewrite (in-primary / runoff branch).** Previously the
   renderer filtered to `withSpend = list.filter(c => c.spend > 0)` and, if
   any with-spend entry existed, *only* showed with-spend candidates (top 3).
   No-spend entries were only surfaced as a fallback when the party had
   **zero** with-spend candidates. Under Follow-up #2 the active-race rule is
   explicit: minimum one candidate per party per race, regardless of spend,
   unless truly none are declared.

   New behavior:
   - `sorted  = list sorted by spend desc` — defensive re-sort so the caller
     can't invert the no-spend tail
   - `withSpend` + `noSpend` split for clarity
   - In-primary / runoff: `base = withSpend.slice(0, 3)`; if `base.length < 2`
     we pad from `noSpend.slice(0, 2 - base.length)` so each party hits a
     `MIN_FLOOR = 2`
   - "See More (+N)" now counts unshown entries across the combined `sorted`
     list (spend + no-spend), so no-spend candidates aren't orphaned under a
     party that has 1–2 with-spend entries
   - Post-primary branch unchanged (still prefers with-spend; falls back to
     `sorted.slice(0,1)` for the nominee)
   - Expanded branch simplified to `sorted.slice()` — See More now reveals
     the full list including no-spend entries
2. **District-list row label corrected.** `renderHouseOffice` previously
   labeled each row `N candidates with spend`, but the `districtMap` bucket
   contains **all** tracker entities (not filtered by spend), so the count
   included no-spend candidates too. Label is now just `N candidate(s)`, and
   the spend column renders `no spend reported` (faint) instead of `—` when
   `totalSpend === 0` — so districts that are *active but unspent* still
   read as present in the list instead of looking stale.
3. **Empty-state copy corrected.** `No House districts with reported spend
   for ${abbr}` → `No active House races on file for ${abbr}`. Same reason:
   the district-list is not gated on spend.

### District illumination — bright fill + faint glow

4. **`highlightActiveDistricts` simplified from 3 layers to 2.** The
   Follow-up #1 implementation stacked three layers (soft 0.18-opacity fill
   + wide blurred halo width 10→16 + crisp 1.75px `#6EE7B7` inner rim). The
   visual result was a district *outline* glowing — the surface stayed mostly
   grey. Follow-up #2 asks for the district surface itself to read as "lit
   up", much brighter, with a faint halo.

   - `districts-active-fill` — now bright uniform fill with `fill-opacity`
     driven by `cand-hover` feature-state: **0.55 idle → 0.72 on hover**
     (was 0.18 flat). Color still `#10B981`.
   - `districts-active-glow-outer` — faint halo. `line-blur: 8`, width
     `5 → 8 on hover`, opacity `0.28 → 0.45 on hover`. Was `line-blur: 10`,
     width `10 → 16`, opacity `0.65 → 0.95`.
   - `districts-active-glow-inner` — **removed**. The bright fill already
     carries the district boundary; the `#6EE7B7` rim was redundant.
   - `clearCandDistrictHighlight` + `hideDistricts` cleanup arrays pruned to
     remove references to `districts-active-glow-inner`.

### Districts illuminate on state selection

5. **`applyCandMapHighlight` no longer gated on `st.office === 'House'`.**
   The spec is explicit: "House districts should be illuminated from the
   moment the state is selected, which upon clicking zooms in to the house
   section of the candidate tab." Previously the highlight layers were only
   added when the user had picked the House slicer in the Candidates tab —
   first entry into a state showed a dim map until the user clicked House.
   New logic:
   - Look up the state metadata and `getRacesForState(s)`; bail early if
     the state has no House races at all
   - Always call `highlightActiveDistricts(abbr)` for House states
   - Only apply the `highlightSelectedDistrict` blue overlay when the user
     *has* drilled into a specific CD (`st.office === 'House' && st.district`)
   - Call sites unchanged — `mapSelectState.once('moveend')` still fires
     it after districts are loaded, and `enrichCandidateFinance` still fires
     it once tracker data is cached (so districts light up as soon as the
     API responds on first entry).
   - The existing map-click handler on `districts-active-fill` already drills
     into House + switches to the Candidates tab, so "click an illuminated
     district → zoom in to house section" is already wired (Follow-up #1
     item 8) and now unconditionally reachable.

### Back to Map · anchored to map panel top-left

6. **Button lifted out of `.map-layer-toggles`, absolutely positioned inside
   `#mapContainer`.** Follow-up #1 moved it *into* the layer-toggles row
   alongside Window Timing / Candidate Density / Total Spend / Cash on Hand.
   Follow-up #2 asks for it to live inside the map panel itself, top-left.
   - Removed the `<button class="back-to-usa">` from inside
     `.map-layer-toggles` in the `.map-card` header
   - Added it as a direct child of `#mapContainer`, which is already
     `position:relative`, so `position:absolute; top:10px; left:10px` anchors
     it to the top-left of the rendered MapLibre canvas
   - Restyled: translucent white (`rgba(255,255,255,0.96)` + `backdrop-filter:
     blur(6px)`) + drop shadow so it stays legible over the dark basemap
     and over glowing district fills
   - `z-index:20` keeps it above the map canvas and tooltip
   - `onclick="mapBackToUSA()"` unchanged; `id="backToUSA"` unchanged so the
     `style.display = 'inline-flex'` toggle in `mapSelectState` and the
     `style.display = 'none'` toggle in `mapDeselectState` still work

### Files touched

- `politicalwindow.com/index.html` — all of the above
- `politicalwindow.com/CHANGES.md` — this entry
- `politicalwindow.com/CLAUDE.md` — Current State + Key Functions update

---

## 2026-04-14 — Candidates tab · Follow-up #1 (spec 4/11/26 9:50pm)

**Spec:** `../Candidates Tab Improved Layout 4.14.26.rtf` (Follow-up #1 section)

Eleven-item punch-list from the user's 4/11 review, spanning the detail panel,
the state/district map layers, candidate-list fallbacks, a bug fix for
at-large district mislabeling, and runoff-window semantics.

### Detail panel

1. **Tabs moved above the state header.** `buildDetailHTML` now renders a new
   `.detail-header` (status chip + state name + abbr) followed immediately by
   `.detail-tabs`, which is `position:sticky` to the top of the scroll area.
   Previously the tabs were buried below the FCC window bars.
2. **New "Overview" tab — first and default.** Contains the race-tag row,
   Primary/Runoff dates, FCC Primary Window bar, FCC Runoff Window bar
   (conditionally rendered for states with a runoff), and FCC General Window
   bar. Candidates/Issues/Polling follow as tabs 2/3/4.
3. **Runoff window rendering.** A new `runoffWindowHTML` block mirrors the
   primary-window styling (date range + progress bar + note) so states in
   runoff show "LUR APPLIES NOW" when the runoff window is open.
4. **"See All Candidates" restyle.** Previously a bold pill-shaped
   `.cand-action-btn.cand-see-all` per party group; now a single small gray
   underlined `.cand-see-all-link` wrapped in `.cand-see-all-wrap` (right
   aligned via flexbox) at the bottom of the whole candidate list. Still
   carries `office`, `party` (when applicable) and `state` URL params.
5. **District back-button + CD-name overlap fix.** The "‹ All Districts"
   button is now wrapped in `.cand-district-back-row` so it occupies its own
   line; `.cand-district-current` is `display:block` + `width:fit-content` so
   the CD pill sits entirely below the back button with no overlap.
6. **No-spend fallback.** New `mergeCuratedIntoEntities` merges tracker-API
   entities with curated `CANDS` so party groups always have at least two
   candidates when tracker data is missing (or one post-primary nominee).
   `renderPartyGroup` takes a new `runoffPhase` arg and applies the fallback
   in both pre-primary and pre-runoff states.

### Runoff-window semantics

7. **New statuses: `runoff-window` + `runoff-upcoming`.** `buildStates` now
   computes `rWindow` (45-day LUR window before runoff) and `dRun`/`dRunW`
   (days to runoff / to runoff window). After a primary completes, if a
   runoff is still ahead, the state moves into `runoff-upcoming` (>30d out)
   or `runoff-window` (actively in the 45-day LUR window) rather than
   dropping straight to `general-only`.
8. **Color + label propagation.** `STATUS_COLORS`, `WC`, `SL`, `SPILL`, the
   MapLibre `fill-color`/`line-color` match expressions in `addStatesLayer`
   and `setMapLayer`, the `statusLabel` tooltip table, and the `SO` sort
   rank all learned the two new statuses (runoff-window uses red like
   window-open; runoff-upcoming uses amber like window-soon).
9. **`isPrimaryComplete` refinement.** Now returns `false` for runoff-phase
   states so party-group logic keeps showing finalists (top 2+) instead of
   collapsing to a post-primary nominee view. New `isRunoffPhase` helper.

### Map

10. **Upcoming state color → purple.** `STATUS_COLORS.upcoming` flipped from
    `#2563EB` to `#7C3AED` (same shade as the Governor pill border/text).
    Propagated through `WC`, `SL`, the tooltip color table, insights
    `statusDot` map, `.tile-upcoming` CSS, `.spill-purple` CSS (previously a
    blue-tinted misnomer), and the status-bar legend (new `.dot-purple`
    class + `dot-purple` legend swatch).
11. **Real glow on active-race districts.** `highlightActiveDistricts` now
    adds three layers instead of one:
    - `districts-active-fill` — soft translucent green fill (`#10B981`, 0.18)
    - `districts-active-glow-outer` — wide blurred line halo (width 10→16
      when `cand-hover` feature-state is on, blur 10, opacity 0.65→0.95)
    - `districts-active-glow-inner` — crisp 1.75px rim in `#6EE7B7` to keep
      the boundary readable through the halo
12. **Click-to-drill on map districts.** `wireMapInteractions` now binds
    click + mousemove/mouseleave to the `districts-active-fill` layer. Click
    sets `candTabState[abbr].district` + forces the Candidates tab active +
    re-renders the pane + calls `highlightSelectedDistrict`. Hover toggles
    the `cand-hover` feature-state per district id, which drives the
    `districts-active-glow-outer` line-width/opacity `case` expressions.
13. **`_currentStateAbbr` + `_hoveredDistrictId` globals** — tracked by
    `mapSelectState`/`mapDeselectState` so the once-wired district click
    handler knows the current state context and the hover handler can
    toggle feature-state.
14. **Alaska — district labels suppressed.** `showDistricts` skips the
    label layer entirely when `abbr === 'AK'` (the data occasionally returns
    multiple mislabeled polys; there's only one at-large district anyway).
15. **Hawaii (and any multi-polygon state) — one label per district.** New
    helpers `buildDistrictLabelPoints`, `collectOuterRings`, `ringBBox`
    collapse the raw districts GeoJSON into a point-feature source
    (`districts-labels-src`) with one centroid per unique `district_number`
    (centroid of the largest bbox ring, so Hawaii's "1"/"2" labels land on
    the biggest island instead of duplicating across every atoll). Label
    symbol layer is renamed to `districts-label-pt` and sourced from the
    new point source.
16. **At-Large bug fix.** `loadDistricts` previously fell back to
    `district_number = 0` and label `AL` whenever a CD numeric couldn't be
    parsed, which meant real districts in large states could be relabeled
    "AL". Now gated on `DISTRICT_COUNTS[abbr] === 1` — only the six true
    single-district states (AK/DE/ND/SD/VT/WY) get the at-large treatment;
    everything else falls back to its index.
17. **House district bucketing.** `renderHouseOffice` now buckets candidates
    without a district into `'unknown'` (labeled `${abbr}-??` in the list)
    for multi-district states instead of forcing them into the AL bucket.
18. **"Back to USA" → "Back to Map".** The button is moved from a standalone
    row above the map into the `.map-layer-toggles` flex row alongside the
    Window Timing / Candidate Density / Total Spend / Cash on Hand toggles,
    and restyled (smaller padding + border radius) to visually match them.

### New / modified CSS

- `.detail-header` (new) — state-header wrapper with bottom border
- `.detail-tabs` — now `position:sticky; top:0; background:#fff; z-index:2`
- `.cand-see-all-wrap` / `.cand-see-all-link` (new) — small gray underlined
  right-aligned bottom link
- `.cand-district-back-row` (new) — forces back button onto its own line
- `.cand-district-back` — now has a hover background + radius for a proper
  button feel
- `.cand-district-current` — `display:block; width:fit-content` so the CD
  pill sits fully below the back button
- `.back-to-usa` — smaller/tighter to fit inside `.map-layer-toggles`
- `.tile-upcoming` — purple tint instead of blue
- `.spill-purple` — purple tint (was a blue tinted misnomer)
- `.dot-purple` (new) — legend swatch for the new upcoming color

### New / modified JS

- `buildStates` — computes `rWindow`, `dRun`, `dRunW`; emits `runoff-window`
  + `runoff-upcoming` statuses
- `isPrimaryComplete` — returns false during runoff phase
- `isRunoffPhase(s)` (new)
- `renderPartyGroup(..., runoffPhase)` — new arg, new no-spend fallback path
- `mergeCuratedIntoEntities(abbr, office, entities)` (new) — union of
  tracker entities + curated CANDS
- `renderStatewideOffice` — passes runoffPhase, uses merged list, appends
  single bottom See All link
- `renderHouseOffice` — runoff-aware, `'unknown'` district bucket for
  multi-district states, new See All link at bottom
- `renderSeeAllLink(abbr, office, district?)` (new)
- `showDistricts` — skips labels for AK, uses new point-feature source for
  the rest (`districts-label-pt`)
- `buildDistrictLabelPoints(geo)` / `collectOuterRings(geometry)` /
  `ringBBox(ring)` (new)
- `highlightActiveDistricts` — now adds three layers (fill + outer glow +
  inner rim), with `cand-hover` feature-state driving the outer glow
- `wireMapInteractions` — new click / mousemove / mouseleave handlers on
  `districts-active-fill`
- `mapSelectState` / `mapDeselectState` — maintain `_currentStateAbbr` and
  `_hoveredDistrictId`
- `clearCandDistrictHighlight` / `hideDistricts` — clean up the new glow +
  label layers and the point source

### Files touched

- `politicalwindow.com/index.html` — all of the above
- `politicalwindow.com/CHANGES.md` — this entry
- `politicalwindow.com/CLAUDE.md` — Current State + Key Functions update

---

## 2026-04-14 — Candidates tab · slicer-based layout

**Spec:** `../Candidates Tab Improved Layout 4.14.26.rtf`

Reworked the state-detail Candidates tab from a flat race-grouped list into
a slicer-driven layout that puts ad-spend context front and center.

### New behavior

1. **Office slicers.** Color-coded buttons (Senate = blue, Governor = purple,
   House = green) act as the primary filter at the top of the Candidates tab.
   Only offices applicable to the selected state are shown.
2. **Party groups with top-3 cap.** Senate and Governor slicers render
   candidates grouped by party (Democratic / Republican / Independent), sorted
   by ad spend descending. For states still in primary, each party group shows
   the top 3 candidates with reported spend + a "See More" button to expand.
3. **Post-primary states.** When the primary is complete
   (`status='general-only'` or `dPrim<=0`), each party group shows all
   candidates with reported spend (nominee + active indies) plus a
   "See All Candidates" link straight into the tracker.
4. **See All Candidates links.** Per-party buttons link to
   `/candidate-tracker?office=X&party=Y&state=Z`, preserving the selected
   slice so the full tracker opens pre-filtered.
5. **House district list.** Clicking the House slicer shows a sortable list
   of congressional districts with active races (candidate count + total
   reported spend). Districts with active races are highlighted on the map
   in green.
6. **House district drill-in.** Clicking a district row pans the map to
   that CD, paints a blue fill over it, and moves the CD to the top of the
   panel while showing its candidates via the same party-group UX.
7. **Tracker page sort.** `candidate-tracker.html` now sorts results by
   `total_spend` DESC by default so it matches the home-page Candidates tab
   "top spenders" emphasis.

### New CSS classes

`.cand-slicer-bar`, `.cand-slicer`, `.slicer-senate|governor|house`,
`.party-group`, `.party-group-D|R|I`, `.party-group-label`, `.cand-action-btn`,
`.cand-see-all`, `.cand-see-more`, `.cand-district-header`,
`.cand-district-list`, `.cand-district-row`, `.cand-district-code`,
`.cand-district-meta`, `.cand-district-spend`, `.cand-district-empty`,
`.cand-district-back`, `.cand-district-current`, `.cand-loading-note`

### New JS functions

- `renderCandidatesPane(abbr)` — top-level renderer for the tab content
- `renderStatewideOffice(abbr,s,st)` / `renderHouseOffice(abbr,s,st)` —
  per-office sub-renderers
- `renderPartyGroup(...)` / `renderCandRowHTML(...)` — primitives
- `renderCuratedFallback(abbr,office)` — pre-tracker-load placeholder
- `wireCandidatesPane(container,abbr)` — delegated click handling on the pane
- `rerenderCandidatesPane(container,abbr)` — in-place refresh on state change
  and after finance data load
- `getRacesForState(s)` / `isPrimaryComplete(s)` — classification helpers
- `getCandidatesFor(abbr,office,district?)` / `normalizeEntity(e)` — tracker
  entity lookups
- `groupByPartySorted(list)` — party-bucketed, spend-sorted grouping
- `lookupCuratedMeta(abbr,name)` — pulls incumbent/note from curated CANDS
- `highlightActiveDistricts(abbr)` / `highlightSelectedDistrict(abbr,cd)` /
  `clearCandDistrictHighlight()` / `applyCandMapHighlight(abbr)` — map
  district highlight layers tied to the Candidates tab

### New JS state

- `const _financeCache = {}` — relocated above the renderers so it's defined
  before any helper reads it (same storage, earlier initialization)
- `const candTabState = {}` — per-state UI state
  `{ office, expanded:{D,R,I}, district }`, preserved across re-entry

### Changed functions

- `buildDetailHTML(abbr)` — candidates-tab content now delegates entirely to
  `renderCandidatesPane(abbr)`. Previous inline race/party rendering removed.
- `enrichCandidateFinance(abbr,container)` — no longer patches individual
  rows with FEC data. It now (a) fetches tracker entities into
  `_financeCache`, (b) calls `rerenderCandidatesPane`, (c) calls
  `applyCandMapHighlight`. FEC cash-on-hand enrichment was auth-gated and
  rarely visible; it is dropped for simplicity.
- `pick(abbr)` / `openDrawer(abbr)` — each now calls
  `wireCandidatesPane(container, abbr)` immediately after inserting the
  detail HTML so slicer/See More/CD clicks work from the first render.
- `mapSelectState(abbr)` — after districts render, calls
  `applyCandMapHighlight(abbr)` so highlights reappear when re-entering a
  state that previously had House + a CD selected.
- `mapDeselectState()` — clears Candidates-tab highlight layers before the
  districts source is torn down.
- `hideDistricts()` — also removes `districts-active-fill` and
  `districts-selected-fill` layers before removing the source.

### Tracker page

- `filterAndRender()` in `candidate-tracker.html` now sorts `FILTERED` by
  `total_spend` descending (tiebreak by `candidate_name`). No schema changes;
  no new URL params.

### Files touched

- `politicalwindow.com/index.html` — CSS additions, new helpers, renderer,
  wiring, map highlight layers
- `politicalwindow.com/candidate-tracker.html` — default sort by spend desc
- `politicalwindow.com/CLAUDE.md` — documented new functions, data
  structures, and Current State section
- `politicalwindow.com/CHANGES.md` — this entry (new file)
