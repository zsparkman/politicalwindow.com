# CHANGES — politicalwindow.com

Frontend changelog. Newest entries first. Document all non-trivial edits here.

---

## 2026-05-12 — `coastal.html` follow-up #3: weekly chart annotation no longer clips

The worst-comp-week annotation on the market-detail weekly trend chart
was clipping in two ways: the label sat above the dot at `wy − 10`
which fell outside the SVG when the worst-week point landed at the
y-axis max (e.g. Anchorage where the comp line peaked at $487K =
top of chart); and right-extending text fell off the right edge when
the worst week was near the end of the cycle. Fix:

- Top padding bumped from 14 → 32 viewBox units, chart height from
  240 → 248, so there's headroom above any peak for an annotation.
- Annotation now smart-positions itself:
  - Right-anchored (`text-anchor="end"`, `x = wx − 8`) when the dot
    sits in the right 55% of the chart, so text never extends past
    the right edge.
  - Plotted below the dot (`y = wy + 18`) when the dot is within
    18 px of the top padding, so the label stays inside the SVG even
    when the data point is at the y-axis max.
  - Left-of-dot above-the-dot is the default (unchanged) for any
    other position.

No data-layer changes; pure rendering fix in `weeklyChart()`.

---

## 2026-05-12 — `coastal.html` follow-up #2: Exports tab removed + weekly chart switched to invoice-level data (KTBY audit fix)

### Exports tab removed

The Exports tab was removed from the top-nav `VIEWS` array. The Owner
Briefing PDF still works from any view via `window.print()` — the
`@media print` stylesheet + `beforeprint` hook are unchanged. The
dedicated tab UI was 90% empty (the only live action was Print; PPTX
is still v2; the share-link copy was a one-line action better placed
inline) and surfacing it as a primary tab implied more was there.

- `VIEWS` array trimmed from 5 entries to 4 (Overview / Markets /
  Compliance / Competition).
- Top-level router still routes `?view=exports` for legacy deep links —
  it falls through to `renderHero()` rather than 404'ing.
- `renderExports()` function deleted entirely (~60 lines of dead code).
- Browser print is the canonical Owner Briefing path now: hit the
  browser's File → Print from any view; the print stylesheet + the
  `beforeprint` listener that injects `#print-briefing` produce the
  full branded report regardless of which view was visible.

### KTBY audit — found a real undercount in the weekly chart, fixed

User asked to double-check KTBY for errors. Cross-referenced
`political_invoices`, `rate_intel`, `rate_lines`, and the dashboard's
aggregator output:

- `political_invoices` for KTBY: **20 rows, $167,071 total gross**, 20
  distinct contracts, all 2026 cycle, all `ANCHORAGE` market,
  no NULL gross/start/year/market, no duplicate contract numbers.
  Matches `rate_intel` to the dollar.
- Dashboard hero / scoreboard / market table / station list / top
  buyers / advertiser cards: all read from `political_invoices` and
  display **$167,071** for KTBY — correct, no bug.
- **`rate_lines` for KTBY: 257 rows, $67,839 total — covers only 41%
  of the invoiced gross.** This is a known data-quality limitation of
  the line-level extraction pipeline (many invoices land in
  `political_invoices` from header totals without the line itemization
  being parseable). The weekly trend chart on the market-detail view
  was built off `rate_lines` and was therefore visually undercounting
  Coastal's Anchorage performance by ~$100K — the chart didn't tie
  back to the headline numbers.
- **Fix:** the weekly trend chart now reads from `political_invoices`,
  bucketing each invoice into the broadcast-week Monday of its
  `flight_start` via the new `weekStart()` helper (mirrors the
  `rate_lines.week_of` convention from `ratewindow-api`). The chart
  now ties exactly to the hero / scoreboard / station-list totals for
  every market, not just KTBY.
- **Tradeoff documented:** invoice-level bucketing attributes the
  entire gross to the flight_start week rather than amortizing across
  flight_start → flight_end. For a sales-pitch viewport that's fine —
  the chart shows when bookings landed, not when impressions delivered.
  An accurate delivery-pace chart would require knowing each flight's
  daily/weekly schedule, which lives in `rate_lines` (which we just
  established is incomplete).

### Daypart heatmap caveat surfaced

The daypart heatmap continues to read `rate_lines` because daypart is
not a column on `political_invoices` — there's no other source. Subhead
was updated to read "*Avg unit rate by daypart × broadcast week ·
line-level extractions only (subset of invoiced gross)*" so the user
understands the heatmap intentionally shows a partial slice; it's the
only view in the dashboard that does.

### Files touched

- `politicalwindow.com/coastal.html` — `VIEWS` array trimmed,
  `renderExports()` deleted, `weekStart()` helper added (after
  `fmtDate`), market-detail weekly aggregation rewritten to use
  invoices, weekly + heatmap subheads updated.
- `CHANGES.md` (this entry), `CLAUDE.md` "Current State" updated.

---

## 2026-05-12 — `coastal.html` follow-up: legends, date formatting, neutralized verbiage

Three fixes after first user review of the new dashboard.

### What changed

- **Removed every strategic-recommendation phrase that was not
  deducible from the source data.** Specifically:
  - The multi-station consolidation panel no longer reads "stop
    competing against yourself" / "share an audience but compete for
    buys" / "A coordinated pitch closes the gap." It now reads only
    "${N}-station market — combined totals" with the literal fact:
    `${dma_label} · N ${group.short_name} stations: ${callsigns}`.
    The three rollup tiles (combined $, combined share %, competitor
    $ in market) stay; the framing language is gone.
  - The hero gap card no longer says "money that flowed past your
    stations to rivals." It now says "across N markets with
    competitor political ad spend on file" — a description of the
    arithmetic, not a characterization of intent.
  - The Exports tile previously labeled "Sales Playbook PPTX · One
    slide per market for GSM handoff" is now "PPTX export · One
    slide per market." The handoff-target language was a
    recommendation about how to use the artifact; the artifact
    description stays.
  - Audit grep over the entire file confirms zero remaining
    instances of: `stop competing`, `coordinated pitch`, `flowed
    past`, `GSM handoff`, `leverage`, `strategic`, `optimize`,
    `recommend`. Anything left in the file describes data or
    artifacts, not strategy.
- **Two date-formatting helpers added — fix for run-over date
  labels.** The API returns `week_of` and `flight_start` as full
  ISO timestamps (e.g. `2026-03-09T00:00:00.000Z`); the v1 code did
  `.slice(5)` which still left `03-09T00:00:00.000Z` — overlapping
  axis labels (visible in the user's screenshots). New helpers:
  - `fmtWeek(d)` — compact axis tick: `2026-03-09T00:00:00.000Z`
    → `3/9` (no zero-pad).
  - `fmtDate(d)` — readable callout/table cell: `2026-03-09T...`
    → `Mar 9` (Bebas Neue display style adjacent).
  Used in: weekly trend x-axis ticks, weekly trend worst-week
  annotation, daypart heatmap top week labels, daypart heatmap
  cell tooltips, and the top-20 LUR violations table week column.
- **Both charts now have explicit legends** (only ones that lacked
  them — scoreboard, donut, station wall, advertiser cards already
  carried in-band labels).
  - Weekly trend: HTML `.chart-legend` bar above the SVG with a
    blue swatch for ${group.short_name}, a red swatch for
    Competitors, and a "Y-axis: gross $ · X-axis: broadcast week"
    note. Visible in the printed PDF too.
  - Daypart heatmap: HTML `.heat-legend` bar below the SVG showing
    a horizontal gradient strip (`linear-gradient(slate → blue →
    red)`) with the actual min and max avg unit rate dollar values
    bracketing it, plus a "Hover any cell for daypart · week · $"
    affordance hint. Daypart row labels (`EM`, `DT`, `EF`, …) now
    also get SVG `<title>` tooltips that resolve to the full daypart
    name (Early Morning / Daytime / Early Fringe / etc.) so the
    two-letter codes are decodable on hover.
- Heatmap cells widened from 8–28px to 14–36px so the grid reads
  cleanly at fewer-than-30 weeks of data; cell tooltip now reads
  `Daytime · Mar 9 · avg $1000` (full daypart name + readable date)
  instead of `DT · 2026-03-09T00:00:00.000Z · avg $1000`.

### Files touched

- `politicalwindow.com/coastal.html` only.
- This changelog entry; `CLAUDE.md` "Current State" updated.

---

## 2026-05-12 — `coastal.html` — Owner Dashboard moved to `/coastal` + visual rework

Renamed and rebuilt the Owner Dashboard from yesterday's `groups.html`:
served at the dedicated path `politicalwindow.com/coastal` (matches the
`/lur`, `/admin`, `/explorer` extension-stripped pattern). The previous
multi-tenant `?group=<slug>` engine is replaced by a per-owner file —
each owner gets their own URL and their own copy of the file. White-glove
onboarding for paid customers; matches the way each existing internal
page is one self-contained file.

### What changed

- **Path moved**: `groups.html?group=coastal-tv` → `coastal.html`,
  reachable at `https://politicalwindow.com/coastal` (no extension via
  GH Pages). `groups.html` deleted from the repo. URL params on the
  new page are only `view` / `dma` / `demo`; `group` is implicit.
- **Auth tightened**: full authentication required (login overlay
  matches the `lur.html` flow exactly — JWT in `localStorage.pw_token`
  via `POST /auth/login` against `ratewindow-api`). The page renders
  nothing data-related until `GET /auth/me` returns 200.
- **Visual rework — hero is now two-card split, not one block.**
  Left card: "${group} captured ${X}M" with the captured number set in
  Bebas Neue at 5.5rem (the dominant element on the screen). Right
  card: red-tinted gap card with "${Z}M went to competitors" at the
  same dominance — the gap number is now the *equal partner* to the
  capture number, not a half-size sub-line. Below: a 4-tile stats
  strip with colored top-edge accents (blue / amber-or-green / red
  / amber) for total addressable, share %, lost $, and LUR exposure.
- **Market scoreboard replaces the table as the primary visual.**
  Each market is now one row with three columns: market name + owned
  callsigns + networks (left) · share-of-voice stacked bar sized
  proportionally to total market $ with Coastal blue / Competitor red
  segments (center) · share % in big Bebas Neue + gap $ in small mono
  (right). Markets without coverage render a hatched "Coverage pending"
  bar instead of fabricated numbers. Sort is by total $ descending.
  The sortable table is still there below as a secondary affordance.
- **Market detail — circular share ring.** The market-detail hero now
  splits into the same two-card layout, with the right card holding
  a 220×220 SVG donut showing the market's share % filled clockwise,
  with a peer-benchmark dot positioned around the rim. Color flips
  green vs blue depending on whether share ≥ peer benchmark. Gap $
  promoted to its own full-width red card below the hero.
- **Market detail — top buyers as horizontal bars, not a table.**
  Each buyer is one row with the advertiser name, a stacked bar
  (blue Coastal | red Competitors) sized by total $, and the total
  on the right. Visual sense of magnitude is immediate.
- **Market detail — weekly trend gets gradient area fills.**
  The line chart now fills underneath both lines with vertical
  gradients (blue 0.4→0.05 for Coastal, red 0.35→0.05 for comp).
  Worst-comp-week annotation gets a 5px white-bordered dot.
- **Compliance — station wall replaces the dense table.** All 22
  Coastal stations render as a responsive grid of tiles
  (`auto-fill, minmax(190px, 1fr)`). Each tile is color-coded:
  red-tinted background with a red top edge if any LUR overcharges
  detected, green-tinted with a green top edge if clean. Tile shows
  callsign (large), market, network (uppercase), and either the
  exposure $ in Bebas Neue red OR a "✓ Clean" green badge. The hero
  for this view is also restructured into the two-card pattern
  (red total exposure card + green clean-stations count card).
- **Competition — advertiser leaderboard cards replace the table.**
  Top 25 advertisers render as a responsive grid of cards
  (`auto-fill, minmax(320px, 1fr)`). Each card shows rank #, party
  tag, advertiser name + top markets meta, total $ in Bebas Neue,
  a stacked split bar (Coastal blue | comp red) with $ labels below
  in the matching colors, and a footer with WIN RATE label + colored
  win-rate %. Hero of the competition view highlights the #1 buyer
  + the worst single loss.
- **Exports view** moved from KPI tiles to the same colored-accent
  stat-tile pattern. Print-PDF, demo toggle, and shareable link all
  inline; PPTX still deferred to v2.

### Onboarding pattern for additional owner groups

Copy `coastal.html` to `<owner-slug>.html` and edit the `GROUP`
constant at the top (slug, display name, short name, station list,
peer benchmark). One file per owner; no other code change. The
shared `GROUPS`-config-object pattern from yesterday's `groups.html`
is gone — that was the multi-tenant playground; per-owner dedicated
files are the production model. Documented in
`politicalwindow.architecture.md` §7.

### Files touched

- Created `politicalwindow.com/coastal.html` (~1,200 lines)
- Deleted `politicalwindow.com/groups.html`
- Doc updates: `CLAUDE.md` "Current State", `politicalwindow.architecture.md`
  (§4 directory layout, §7 page/route map), this changelog,
  root `political-window.architecture.md`

### Visual decisions worth flagging

- Bebas Neue at 5.5rem for hero numbers is intentionally larger than
  any other page's display type. The pitch is the gap number; the
  gap number must dominate the screen.
- Red is reserved for "money you lost" — gap, LUR exposure, lost
  buyers. Blue is "money you captured." Green is "you're clean / you
  beat the benchmark." Amber is "exposure / pending v2."
- The scoreboard bars are sized by *total market $* (not just gap),
  so big markets visibly dominate small markets even when the
  smaller market has worse share %. This communicates which markets
  are worth fighting for; the per-row share % captures the quality
  question separately.
- Stations clean vs exposed are the only two states on the wall;
  no in-between gradient. A station either has zero LUR overcharges
  detected or it has some — there's no "yellow caution" middle.

---

## 2026-05-11 — `groups.html` — Owner Dashboard viewport added (superseded 5/12)

New page: `groups.html`. A station-group owner dashboard built as a pure
**read-only viewport** over the existing `ratewindow-api` data — no
schema migrations, no new backend service, no edits to any existing
file. Designed as a sales artifact for closing SaaS license deals with
station-group owners. Coastal Television Broadcasting Group is template
#1; a stubbed second group (`example-broadcasting`) is included in the
config to prove genericity.

### What it is

Single self-contained HTML/CSS/JS file (~1,000 lines) following the
`lur.html` pattern: login overlay → JWT in `localStorage.pw_token` →
authed fetches against `https://rates.politicalwindow.com`. No new
endpoints. All aggregation happens client-side from existing
`/api/invoices`, `/api/lur`, `/api/lines` responses.

### Routing

Query-param based to fit the static-host model:

- `groups.html?group=coastal-tv` — hero (default)
- `groups.html?group=coastal-tv&view=markets`
- `groups.html?group=coastal-tv&view=market&dma=ANCHORAGE`
- `groups.html?group=coastal-tv&view=compliance`
- `groups.html?group=coastal-tv&view=competition`
- `groups.html?group=coastal-tv&view=exports`
- `groups.html?group=coastal-tv&demo=true` — anonymizes competitor
  callsigns to "Competitor A/B/C" for screenshots

Subdomain rewrites (`coastal-tv.politicalwindow.com → groups.html?group=coastal-tv`)
are documented as DNS+CNAME post-sale; no middleware needed because
nothing on this host runs server-side.

### Group config — extensibility

`GROUPS` JS object keyed by slug. Each entry carries:

- `slug`, `display_name`, `short_name`, `cycle`, `peer_share_pct`
- `stations[]` — `{callsign, dma_label, dma_db, network, state}` per
  owned callsign. `dma_db` is the UPPERCASE FCC-format DMA name as
  stored in `political_invoices.station_market` (e.g. `'ANCHORAGE'`,
  `'FARGO-VALLEY CITY'`); `dma_label` is the human display string
  (e.g. `'Anchorage, AK'`).

Adding a new owner group = one config entry. No code change anywhere
else. Coastal seeded with the 22-station footprint provided by the
maintainer (verbatim — do not re-fetch coastaltvgroup.com per the
build directive).

### Views

1. **Hero** (`view=hero`, default) — the entire pitch in one screen at
   1440px. Headline reads
   "${group} captured ${X}M of ${Y}M political dollars across your N
   markets in the ${cycle} cycle" with a half-size red sub
   "$Z.ZM went to competitor stations." Below: competitive-share
   stacked bar with peer-benchmark tick mark, three KPI tiles
   (total addressable / share % vs benchmark / LUR exposure $),
   sortable market table (default sort: Gap $ desc), CSV export.
2. **Markets** (`view=markets`) — the same market table as a
   stand-alone page.
3. **Market detail** (`view=market&dma=...`) — header with DMA total
   $, share %, gap $; multi-station consolidation panel for the 9
   multi-station markets (Anchorage, Fairbanks, Juneau, Casper,
   Cheyenne, Meridian, Jackson TN, Jonesboro, Lafayette IN);
   station comparison table (Coastal-highlighted); top-10 buyers
   in market with Coastal vs comp split; weekly trend SVG line
   chart with worst-week annotation; daypart × week heatmap of
   Coastal-side under-indexing.
4. **Compliance** (`view=compliance`) — KPI tiles (total LUR
   exposure / refund-risk top station / clean-station count); two
   tables (stations by LUR exposure, top-20 violation flights).
   Print-PDF button on the violations table.
5. **Competition** (`view=competition`) — top-25 advertisers across
   the entire footprint with total $ / Coastal capture / lost to
   comp / win-rate bar / top-3 markets. CSV export.
6. **Exports** (`view=exports`) — Owner Briefing PDF (browser print
   stylesheet — see below); PPTX deferred to v2 with explicit "v2"
   tile; shareable read-only link with copy-to-clipboard.

### Owner Briefing PDF — print stylesheet approach

`@media print` block hides the header, footer, and tabs; widens the
page; switches the body to white; ensures cards page-break-inside:
avoid. A `.print-cover` div renders only on print and provides the
"Prepared for ${group}, ${date}" cover page. A `beforeprint` event
appends a `#print-briefing` block with the full hero numbers + market
table + LUR exposure + top-15 advertisers, regardless of which view
the user is currently in. `afterprint` removes it. Result:
File → Print → Save as PDF in any browser produces a branded,
multi-page Owner Briefing without a server. PPTX generation is
explicitly deferred to v2 (would require adding `pptxgenjs` to
`ratewindow-api`).

### Data flow per page load

For the active group:

1. `Promise.all` fetches `/api/invoices?state=<st>&limit=2000` per
   distinct state in the group's footprint (≤9 calls for Coastal).
   Filtered client-side to invoices whose `station_market` matches
   the group's `dma_db` set.
2. `Promise.all` fetches `/api/lur?callsign=<cs>` for every owned
   callsign (22 calls for Coastal).
3. `Promise.all` fetches `/api/lines?station=<cs>&limit=2000` per
   owned callsign for the daypart × week heatmap and weekly trend.
4. Aggregation runs client-side: per-DMA totals, Coastal vs comp
   split, top advertisers, LUR exposure (charged − LUR × spots,
   summed), weekly series, daypart matrix.
5. Coverage indicator: any DMA with zero indexed invoices renders a
   `No coverage` tag on its row instead of fabricating numbers
   (per the build directive: "no placeholder data in the live
   view").

### Auth

Reuses the existing `ratewindow-api` JWT flow exactly as in
`lur.html`: `POST /auth/login` → `localStorage.pw_token` →
`Authorization: Bearer` on every authed fetch. Registration accepts
the existing `invites` token. Per-group invite scoping is **not**
added in v1 — anyone with a valid `pw_token` can hit `groups.html`.
A future schema-additive `group_invites` table can introduce
group-scoped access without breaking the existing flow; deferred
until first paying customer signs.

### Design

Matches the production token set in `TOKENS.md` (light theme — the
"Bloomberg dark navy" line in `CLAUDE.md` is stale per the
inconsistency note in `politicalwindow.architecture.md` §5).
Executive treatment: more whitespace than the operational pages,
display typeface (Bebas Neue) on hero numbers, restrained palette
(blue + red + amber + green only), no chart junk. Microsoft Clarity
tag included (per `CLAUDE.md` "all 7 HTML pages" convention; this
makes 8). No GA4 — internal/auth-gated page like `lur.html` and
`admin.html`.

### Files touched

Only `politicalwindow.com/groups.html` was created. Zero edits to
`index.html`, `lur.html`, or any other existing page. Zero schema
changes. Zero changes to `ratewindow-api` or `politicalwindow-api`.

### Future v2 candidates

- Group-scoped auth (additive `group_invites` table; route gating)
- Server-rendered Owner Briefing PDF (replace browser-print with a
  `pdfkit` endpoint on `ratewindow-api` for richer layout)
- Sales Playbook PPTX (`pptxgenjs` on `ratewindow-api`)
- Subdomain rewrite at the CDN layer (Cloudflare Worker or similar)
- Federal/state/local + party + PAC-type filters on the
  Competition view (currently shows totals only)

---

## 2026-05-07 — Microsoft Clarity analytics installed

Added the Microsoft Clarity tag (project ID `wnbzwo190n`) to every HTML
page so we capture session recordings, heatmaps, and rage-click signal
across the full site, not just the GA4 funnel.

### What changed

- **All 7 HTML pages** now load the Clarity loader snippet inside
  `<head>`, **after** the existing `gtag.js` block (or immediately after
  `<head>` for the two pages that don't carry GA4):
  - `index.html` — main map / state-detail SPA
  - `candidate-tracker.html` — candidate spend table
  - `explorer.html` — FCC rate explorer
  - `public-file.html` — public-file activity feed
  - `rates.html` — candidate rate intelligence
  - `admin.html` — admin tools (no GA4 block; Clarity inserted directly
    after `<head>`)
  - `lur.html` — LUR monitor (no GA4 block; same insertion pattern)

### Snippet (identical on all 7 pages)

```html
<!-- Microsoft Clarity -->
<script type="text/javascript">
    (function(c,l,a,r,i,t,y){
        c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
        t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
        y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window, document, "clarity", "script", "wnbzwo190n");
</script>
```

### Notes / why it's safe

- The loader is async (`t.async=1`) and inserts a remote `<script>`
  before the first existing `<script>` in the document — no blocking on
  first paint, no interaction with MapLibre, gtag, or any of our inline
  bootstrap code.
- No build step touched. Single-file SPA constraint preserved — the
  snippet is inline, no new asset references beyond the Clarity CDN.
- Coexists with GA4 (`G-JGHV19ZYEQ`); Clarity uses its own `clarity`
  global, no collision with `dataLayer` / `gtag`.
- Project ID `wnbzwo190n` is the production Clarity project. If we add
  staging later, gate the loader on hostname.

### How to verify post-deploy

1. Hit politicalwindow.com in a fresh browser, open DevTools → Network,
   filter `clarity.ms` — expect a request to
   `https://www.clarity.ms/tag/wnbzwo190n` returning 200.
2. `window.clarity` should be defined in the console.
3. Within ~30 min, the new session shows up in the Clarity dashboard.

---

## 2026-04-24 — Session close: merged 2+3 (frontend half)

Bookmark for next session. Branch `fix/candidate-data-2026-04-24`,
3 commits, not pushed/merged at session close. Companion bookmark in
`../politicalwindow-api/CHANGES.md`.

### Landed

Three commits, all in `index.html`. Together they close the IL-8
"Christ Kallas alphabetically wins" bug and the "JB Pritzker vs
JB PRITZKER" cross-API duplicate.

- **`cc59045` — plumb status string.** `loadLiveData` carries the
  Title-Case server `status` (Declared/Nominee/Lost Primary/In
  Runoff/Retiring/Exploring) onto every CANDS row alongside the
  legacy 3-value `s` code. `normalizeEntity` adds `status: null`
  to tracker entities. `mergeCuratedIntoEntities` backfills status
  onto tracker entities from the CANDS-side row. Incidental fix:
  the `Exploring` badge regression caused by Item 4a's Title-Case
  backfill (lowercase compare missed the new value) — now case-
  insensitive.

- **`95809d9` — dedupe key normalization with merge-on-collision.**
  New `normalizeNameKey` helper mirrors the SQL audit normalizer
  and adds a `"LAST, FIRST" → "FIRST LAST"` reorder. Composite key
  `(normalizeNameKey, district)` replaces the prior `name.toLowerCase()`
  Set-based dedupe. On collision the function MERGES rather than
  drops — CANDS supplies authoritative display fields (name, party,
  incumbent, status), tracker preserves financial signal (spend,
  fec_id, entity_id). Closes the JB Pritzker symptom and the
  district-shadowing bug noted as Item 1 follow-ups.

- **`47dd0ad` — status-driven post-primary selection.** Replaces the
  alphabetical-first-name fallback in `renderPartyGroup` post-primary
  branch with status-aware selection. Adds five new `.cbadge-*`
  variants (NOMINEE, RUNOFF, LOST PRIMARY, RETIRING, EXPLORING), a
  `.cparty-runoff-chip` element next to the party letter for In
  Runoff candidates, a "Primary completed [date]. Nominee data
  forthcoming." banner above the party groups in post-primary states
  with no Nominee row, a `<details>`-based Lost Primary disclosure
  below the party groups, and a pre-primary "See all N declared
  candidates" affordance. New temporary helper `getStateElectionPhase`
  built on existing date fields — Item 4b will refine without
  changing call signature. Honors the no-client-storage rule
  (disclosure state is DOM-local, not persisted).

### Backend dependency

None at merge time. The 3 commits read `cand.status` which the API
emits Title-Case after the API-side commit `0c22c5e` landed earlier
in the session.

### IL-8 user-visible behavior change

Before: Christ Kallas alone displayed in the IL-8 Democratic slot,
because the post-primary fallback selected the alphabetically-first
row when no candidate had tracker spend.

After: banner shows "Primary completed 2026-03-17. Nominee data
forthcoming." All 10 IL-8 Democrats render below, sorted by spend
desc. Same correction applies to every post-primary state with no
Nominee data yet — until Item 4c lands and tags actual nominees.

### Deferred (next session)

- **Item 4b — state election phase.** `getStateElectionPhase` is a
  temporary impl; refine the boundaries (Primary window width,
  General cutoff alignment) and surface the phase as a UI chip in
  the state-detail sidebar. Call sites stable, only the helper body
  moves.
- **Item 4c — primary results.** Plan-only at session close. Once
  shipped, `Nominee`/`Lost Primary`/`In Runoff` rows appear in the
  DB and the dormant code paths added in `47dd0ad` activate.
- **Item 4d — visual continuity.** Blocked on 4b's state-phase chip
  + 4c's status data flowing.

### Followups (smaller scope, no item assigned)

1. **Same-district same-name dedupe collision.** Composite key in
   `mergeCuratedIntoEntities` may over-collapse pairs like "Joe
   Kennedy III" / "Joe Kennedy" in the same district —
   `normalizeNameKey` strips JR/SR/II/III/IV. Mitigation: add
   `fec_id`/`external_id` to the composite key as a tiebreaker.
2. **`countShownInPartyGroups` + `renderPartyGroup` drift risk.**
   Two implementations of the same MIN_FLOOR=2 / top-3 slice math
   in `index.html`. Will drift on next touch; refactor to a shared
   helper next time either is edited.

(Companion CHANGES entry in `politicalwindow-api/CHANGES.md` lists
backend-side followups: `seed.js` DELETE-FROM footgun guard, and
the deferred seed-cycle audit for other off-cycle Governor
miscodings.)

---

## 2026-04-24 — LUR invoice review modal: per-panel selection + matched-line indicator removed + "COMPARE INVOICES" caps

**Scope.** `lur.html` only. Three changes to the Invoice Review modal
(the pop-out that opens when a user clicks the "compare invoices" button
on a rate-variance row).

### 1. Per-panel independent invoice selection

**Before.** When a variance group had 3+ buyers, the modal rendered a
single row of tabs at the top labeled `Invoice A vs Invoice C`,
`Invoice B vs Invoice C`, etc. The right-hand panel was hard-wired to
the LUR (lowest-rate invoice, always `sorted[sorted.length-1]`); tabs
only rotated which *other* invoice landed in the left panel. Users
could not compare, e.g., A vs B directly.

**After.** Each panel has its own selector row directly above its PDF
viewport, listing every invoice in the group (`Invoice A / Invoice B /
Invoice C / ...`). Left and right selections are fully independent, so
any pair (including same-on-both) is viewable. The active invoice on
each side gets the blue underline; styling deliberately matches the old
tab strip — same monospace-caps text, same faint→blue active state.

**Implementation.**
- Removed `<div class="inv-tabs" id="inv-tabs">` from the modal HTML
  and removed the `.inv-tabs` / `.inv-tab*` CSS.
- Added `.inv-panel-selector` / `.inv-panel-tab` CSS (visually
  identical to the old tabs).
- `openInvoiceReview` no longer populates a top tab strip; it stores
  `overlay._leftIdx = 0` and `overlay._rightIdx = sorted.length - 1`
  (initial view preserves the old behavior: highest-rate left, LUR
  right) and calls the new `renderInvoicePanels`.
- New `renderInvoicePanels(sorted, leftIdx, rightIdx, lurRate)` —
  full rebuild of both panels on modal open.
- New `renderInvoicePanelSide(side, sorted, idx, lurRate)` — re-renders
  only the affected panel on selector click; preserves the other side's
  PDF scroll + rotation state.
- `buildPanelHTML` was refactored to take
  `(side, sorted, idx, lurRate)` instead of
  `(row, label, idx, isLur, lurRate)`. It now renders the selector
  row inline at the top of each panel and derives the "is this the
  LUR?" flag from the row's rate (`rate === lurRate`) instead of
  position, so the green rate color + "LUR Rate" footer badge follow
  the invoice regardless of which panel it ends up in.
- Each `.inv-panel` got a `data-side="left|right"` attribute so the
  single-side re-render can replace it in place.
- `switchInvoiceTab(tabIdx)` was replaced by
  `selectInvoice(side, idx)`; the onclick on each `.inv-panel-tab`
  calls it.
- Removed `renderInvoicePair` (replaced by the two functions above).

### 2. Matched-line indicator removed

**Before.** After rendering each PDF page, `renderPanelPages` scanned
the text content for the row's `order_number` / `line_number` / `rate`,
scored candidate rows by keyword hits, and drew:
- a translucent amber highlight bar (`.inv-pdf-highlight-row`)
  across the matched row,
- an orange `"▶ MATCHED LINE"` tag (`.inv-pdf-highlight-tag`) at its
  left edge, and
- scrolled the viewport to center the match.
On miss, it prepended a red `.inv-pdf-banner` that read
`Locate Order #… in the document below`.

**After.** Removed entirely. `renderPanelPages` now just renders each
page as a canvas. No text scanning, no highlight, no tag, no banner.
Rationale from user: the per-panel selector now makes it obvious which
invoice is in view; the highlight was judged more intrusive than
helpful.

**Implementation.** Stripped the inner `.then(function(textContent){…})`
block, the `scrollTarget` scroll-into-view, and the miss-banner from
`renderPanelPages`. Removed the unused `.inv-pdf-highlight-row`,
`.inv-pdf-highlight-tag`, and `.inv-pdf-banner` CSS rules, along with
the unused `.inv-pdf-highlight` rule (no longer referenced anywhere).

### 3. "compare invoices" → "COMPARE INVOICES"

Label text inside `.inv-review-label` on the variance-table row button
(pre-pop-out) is now all-caps `COMPARE INVOICES`. Styling (monospace,
0.48rem, red, 0.02em letter-spacing) and positioning unchanged — this
is a pure text change.

### Verification

- `grep -n 'switchInvoiceTab\|renderInvoicePair\|inv-tabs\|MATCHED' lur.html`
  returns only comments in JSDoc describing the removal.
- `node --check` on extracted `<script>` block passes clean.
- No changes to `CLAUDE.md` rules; added a short note under
  "Current State (Apr 2026)".

---

## 2026-04-16 — Favicon on every page + sidebar size consistency

**Symptom.** User flagged two UI gaps:
1. The browser-tab favicon only rendered on `/` — every other page
   (`/public-file`, `/candidate-tracker`, `/rates`, `/lur`, `/explorer`,
   `/admin`) showed the browser default icon.
2. The left sidebar rendered taller on `/` than on subpages. The
   `.sidebar-panel-link` rule on `index.html` used `font-size:0.72rem`,
   `padding:7px 16px 7px 18px`, `border-left:3px solid transparent` —
   every other page used the more compact `0.68rem`, `6px 16px`,
   `2px` border. Result: same nav items rendered ~3px taller per row
   on the home page and drew noticeably bolder.

**Fixes.**
1. **Favicon.** Copied the inline SVG `data:image/svg+xml,...` favicon
   `<link>` (the orange-gradient `PW` glyph matching `.sidebar-rail-logo`)
   from `index.html` into every other page (`public-file.html`,
   `candidate-tracker.html`, `rates.html`, `lur.html`, `explorer.html`,
   `admin.html`). The data-URI approach preserves the single-file SPA
   constraint — no separate `.ico`/`.png` ships.
2. **Sidebar link size.** Changed `index.html` `.sidebar-panel-link` to
   match the 5 subpages (`font-size:0.68rem; padding:6px 16px;
   border-left:2px solid transparent;`). Chose to shrink index rather
   than grow the subpages because 5 pages already use the compact style.

**Verification.** `grep -l 'rel="icon"' *.html` returns all 7 pages.

---

## 2026-04-16 — Consistent candidate party/office on /public-file (backend fix)

**Symptom.** On `/public-file`, the Contract Detail table showed the
PARTY column inconsistently for the same candidate across sibling rows.
Example from the user screenshot (Apr 9 2026, Spectrum LA, CA Governor):

- Antonio Villaraigosa — 1 row with `D`, 4 rows with `—`
- Becerra — 2 rows, both with `—` (though the group header pill still
  showed `Becerra $119K` with a D-colored pill context because party was
  inferred from *at least one* sibling row during group assembly, which
  only amplified the per-row inconsistency)

Because the PARTY column is color-coded (`pc(r.party)` → `party-D` /
`party-R` / `party-I`), the effect was visually jarring — the same
candidate name would render blue, then neutral gray, then blue again
down a single table.

**Root cause.** The fix is entirely backend. This changelog entry
exists because the rendering site is on the frontend (`public-file.html`
line 508: `<td class="'+pc(r.party)+'">'+esc(r.party||'—')+'</td>`) and
frontend engineers inspecting the bug would start here.

`ratewindow-api`'s `/api/rates` endpoint (`index.js` lines ~150-175)
resolved the *candidate name* through a `political_entities` join —
`COALESCE(pe.candidate_name, ri.candidate_name) AS candidate` — but
selected `ri.party` and `ri.office_type` raw. Claude's PDF extraction
frequently misses the party field on a given invoice (especially on
non-federal races like Governor, where FEC lookup is skipped), so
`rate_intel.party` is NULL on many rows even when the canonical entity
record has party filled in.

**Fix (in `ratewindow-api`).** `/api/rates` and `/api/lines` now
COALESCE party and office through `political_entities` first:

```sql
COALESCE(pe.party,  ri.party)       AS party
COALESCE(pe.office, ri.office_type) AS office_type
```

`/api/lines` uses a two-hop chain (`pe.* → pe2.* → rl.*`) because it
joins the entity both directly via `rl.political_entity_id` and through
`advertiser_aliases`.

The `/api/rates` party/office filter predicates also moved to the
resolved value so `?party=D` returns rows where the resolved party is D
(not just rows where Claude happened to extract it), matching the
displayed column.

A new one-time script `backfill-entity-party.js` propagates the most
common non-null `rate_intel.party` / `rate_intel.office_type` up into
`political_entities` where those columns are still NULL — needed for
candidates like Becerra whose canonical entity was auto-created from a
Claude extraction that didn't include party. See the ratewindow-api
CLAUDE.md "Candidate-Detail Consistency Rule" section for the invariant
future endpoints must follow.

**Frontend.** No frontend code changed. `public-file.html` renders
whatever the API returns; the color-coding, group-header party pills,
and candidate slicers now all agree because the API now returns a
single party value per (candidate, row) pair.

---

## 2026-04-15 — Drop `??` district buckets + refit-on-slicer-change

**Context.** Two small UX fixes bundled in the same user thread.

### 1. Remove `{STATE}-??` buckets from the House district list

**Symptom.** CA Governor's candidate tab showed a `CA-??` row in the
House district list. Clicking through revealed real candidates
(`Saikat Chakrabarti`, `Kevin Kiley`) who should have been under their
proper CDs (CA-11 and CA-03 per FEC IDs H6CA11219 / H2CA03157). Same
issue was latent across every multi-district state — any House
candidate whose tracker-API feed returned `district: null` ended up
in a `??` bucket.

**Root cause.** Two steps:
1. `ratewindow-api`'s `/api/candidates/tracker?state=CA` sometimes
   returns House entities with `district: null`. That's a data quality
   issue upstream — the committee-linking code there doesn't always
   resolve the CD. Our own FEC scraper in `politicalwindow-api` does
   resolve it correctly, so `CANDS` (loaded from `/api/candidates`)
   has the authoritative district.
2. `mergeCuratedIntoEntities(abbr, 'House', …)` deduped the two
   sources by `name.toLowerCase()`, with tracker winning. When the
   tracker entity had `district: null`, the correctly-districted CANDS
   twin got dropped on the floor, and `renderHouseOffice` then bucketed
   the null-district tracker entity into a `'unknown'` district that
   renders as `${abbr}-??`.

**Fix.**
- `mergeCuratedIntoEntities` (index.html ~2693): before the existing
  dedupe loop, build a `name → district` map from `CANDS` (only for
  `office === 'House'`) and backfill missing districts onto tracker
  entities. Atlarge states collapse to `'AL'`; multi-district states
  get the numeric CD as a string. This preserves tracker spend totals
  while recovering the district.
- `renderHouseOffice` (index.html ~2732): after backfill, any House
  entity still lacking a district in a multi-district state is
  dropped from the district list entirely (instead of being bucketed
  into `'unknown'`). At-large states keep their `'AL'` fallback —
  they're the only legit single-bucket case.
- `rowLabel` in the district list (~index.html:2799) no longer has
  the `d.district === 'unknown' ? '${abbr}-??' : ...` branch. Dead
  code removed.

**Validation.** Open CA, Candidates tab → House slicer:
- Before: district list shows `CA-01 … CA-52 … CA-??`.
- After: district list shows `CA-01 … CA-52` with no `??` row.
- `Saikat Chakrabarti` now appears under CA-11; `Kevin Kiley` under CA-03.

### 2. Refit to state bounds when leaving a drilled-in House CD

**Symptom.** When the user drills into a House district (map zooms in
to the CD), then switches the Candidates slicer to Senate or Governor,
the map stayed zoomed to the CD — giving a confusing mismatch between
the candidate list (statewide) and the map viewport (one district).

**Fix.**
- New helper `fitStateBounds(abbr)` (~index.html:3072) that refits the
  map to a state's full bounds using the same logic `mapSelectState`
  uses, but without touching feature-state or tearing down district
  layers.
- Slicer click handler in `wireCandidatesPane` (~index.html:2853) now
  checks if the user was drilled into a House district
  (`st.office === 'House' && st.district`) and, if the new slicer is
  Senate/Governor, calls `fitStateBounds(abbr)` after resetting state.

**Not touched.** The explicit "‹ All Districts" back button (inside
the House pane) still leaves the map at its current district zoom —
that's a deliberate breadcrumb-style back-to-district-list action, not
a context switch away from House. If the user eventually wants that
to zoom out too, we can revisit.

---

## 2026-04-15 — All 435 House districts + brighter palette

**Spec.** User directive: *"this is the us house of reps. every seat is up
every 2 years so you should have 'hover over' and candidate listing with
at least incumbent if not declared competitor for every single district
on the map. use the FEC API and get all of this info reflected on the
map. and make colors a bit less dull."*

**Backend status (audited first — nothing to change).** The FEC pipeline
was already wired end-to-end before this session:

- `../politicalwindow-api/scraper.js` pulls from `api.open.fec.gov/v1/candidates/`
  for every `(state, office='H')` pair whose state has `'House'` in its
  `states.races` array.
- `seed.js` already includes all 51 jurisdictions (50 states + DC) with
  `'House'` in every `races` array — confirmed via regex audit.
- `FEC_API_KEY` is set as a Railway env var; daily cron at 6 AM ET has
  been populating the `candidates` table for weeks.
- Live audit against `https://api.politicalwindow.com/api/candidates`:
  - Total House candidates: **1,688**
  - Incumbents on file: **426** (roughly matches the 435 seat count;
    9 diff is unfilled open seats / retirements not yet backfilled)
  - Party breakdown: D 948, R 678, I 62
  - 51/51 states have at least some House rows
  - CA: 52 districts populated, 50 with an incumbent on file
  - TX-1 spot-check: Moran (R-inc) + 2 D challengers ✓
  - WY-AL spot-check: Hageman (R-inc) + 3 R challengers ✓

So the "missing districts" problem was entirely a **frontend** issue: the
UI was filtering the map highlight + candidates tab to only districts
backed by tracker-API spend records, which is a much sparser dataset than
the FEC candidate filings. The fix is in `index.html` only; no backend
code, seed, or DB writes touched.

**Frontend changes (all in `index.html`).**

1. **`loadLiveData` CANDS transformation preserves district.** The
   transform at line ~3476 was dropping `district` on the floor when
   converting `/api/candidates` rows into the frontend `CANDS` shape.
   Now pushes `d: c.district` so `getHouseDistrictInfo` and
   `mergeCuratedIntoEntities` can read it. Senate/Governor rows still
   come through with `d: undefined`.

2. **`mergeCuratedIntoEntities` preserves House district.** Line 2638
   previously hardcoded `district: null` for all curated pushes. Now:
   - For `office === 'House'`, it reads `c.d` from the CANDS entry,
     parses to an int, and resolves to `'AL'` when the state is at-large
     (`DISTRICT_COUNTS[abbr] === 1`), otherwise the numeric string.
   - Also now passes through `incumbent: c.s === 'inc'` so downstream
     renderers can still flag incumbency on curated-only entries.

3. **`renderHouseOffice` merges tracker + CANDS.** Previously took
   `getCandidatesFor(abbr, 'House')` (tracker-only) and bucketed by
   district. Now wraps that in `mergeCuratedIntoEntities(abbr, 'House', ...)`
   so every district with any FEC-filed candidate appears in the
   district list, even with no tracker spend recorded. CANDS-only
   entries carry `spend: 0` and surface as `no spend reported` in the
   district row (existing fallback path).

4. **`highlightActiveDistricts` lights up every district.** Previously
   built the active set from tracker entities via `getCandidatesFor`.
   Now iterates the loaded GeoJSON feature collection:
   ```javascript
   const active = new Set();
   (DISTRICT_CACHE[abbr].features || []).forEach(f => {
     const n = f.properties && f.properties.district_number;
     if (n != null) active.add(String(n));
   });
   ```
   Every district in the selected state paints the bright overlay
   on selection.

5. **New helper `getHouseDistrictInfo(abbr, districtNum)`.** Returns
   `{incumbent, challengers, all}` for one CD, reading from CANDS.
   - Looks up `CANDS[abbr]`'s House race list
   - Filters by `d === target` (parseInt both sides)
   - Handles at-large states: `DISTRICT_COUNTS[abbr] === 1` matches any
     entry regardless of its district number (the data sometimes stores
     at-large as 0, sometimes as 1, and the FEC sometimes emits both)
   - Picks the first `s === 'inc'` as the incumbent, rest as challengers

6. **New district-level mousemove tooltip.** Added to
   `wireMapInteractions` alongside the existing district hover feature-state
   toggle. Reuses the `#mapTooltip` element. Content:
   - `STATE-CD` header (or `STATE-AL` for at-large)
   - `HOUSE` label (small, faint)
   - Incumbent line: party code (colored via `--dem`/`--rep`/`--ind`),
     name, `INCUMBENT` chip — or "No incumbent on file" in faint text
   - Up to 3 challenger rows with party color + name
   - `+N more` tail if there are more than 3 challengers
   - Border-left color from `STATUS_COLORS[state.status]`
   - Positioning logic matches the state-level tooltip (flips to the
     left edge when it would overflow the map container)

7. **Palette brightness pass.** User said "a bit less dull" — bumped all
   four status hues one step toward the generic palette (which is still
   one step back from the original "box of crayons" set). Affected:
   - `--status-open`: `#991B1B` → `#B8221E` (+saturation, +lightness)
   - `--status-soon`: `#B45309` → `#D1680F` (brighter amber)
   - `--status-upcoming`: `#1D4ED8` → `#2563EB` (brighter blue)
   - `--status-general`: `#1D4ED8` → `#2563EB` (kept synced with upcoming)
   - `--status-none`: `#94A3B8` → `#8B95A5` (slightly warmer slate)
   All dependent rgba tile backgrounds, spill backgrounds, and the
   `STATUS_COLORS` JS object were updated in lockstep so the MapLibre
   `match` expressions, insight-panel status dots, and detail-panel
   accents all shift together. Generic `--red`/`--amber`/`--blue`/`--green`
   still back the KPIs, buttons, and `✓ COMPLETE` chip — unchanged.

**Verification done.**

- Inline `<script>` blocks parse cleanly via `new Function(...)`:
  ```
  Inline scripts found: 2
  All inline scripts parse OK
  ```
- Live API spot-checks (TX-1, CA-12, WY-AL) return correct
  `{incumbent, challengers, all}` shapes via the new helper logic
  simulated against real `/api/candidates` data.
- CA coverage: 52/52 districts present, 50 with incumbent on file
  (remaining 2 are open seats).

**Verification still needed (manual, in-browser).**

- [ ] Load the production page, select a state, and confirm every
  district lights up (not just the 1–3 that used to).
- [ ] Hover over a district and verify tooltip shows `STATE-CD` +
  incumbent + challenger list.
- [ ] Click a district and verify drill-in still works (the click
  handler was untouched but the layer filter changed, so worth
  double-checking).
- [ ] Verify at-large states (AK, DE, ND, SD, VT, WY) still render
  one district with the `AL` label (not duplicated per polygon).
- [ ] Eyeball the new palette — less dull but still terminal.

---

## 2026-04-15 — Merge "General Only" + "Upcoming" into single blue category

**Spec.** User request: *"let's just combine 'General Only' and 'Upcoming'
into one category called 'Upcoming' and make all of those states blue."*

**Context.** Earlier the same day, `--status-upcoming` had been swapped
from violet `#6D28D9` → teal `#0F766E` after the violet collided with the
governor pill hue. User then asked whether teal was the most balanced
choice; the honest answer was no — teal compressed the cool half of the
palette against navy (`--status-general: #1E3A8A`) and slate
(`--status-none: #94A3B8`), and it broke the warming ramp of
red→amber→(cool). Rather than iterate on a third hue, the user decided
to collapse the two primary-complete-but-waiting phases into one visual
category and pick blue.

**Semantic vs. visual collapse.** The internal `general-only` status
value is still produced by `classifyStatus()` (line ~1271) for states
whose primary date is past, and it's still consumed by:

- `isPrimaryComplete()` — drives whether the Candidates tab shows
  post-primary nominees vs. in-primary top-3
- The `✓ COMPLETE` spill-green chip on the state table/cards — a
  functional indicator that stays green, independent of the status hue

So the status value stays bifurcated in JS. What unifies is the *color*
and the *label* that the user sees.

**Changes (all in `index.html`).**

1. **CSS variables** (lines 47–48). Both `--status-upcoming` and
   `--status-general` now resolve to `#1D4ED8` (the same hex as the
   generic `--blue`). The `--status-general` var name is retained so
   the ~30 call sites referencing it keep compiling.
2. **Tile backgrounds** (lines 333–334). `.tile-upcoming` and
   `.tile-general-only` backgrounds both set to `rgba(29,78,216,0.10)`.
3. **Status spills** (lines 539–540). `.spill-status-upcoming` and
   `.spill-status-general` backgrounds and borders both set to
   `rgba(29,78,216,0.08)` / `rgba(29,78,216,0.22)`.
4. **Legend** (line 938). Removed the `General Only` legend row
   entirely; only the unified `Upcoming` (`dot-purple`) row remains.
   The `.dot-green` CSS rule at line 240 is now unused by the legend —
   left in place since it resolves via `--status-general` and could
   still be referenced elsewhere in the future.
5. **STATUS_COLORS JS** (lines 1327, 1330). Both `'upcoming'` and
   `'general-only'` entries now resolve to `#1D4ED8`. This drives
   the MapLibre state-fill match expression, so every state in either
   phase paints the same blue on the map.
6. **Hover tooltip label** (line 1611). `'general-only'` status label
   changed from `GENERAL ONLY` to `UPCOMING`.
7. **SL (state-detail status line)** (lines 2301, 2303). `'upcoming'`
   shortened from `○ PRIMARY UPCOMING` to `○ UPCOMING`; `'general-only'`
   changed from `◇ GENERAL WINDOW ONLY` to `○ UPCOMING`. Same glyph,
   same label — visually indistinguishable.
8. **State detail post-primary override** (line 3042). The override
   branch that shows `✓ PRIMARY COMPLETE · GENERAL WINDOW` for
   general-only states with a primary date now reads `✓ PRIMARY
   COMPLETE · UPCOMING`.
9. **SO sort function** (line 3256). Previously ranked
   `upcoming:2, general-only:3, inactive:4` (inactive shifted); now
   ranks both at 2, with `inactive` moving from 4 → 3. The table's
   sort order no longer distinguishes the two visually-merged phases.
10. **SPILL map** (line 3267). `'general-only'` spill changed from
    `◇ GEN ONLY` to `○ UPCOMING` (still using `spill-status-general`
    class, which now shares the same blue paint as `spill-status-upcoming`).

**What deliberately did NOT change.**

- `classifyStatus()` still emits `'general-only'` as a distinct value.
- `isPrimaryComplete()` still branches on `'general-only'`.
- The `✓ COMPLETE` spill-green chip rendering on the state table
  (lines 3331, 3369) still uses the generic `.spill-green` class with
  the generic `--green`. That chip indicates primary completion
  regardless of map overlay and is functionally separate from the
  window-timing palette.
- `WC` (line 2310) still maps `'general-only' → 'var(--status-general)'`
  — it works because both vars now point at the same blue.
- `statusDot` in the insight panel (line 3654) similarly resolves via
  the shared var.
- The `.dot-purple` class name (line 239) is now slightly misleading
  (it paints blue) but is left as-is to avoid churning every
  `legend-dot dot-purple` call site for a cosmetic rename.

**Verification checklist.**

- [ ] Grep for `GEN ONLY`, `GENERAL ONLY`, `GENERAL WINDOW ONLY`,
  `PRIMARY UPCOMING` — should return zero label hits in `index.html`
  (the `general-only` status *value* should still appear).
- [ ] Load the map and visually confirm CA/NY/NJ/etc. (upcoming) and
  VA/NJ (general-only post-primary) all paint the same blue.
- [ ] Click a post-primary state and verify the detail header reads
  `✓ PRIMARY COMPLETE · UPCOMING` in blue.
- [ ] Verify the table column status shows `○ UPCOMING` for both
  phases (unless primary is complete, in which case `✓ COMPLETE`
  green chip still overrides).
- [ ] Legend bar shows 4 items instead of 5: Window Open, Opens <30d,
  Upcoming, None.

---

## 2026-04-15 — Congressional district hover matches state hover

**Spec:** user asked for the hover effect on active congressional districts
to visually match the hover effect on states in the full-map view. The
state hover had a distinctive three-piece "tighten up" feel the district
hover was missing.

**Analysis.** Before this pass, state hover (in `addStatesLayer`) had three
coordinated paint-expression deltas:

1. **Fill opacity**: 0.45 → 0.6 on `feature-state.hover`
2. **Outer blurred glow**: `line-width` 0 → 4, `line-blur` 4,
   `line-opacity` 0 → 0.35 — colored via the same `STATUS_COLORS` match
   the fill uses
3. **Crisp border snap**: `line-color` `rgba(148,163,184,0.5)` → `#64748B`,
   `line-width` 0.6 → 1.5, `line-opacity` 1

Active-district hover (in `highlightActiveDistricts`) had only pieces 1
and 2, with the following mismatch:

- Fill hover opacity was 0.72 (vs state's 0.6)
- Glow was wider + softer: `line-width` 5 → 8, `line-blur` 8,
  `line-opacity` 0.28 → 0.45
- **No crisp border snap at all** — only the always-on dashed
  `districts-line` base layer

The missing border snap was the biggest piece. State hover reads as
"tight and interactive" because of that crisp slate-500 outline
appearing out of nowhere; the district hover just brightened and felt
soft by comparison.

**Change.** `highlightActiveDistricts(abbr)` is now a three-layer
treatment. The baseline (what active districts look like *before* hover)
intentionally stays visible — this is the signal that a district has an
active race — but the hover *delta* on every piece now matches the state
hover exactly:

- **`districts-active-fill`** — hover opacity pulled from 0.72 to 0.6
  so it snaps to the same value state-fill does on hover. Baseline
  stays 0.55 (brighter than the state baseline of 0.45, because active
  districts should read as lit before hover).
- **`districts-active-glow-outer`** — tightened to match the state
  glow's dimensions: `line-blur` 8 → 4, `line-width` 5/8 → 2/4,
  `line-opacity` 0.28/0.45 → 0.20/0.35. Baseline keeps a restrained
  always-on glow so the district isn't invisible before hover, but the
  on-hover value (4 / 0.35) is identical to the state glow's on-hover
  value.
- **`districts-active-border`** (NEW) — a third paint layer that
  mirrors the `states-border` hover snap piece-for-piece. Invisible at
  rest (`rgba(100,116,139,0)` at `line-width` 0), snaps to `#64748B` at
  `line-width` 1.5 on `cand-hover`. Rendered on top of the glow and
  filtered to the same active-district set, so the hover border
  visually "tightens" the shape the same way state hover does.

**Baseline vs delta (why both can match).** "Match the state hover" had
two plausible readings: (a) make districts look identical to states
even at rest, losing the always-lit signal; or (b) transplant the state
hover *character* onto the district baseline while keeping active
districts visible before hover. Went with (b). The hover-state values
of all three paint expressions are now literally identical to the
state hover values; only the rest-state of fill + glow stays elevated
to preserve the designed activity signal.

**Teardown.** `districts-active-border` is added to both `hideDistricts`
and `clearCandDistrictHighlight` so state deselect / district fade-out
properly remove the new layer alongside the other two.

**No changes to event wiring.** The existing
`mousemove`/`mouseleave`/click handlers on `districts-active-fill`
still drive the `cand-hover` feature-state, which all three paint
layers read from. The new border layer is filter-only; no new event
listener needed.

---

## 2026-04-15 — Cable PSID → retail brand display across all pages

**Spec:** user asked to (a) ingest all invoices for cable PSID `003777`
(Charter/Spectrum Los Angeles — same operator as already-seeded `020452`)
and (b) surface cable systems everywhere on the site with their retail
brand (Spectrum, Comcast, Cox, etc.) instead of the raw FCC PSID number.

**Display format.** User chose Option B: `"<Brand> (<DMA>)"`. Example:
PSID `003777` now renders as `Spectrum (Los Angeles)`. Multiple PSIDs in
the same DMA (all four LA Spectrum PSIDs: 020452 / 014104 / 008079 /
003777) intentionally collapse to the same label — the user accepted this
tradeoff. Comcast stays on its legal name (not Xfinity) per user choice.

**Backend (ratewindow-api).**
- `seed-cable-systems.js` — added PSID `003777` (Charter LA) and a
  `display_brand` field on every row (`Charter Communications` →
  `Spectrum`, `Comcast` → `Comcast`, `Cox Communications` → `Cox`). Both
  the INSERT and `ON CONFLICT DO UPDATE` branches now propagate
  `display_brand` so re-running is idempotent.
- `migrate-cable-display.js` (NEW) — adds `cable_systems.display_brand`
  column (idempotent via `ADD COLUMN IF NOT EXISTS`), backfills it via
  `CASE operator_name ILIKE ...`, and creates a `station_display_name(p_callsign)`
  SQL function (`LANGUAGE sql STABLE`) used by every endpoint that
  returns a station/callsign column. The function accepts *any* callsign:
  cable PSIDs resolve to `display_brand || ' (' || INITCAP(LOWER(dma)) || ')'`
  while broadcast callsigns (KTLA, KXAS, WPVI) fall through `COALESCE` and
  render as-is. One helper, one source of truth, no per-endpoint JOIN
  boilerplate.
- `index.js` — 10+ endpoints now SELECT `station_display_name(...) AS
  station_display` alongside their existing callsign/station/psid column:
  `/api/rates`, `/api/benchmarks`, `/api/lur`, `/api/invoices` (cable rows
  use `COALESCE(station_call_sign, psid)` since cable rows have NULL in
  the former), `/api/lines`, `/api/cable-systems` (also exposes
  `display_brand`), `/api/lur-options`, `/api/lur-violations`,
  `/api/lur-monitor`, `/api/lur-campaign`, `/api/lur-slots`,
  `/api/entities/:id` (nested `invoices` array), and
  `/api/candidates/tracker/:id/filings`.
- Backfill: ran `backfill.js --source cable --psid 003777` on Railway —
  24 files discovered across 4 folders (Villaraigosa Governor / Becerra
  Governor / Steyer Governor / Cornachio Court of Appeals Judge),
  18 parsed successfully, 6 failed on transient FCC 503s (the documented
  ~3% failure rate). Re-run `reset-failed.js` then `backfill.js --source
  cable --psid 003777` to retry the tail.

**Frontend (this repo).** All five HTML pages that render a station
column now use the same resolution pattern:

1. **`stationLabel(row)`** — returns `row.station_display` if the backend
   provided it, else falls back to the PSID → label map (`STATION_DISPLAY`)
   built from `/api/cable-systems`, else returns the raw code. Broadcast
   callsigns pass through unchanged.
2. **`stationLabelFromCode(code)`** — same lookup without a row, used for
   dropdown option labels where we only have the code.
3. **`STATION_DISPLAY` map** — an IIFE fetches `/api/cable-systems?limit=500`
   at page load and populates the map with `psid → station_display`. If
   the deployed API doesn't yet return `station_display`, the loader
   builds it client-side from `display_brand + dma` (title-cased) — so
   the rollout works even during the moment the GitHub-Pages push is
   already live but the Railway redeploy hasn't finished.

Files touched (each in a matching place — `var ALL_DATA=[]` site for the
helpers; detail-row / renderRaw / renderMatchups / dropdown population
for the call sites):

- `rates.html` — `stationLabel(r)` in the line-item `<td class="c-amber">`
  cell, the `renderMatchups` slot header and row cell, the search filter,
  and `buildPivot()` when the pivot key is `station`. Dropdown `pop()`
  signature extended with an optional `fmt` callback; `f-station` uses
  `stationLabelFromCode` so PSIDs display as their brand label while the
  option value stays the raw code (filter compare at `getFiltered()`
  continues to match `r.station`).
- `public-file.html` — `stationLabel(r)` in `detailRow()` (which powers
  both the grouped detail tables and the raw view), the `f-search`
  text match, and the `station` view's `buildGroups()` label so each
  station-grouped row shows the friendly name. The station dropdown now
  derives its option text from `stationLabel` while keeping raw callsigns
  as option values so the existing filter compare still matches. The
  field function in `updateOptionStates` intentionally still returns raw
  `r.callsign` — otherwise the option value would diverge from the filter
  compare.
- `lur.html` — `stationLabel` in `renderMatchups` (slot header carries
  `station_display` forward into the `slots` map), the row cell, the
  invoice-review modal top bar (`inv-topbar-left`), and both dropdown
  `pop()` sites (the one in `buildDropdowns()` that runs against
  `ALL_DATA`, plus the one in `loadFilters()` that runs against
  `/api/lines/filters`).
- `explorer.html` — `stationLabel` in the grouped detail row, the raw
  view row, the search filter, and the `station` view's `buildGroups()`
  group label.
- `candidate-tracker.html` — the filings list now reads
  `f.station_display || f.station_call_sign || f.psid || '—'` so cable
  filings (where `station_call_sign IS NULL`) show the brand label while
  broadcast filings continue to show the callsign.
- `index.html` — **no edits needed**. The main SPA hits
  `/api/candidates/tracker` for state detail panels, which returns
  entity-spend rollups (not station rows), so no station string ever
  reaches the rendering path.

**Deploy order.** The migration + seed were run on Railway first (so
`display_brand` exists before any endpoint selects it), then the
`index.js` edits can be pushed at any time. The frontend loader's
client-side fallback means `politicalwindow.com` pushes are also safe to
deploy independently — there is no ordering constraint between the two
repos once the DB migration is in place.

**Verification.** `migrate-cable-display.js` ends with a spot-check
SELECT that prints every cable system's resolved display name plus a
broadcast-passthrough sanity check (`KTLA → KTLA`, `KXAS → KXAS`,
`WPVI → WPVI`). Output confirmed on Railway:
- `003777 → Spectrum (Los Angeles)`
- `015288 → Comcast (San Francisco-Oak-San Jose)`
- `020168 → Cox (San Diego)`
- `016356 → Spectrum (Dallas-Ft. Worth)`
- `016680 → Comcast (Houston)`
- `013707 → Spectrum (Austin)`
- broadcast passthrough: all three callsigns returned unchanged.

---

## 2026-04-15 — Window Timing: upcoming swapped from violet to teal

**Spec:** user followed up on the Follow-up #2 palette fix — the refined
violet (`#6D28D9`) assigned to `upcoming` states on the Window Timing overlay
was still too purple. Replace it with a non-purple hue.

**Change.** `--status-upcoming` is now `#0F766E` (teal-700). This cascades
automatically through every status-timing surface because the earlier
Follow-up #2 fix already decoupled the `--status-*` palette from the
generic `--red`/`--amber`/`--blue`/`--green` vars:

- `:root --status-upcoming` (line ~47): `#6D28D9` → `#0F766E`
- `.tile-upcoming` tint (line ~333): `rgba(109,40,217,0.10)` →
  `rgba(15,118,110,0.10)` — border + color still use the var
- `.spill-status-upcoming` (line ~539): `rgba(109,40,217,0.08)` +
  `rgba(109,40,217,0.22)` border → `rgba(15,118,110,0.08)` +
  `rgba(15,118,110,0.22)`
- `STATUS_COLORS.upcoming` JS (line ~1327): `#6D28D9` → `#0F766E`

No change needed for `SL`, `WC`, `SPILL`, map tooltip `statusLabel`, or the
insights `statusDot`, because each of those already referenced
`var(--status-upcoming)` or the JS `STATUS_COLORS` map instead of a
hardcoded hex. Same for `.dot-purple` (legend dot) — it reads from the CSS
var so it recolors automatically with this single edit.

**Not changed.**
- `.ballot-measure-title` color (line ~560, `#6D28D9`) stays violet — it's
  a ballot-measure accent, not a window-timing status.
- `.spill-purple` (line ~530, `#7C3AED`) and `.tag-governor` /
  `.slicer-governor` stay on the governor violet — "Governor" is a race
  type, not a window-timing status, and the existing CLAUDE.md convention
  keeps them on their own hue.
- Generic `--red` / `--amber` / `--blue` / `--green` vars are untouched and
  still back buttons, KPIs, and the ✓ COMPLETE chip.

**Why teal.** The existing Window Timing palette is red (`--status-open`),
amber (`--status-soon`), navy (`--status-general`), slate (`--status-none`).
Teal fits between the warm (red/amber) and cool (navy/slate) halves without
colliding with the governor violet, reads as professional on a dark
basemap, and keeps clear separation from the other four phases at a glance.

---

## 2026-04-15 — Follow-up #2 fixes (map click, overlay-matched glow, palette)

**Spec:** user feedback on Follow-up #2 implementation — three issues:
1. Clicking a congressional district bounced back to the national view
   instead of drilling in.
2. Glow color was hardcoded green; should inherit the state's color on the
   currently active map overlay (Window Timing / Density / Spend / Cash).
3. The Window Timing palette read like a "box of crayons" — needed to feel
   professional while still differentiating each phase.

### District click no longer bounces out

1. **Cause.** MapLibre dispatches click events to each subscribed layer
   independently, so the `states-fill` click handler (registered in
   `wireMapInteractions`) kept firing even when the user clicked a district.
   `e.originalEvent.cancelBubble = true` inside the district handler had no
   effect on MapLibre's own dispatch. Since the state was already selected,
   the state-level click toggled it off and `mapDeselectState` bounced the
   view back to the national map.
2. **Fix.** `states-fill` click handler now calls
   `glMap.queryRenderedFeatures(e.point, { layers: ['districts-active-fill'] })`
   first. If any active-race district was hit at the same point, it returns
   early and lets the district-layer handler handle the drill-in alone.
   One extra rendered-feature query per state click — negligible cost.

### Glow color follows active overlay

3. **`getDistrictGlowColor(abbr)` (new helper).** Picks the glow color from
   the current overlay context: if `_lastHeatmapColors[abbr]` is populated
   (Density / Spend / Cash overlays apply per-state colors here via
   `applyHeatmapToMap`), use it; otherwise fall back to
   `STATUS_COLORS[SM[abbr].status]` (Window Timing overlay); final fallback
   `#475569` (slate-600).
4. **`highlightActiveDistricts` — paint updates in place.** Previously the
   function only called `setFilter` on subsequent invocations (once the
   layers existed). Now it also calls `setPaintProperty('districts-active-
   fill', 'fill-color', glowColor)` and the equivalent `line-color` on
   `districts-active-glow-outer`. Without this, switching overlays while a
   state was zoomed-in left the glow painted in the old color.
5. **`setMapLayer` triggers a repaint.** After the state fill-color match
   expression (or the heatmap) is applied, `setMapLayer` now calls
   `applyCandMapHighlight(_currentStateAbbr)` to recolor the glow layers.
   The call no-ops when no state is zoomed in or when the selected state
   has no House races, so it's safe to invoke unconditionally.

### Refined Window Timing palette

6. **New `--status-*` CSS variables in `:root`.** Adds a dedicated,
   status-coupled color family separate from the generic
   `--red`/`--amber`/`--blue`/`--green`/`--slate` (which other UI like
   buttons, KPIs, ✓ COMPLETE spills still uses):
   - `--status-open:     #991B1B` (was `#DC2626` — red-700)
   - `--status-soon:     #B45309` (was `#D97706` — amber-700)
   - `--status-upcoming: #6D28D9` (was `#7C3AED` — violet-700)
   - `--status-general:  #1E3A8A` (was `#60A5FA` — deep navy)
   - `--status-none:     #94A3B8` (unchanged — slate)
7. **`STATUS_COLORS` JS object** updated to the same hex values. This
   cascades through the MapLibre `fill-color` match expressions in
   `addStatesLayer`, the alternate-layer match in `setMapLayer`, the
   `applyHeatmapToMap` fallback (when window-timing is active),
   `updateInsetColors`, and anywhere else that reads `STATUS_COLORS[status]`.
8. **`SL` (status-line label) + `WC` (window-color accent) + `SPILL`
   (table status cell) + insights `statusDot` + map-tooltip `statusLabel`**
   all switched from hardcoded hex or generic `var(--red)` etc. to
   `var(--status-open)` / `var(--status-soon)` / `var(--status-upcoming)` /
   `var(--status-general)` / `var(--status-none)`.
9. **`.tile-window-open` / `.tile-window-soon` / `.tile-upcoming` /
   `.tile-general-only`** now use the `--status-*` variables for both
   border + text, and 10% rgba tints in their background.
10. **New `.spill-status-open` / `.spill-status-soon` /
    `.spill-status-upcoming` / `.spill-status-general` / `.spill-status-none`
    classes.** Added alongside (not replacing) the existing `.spill-red` /
    `.spill-amber` / `.spill-purple` / `.spill-green` / `.spill-slate` so
    the COMPLETE indicator at `../index.html:3219` and `:3257` (which reuses
    `.spill-green` for a "primary completed" chip) keeps its green. The
    `SPILL` map now points at the new `.spill-status-*` classes.
11. **Legend dots.** `.dot-red`/`.dot-amber`/`.dot-purple`/`.dot-green`/
    `.dot-slate` now pull from `--status-*`. `.dot-blue` untouched — it's
    a generic accent used outside the status legend.
12. **Governor pill / slicer left alone.** `.tag-governor`,
    `.cand-slicer.slicer-governor`, and `.spill-purple` still use
    `#7C3AED`. Governor is a *race-type* identifier, not a window-timing
    status. Decoupling it here lets the status palette evolve
    independently without dragging the race-type colors along.

### Files touched

- `politicalwindow.com/index.html` — click handler, glow helper, palette,
  paint-property updates, CSS vars + classes
- `politicalwindow.com/CHANGES.md` — this entry
- `politicalwindow.com/CLAUDE.md` — Current State updated for palette +
  glow-color behavior

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
