# politicalwindow.com

Static multi-page frontend for Political Window. Hosted on GitHub Pages
at `politicalwindow.com`. No build step — files are served verbatim.

## Pages

| Path | File | Purpose |
|---|---|---|
| `/` | `index.html` | Map / state-detail home page |
| `/candidate-tracker` | `candidate-tracker.html` | Candidate spend explorer |
| `/rates` | `rates.html` | Rate intel browser (pro tier) |
| `/public-file` | `public-file.html` | FCC public file viewer |
| `/lur` | `lur.html` | Lowest-Unit-Rate violations + invoice review |
| `/explorer` | `explorer.html` | Pivot-style line-item explorer |
| `/admin` | `admin.html` | Admin console (invites, users, API keys) |
| `/w/<slug>` | `w/<slug>.html` | **Workspace namespace** — see below |

## Workspaces (`/w/<slug>`)

`/w/<slug>` is the namespace for **Workspaces** — scoped data viewports
parameterized by persona. A workspace renders the same underlying
invoice / line / LUR data through a different lens depending on who
the workspace is for.

| Workspace type | Scope key | Persona | Status |
|---|---|---|---|
| Ownership group | set of `station_call_sign` | Group exec, sales VP | **Live** — Coastal TV at `/w/coastal` |
| Agency | set of `agency_name` | Agency principal | v2 — planned |
| Campaign | set of `political_entity_id` | Campaign manager / treasurer | v2 — planned |
| Custom | arbitrary saved filter | Analyst, consultant | v2 — planned |

### Onboarding a new workspace (current model)

1. Copy `w/coastal.html` → `w/<new-slug>.html`.
2. Edit the `GROUP` constant at the top of the new file:
   - `slug`, `display_name`, `short_name`, `cycle`, `peer_share_pct`
   - `stations[]` — each entry needs `callsign`, `dma_label`,
     `dma_db` (UPPERCASE — must match
     `political_invoices.station_market` in the ratewindow-api DB),
     `network`, `state`
3. Push to `main`. GH Pages serves the new workspace at
   `politicalwindow.com/w/<new-slug>` immediately.
4. Issue an invite token via the admin console (`/admin`). The
   workspace inherits the existing `ratewindow-api` JWT auth flow.

The `GROUP` constant is hardcoded per file today. When a second
workspace type is built (agency / campaign / custom), it becomes a
typed `WORKSPACE` config (`type: 'ownership_group' | 'agency' |
'campaign' | 'custom'` + scope-discriminated fields). Workspaces as
DB-backed first-class entities (with CRUD endpoints + per-workspace
ACLs) is a v3 concern.

### URL routing inside a workspace

All routing is query-param based — `location.pathname` stays
`/w/<slug>`:

- `?view=hero` (default) | `markets` | `market` | `invoices` |
  `compliance` | `competition`
- `&dma=<DMA_DB>` for market drill-in (UPPERCASE DMA name)
- `&demo=true` anonymizes competitor callsigns to "Competitor A/B/C"

## Architecture

- This repo: [politicalwindow.architecture.md](politicalwindow.architecture.md)
- System: [../political-window.architecture.md](../political-window.architecture.md)
- Operating guide: [CLAUDE.md](CLAUDE.md) (recent changes; what to
  modify and how)
- Changelog: [CHANGES.md](CHANGES.md) (chronological)
- Design tokens: [TOKENS.md](TOKENS.md)

## Conventions

- All HTML pages load Microsoft Clarity (`wnbzwo190n`); root pages
  also load GA4 (`G-JGHV19ZYEQ`). Internal/auth-gated pages
  (`admin`, `lur`, `w/*`) skip GA4 by design.
- Auth: pro/admin endpoints on `ratewindow-api` use a JWT stored in
  `localStorage.pw_token`. Login overlay is embedded in
  `index.html`, `lur.html`, and every `w/<slug>.html`.
- Station display convention: every page that renders a station
  column uses `stationLabel(row)` / `stationLabelFromCode(code)` —
  see CLAUDE.md "Station display convention".
