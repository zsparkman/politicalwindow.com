# CHANGES — politicalwindow.com

Frontend changelog. Newest entries first. Document all non-trivial edits here.

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
