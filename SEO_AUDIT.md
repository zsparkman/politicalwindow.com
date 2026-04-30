# SEO Audit — politicalwindow.com

_Generated 2026-04-25 by Claude Code (Opus). Repo HEAD: `main` (working
tree clean except untracked `architecture.md` overlap)._
_Live site verified at `https://politicalwindow.com` on the same date —
HTTP 200, content-length 221434 bytes, matches local `index.html`
exactly. No drift between repo and prod._

This audit is read-only. No code has been changed. Awaiting approval
before P0 / P1 implementation per the SEO_Optimization_Prompt.md
"Pause for review between steps 2/3" rule.

---

## TL;DR

The product is content-rich, but **search engines and AI retrievers
currently see almost nothing**. Every HTML page is missing the
fundamentals: no meta description, no canonical, no Open Graph, no
Twitter Card, no schema.org JSON-LD, no `<h1>` / `<h2>` / `<h3>`
elements, no `robots.txt`, no `sitemap.xml`. Page titles exist but
are short and lead with the brand instead of the keyword. The seven
HTML pages each leak the same gaps, so a single shared-snippet edit
pattern fixes most of them in parallel.

The biggest single ranking lever is **JSON-LD structured data for
candidates, ballot measures, and FCC primary windows** — that data is
live in the API and is the unique value proposition, but it is
JS-rendered into anonymous `<div>`s today, so neither Google rich
results nor Perplexity / ChatGPT browsing can extract it. Embedding
the same facts as inline JSON-LD in the served HTML closes the gap
without changing the visual UI.

**Estimated effort:** P0 + P1 land in ~14 commits, all small, all
mechanical. No content rewrites except meta titles / descriptions and
one explanatory About / Methodology paragraph per page.

**Open questions for you (no fabrication):**
1. Business address / publisher legal entity name?
2. Founding date / launch date?
3. Twitter/X, LinkedIn, GitHub handles, if any?
4. Author / editor names + bios for the methodology page?
5. OG image — should I generate a 1200×630 SVG-rendered card from the
   existing favicon palette, or do you have a designed asset?
6. Should `admin.html` be `noindex,nofollow`? (My recommendation: yes.)
7. Should `?state=IL` style URLs be canonicalized to `/` or treated
   as their own indexable surfaces? (My recommendation: canonical to
   `/` for now; add real per-state landing pages as a P2 — see below.)

---

## 1. Current State (with file:line citations)

### 1.1 Crawlability and indexation

| Concern | Status | Evidence |
|---|---|---|
| `robots.txt` | **Missing** | `curl -sI https://politicalwindow.com/robots.txt` → 404; no `robots.txt` in repo root |
| `sitemap.xml` | **Missing** | Same — 404 live, file absent in repo |
| `humans.txt` | Missing (cosmetic) | Absent |
| `.well-known/` | Missing | Absent — no `ai-plugin.json`, no `security.txt` |
| HTTPS | OK | `HTTP/2 200` on apex, GitHub Pages with Fastly edge |
| Canonical tag | **Missing on every page** | `grep -niE 'rel="canonical"'` → 0 matches across all 7 HTML files |
| Internal linking | Adequate but flat | Sidebar (`index.html:916-962`, mirrored in each page) reaches every page in 1 click. No breadcrumbs. |
| 404 / redirect chains | Untested at scale | No redirects observed on apex; GitHub Pages serves a default 404. No custom 404.html exists. |

### 1.2 Page titles

Existing `<title>` tags (front-loaded with brand, not keyword):

| File | Line | Title | Length | Issue |
|---|---|---|---|---|
| `index.html` | 15 | `PoliticalWindow.com — 2026 FCC Ad Intelligence` | 47 | Brand-first; "2026 FCC Ad Intelligence" buries higher-value keywords like "FCC political file" / "primary windows" |
| `candidate-tracker.html` | 14 | `Candidate Tracker — PoliticalWindow.com` | 40 | Generic — no cycle, no race, no spend angle |
| `rates.html` | 16 | `Candidate Rate Intelligence — PoliticalWindow.com` | 50 | OK length but "Rate Intelligence" is internal jargon; users search "political ad rates" / "lowest unit rate" |
| `public-file.html` | 16 | `Public File Activity — PoliticalWindow.com` | 43 | "Activity" is vague; "FCC public file" is the searchable term |
| `lur.html` | 6 | `LUR Monitor — PoliticalWindow.com` | 33 | "LUR" is a niche acronym; ranks for nothing without expansion |
| `explorer.html` | 16 | `FCC Rate Explorer — PoliticalWindow.com` | 40 | Too vague; "rate explorer" doesn't match user intent |
| `admin.html` | 5 | `Admin — PoliticalWindow.com` | 28 | Should be `noindex` regardless |

### 1.3 Meta tags (per-page audit)

`grep -niE 'meta name="(description|keywords|robots|author)"'` → **0 matches across all 7 files.**

| Tag | Present anywhere? |
|---|---|
| `<meta name="description">` | No |
| `<meta name="keywords">` | No (good — it's deprecated; just confirming absence) |
| `<meta name="robots">` | No |
| `<meta name="author">` | No |
| `<meta charset="UTF-8">` | Yes, every page (e.g. `index.html:13`) |
| `<meta name="viewport">` | Yes, every page (e.g. `index.html:14`) |
| `<html lang="en">` | Yes, every page (e.g. `index.html:2`) |

### 1.4 Open Graph / Twitter Card

`grep -niE 'og:|twitter:'` → **0 matches across all 7 files.**

No OG title, description, image, url, type, or site_name. No Twitter
card type, site, creator, or image. Sharing the URL on Slack,
LinkedIn, Twitter/X, or iMessage produces a generic preview pulled
from `<title>` only.

### 1.5 Structured data (schema.org JSON-LD)

`grep -niE 'application/ld\+json|schema\.org'` → **0 matches across all 7 files.**

No `Organization`, no `WebSite`, no `BreadcrumbList`, no `Person` for
candidates, no `Event` for primaries / ballot measures, no `Place` /
`AdministrativeArea` for states. This is the single highest-leverage
miss for both Google rich results and AI search retrieval.

### 1.6 Headings and semantic HTML

`grep -niE '<h[1-6]'` → **0 matches across all 7 files.**

Every "heading" is a styled `<div>`. Examples:

- Homepage card title: `<div class="card-title">2026 Election
  Activity Map</div>` (`index.html:1046`)
- Candidate Tracker page title: `<div style="font-size:0.95rem;
  font-weight:700;...">Candidate Tracker</div>`
  (`candidate-tracker.html:310`)
- Rates page title: `<div style="font-size:1rem;font-weight:600;...">
  Rate Analytics</div>` (`rates.html:366`)
- LUR page title: `<div style="font-size:1rem;font-weight:600;...">
  LUR Monitor Report</div>` (`lur.html:269`)
- Public file title: `<div style="...">Filings</div>`
  (`public-file.html:313`)
- Explorer title: `<div style="...">Rate Intelligence</div>`
  (`explorer.html:225`)

Semantic landmarks _are_ present: `<header>`, `<main>`, `<nav>`,
`<footer>` exist on most pages (`index.html:967, 980, 916, 4404`;
similar on others). `lur.html`, `rates.html`, `public-file.html`,
`explorer.html` lack `<main>` (they use `<div id="main">` instead);
fix is trivial.

`<article>`, `<section>`, `<aside>` are absent entirely.

### 1.7 Image alt text

The pages render almost no `<img>` tags — content is JS-rendered SVG
icons and MapLibre canvas. SVGs are decorative and inline; they
don't need `alt`, but adding `role="img"` + `aria-label` to the
sidebar-rail logo (`index.html:919`, mirrored each page) is a small
accessibility-and-SEO win. The MapLibre canvas itself has no text
alternative — recommend a `<figcaption>` describing the map content
for the static-rendered case (P1).

### 1.8 Analytics

Google Analytics 4 (`G-JGHV19ZYEQ`) is installed on five of the
seven pages: `index.html:4`, `candidate-tracker.html:4`,
`rates.html:4`, `public-file.html:4`, `explorer.html:4`. **Missing
on `lur.html` and `admin.html`.** Recommend adding to `lur.html`
(it's a public-facing page); `admin.html` is a judgment call and
probably should stay un-instrumented.

### 1.9 Performance / Core Web Vitals (heuristic, no Lighthouse run)

- MapLibre GL JS 4.7.1 loaded synchronously in `<head>`
  (`index.html:23`). This is render-blocking. Defer with
  `defer` attribute or move to end-of-body.
- Google Fonts loaded via `<link rel="stylesheet">` without
  `display=swap` enforced inside the URL — actually, it _is_ there
  (`&display=swap`, `index.html:21`). Good.
- No `preconnect` to API origins (`api.politicalwindow.com`,
  `rates.politicalwindow.com`, `unpkg.com`,
  `basemaps.cartocdn.com`, `fonts.gstatic.com`). Adding 4×
  `<link rel="preconnect">` saves ~100-300ms on first paint with no
  visual change.
- Inline `<style>` block in every page is ~200 lines; not minified,
  but small enough that it's not a real problem. Skip.
- Images: none in document; map tiles are CDN-served and outside
  our control.
- Cache headers: `cache-control: max-age=600` from GitHub Pages /
  Fastly — fine for HTML.
- No service worker, no compression beyond GitHub Pages defaults
  (gzip/brotli applied by Fastly).

### 1.10 Mobile / responsive

CSS already breaks at 640px (per `CLAUDE.md`). Tap targets and
typography look reasonable in the existing markup. No
mobile-specific SEO blocker observed.

### 1.11 Discoverability for AI / LLM search

Specific gaps for ChatGPT browsing, Perplexity, Gemini, Claude,
Brave Search summarization:

- No structured data → retrievers can't classify entities (Person /
  Event / Place / Organization).
- All key facts (state windows, candidate spend, primary dates,
  ballot measures) are JS-rendered from the API at runtime. A
  retriever doing a static fetch sees an empty shell. **This is the
  biggest single problem for AI search.**
- No `llms.txt`, no `/.well-known/ai-plugin.json`.
- Footer attribution names "FCC OPIF PUBLIC FILES" but doesn't link
  to `https://publicfiles.fcc.gov/`. Source links help retrievers
  treat the site as a primary citation.

### 1.12 Trust / E-E-A-T signals

- No About page.
- No Methodology page (data sourcing, refresh cadence, reconciliation
  rules).
- No author bios.
- No "Last updated" timestamps surfaced in HTML (the data badge says
  `LIVE` / `CACHED SNAPSHOT` but no ISO timestamp).
- Footer: `© 2026 PoliticalWindow.com · All rights reserved · FCC
  Political Ad Intelligence` (`index.html:4404-4406`). No publisher
  legal name, no contact link other than `mailto:info@...` in the
  login overlay (`index.html:4430`).
- Data source attribution is text-only ("DATA SOURCE: FCC OPIF
  PUBLIC FILES") and not linked.

### 1.13 Industry-specific keyword opportunities (left on the table)

The site doesn't currently target these high-intent industry
queries (none appear in titles, headings, or meta — because there
are no meta or headings):

- "FCC political file" / "FCC OPIF political file"
- "Lowest Unit Rate" / "LUR compliance" / "FCC LUR violation"
- "political ad rates [station]" / "political ad rates [DMA]"
- "political broadcast windows" / "FCC 45-day political window"
- "[state] 2026 primary date" / "[state] 2026 ballot measures"
- "[candidate name] FCC ad spend" / "[candidate name] political ads"
- "candidates running for senate [state] 2026"

The data to rank for every one of these is in the API. The
content-strategy lift is small — the JSON-LD lift carries most of
the weight.

---

## 2. Ranked Recommendations

### P0 — High impact, low effort. Implement first.

| # | Recommendation | Files | Effort |
|---|---|---|---|
| P0-1 | Create `robots.txt` allowing all crawlers, declaring `sitemap.xml` location | new file | XS |
| P0-2 | Create `sitemap.xml` with 6 indexable URLs (exclude `admin.html`) and `lastmod` dates | new file | XS |
| P0-3 | Add `<link rel="canonical">` to every page pointing at the canonical URL with no query string | 7 files | S |
| P0-4 | Rewrite `<title>` on every page — keyword-front, 50-60 chars, page-specific | 7 files | S |
| P0-5 | Add `<meta name="description">` on every page — 150-160 chars, specific, expert-toned | 7 files | S |
| P0-6 | Add `<meta name="robots" content="index,follow">` on public pages and `noindex,nofollow` on `admin.html` | 7 files | XS |
| P0-7 | Add `Organization` + `WebSite` (with `SearchAction`) JSON-LD to every page | 7 files | S |
| P0-8 | Add Open Graph + Twitter Card meta tags to every page | 7 files | S |
| P0-9 | Promote the existing styled-div page titles to actual `<h1>` (one per page, keyword-bearing) and downgrade existing card titles to `<h2>` / `<h3>` | 7 files | S |
| P0-10 | Add `<link rel="preconnect">` to api.politicalwindow.com, rates.politicalwindow.com, unpkg.com, basemaps.cartocdn.com, fonts.gstatic.com | 7 files | XS |

### P1 — High impact, medium effort.

| # | Recommendation | Files | Effort |
|---|---|---|---|
| P1-1 | Generate / commit a 1200×630 OG image and a 1200×675 Twitter card image | new files | M (waits on you re: design) |
| P1-2 | Inline JSON-LD `BreadcrumbList` on every secondary page (Tracker / Rates / Public File / LUR / Explorer) | 5 files | S |
| P1-3 | Inline a static, server-renderable JSON-LD blob on `index.html` listing the 51 state primary windows (`Event` schema) — pre-baked at build time, not API-fetched | `index.html` + new build script | M |
| P1-4 | Same for declared candidates (`Person` / `Candidate` schema) — top-N or all, server-rendered | `index.html` + script | M |
| P1-5 | Same for ballot measures (`Event` / `LegislationObject`) | `index.html` + script | M |
| P1-6 | Add an "About FCC Political Windows" 100-word explainer block to `index.html` (visually unobtrusive, expert-toned) | `index.html` | S |
| P1-7 | Add a parallel "How LUR works" callout to `lur.html` and "What's in the FCC Public File" callout to `public-file.html` | 2 files | S |
| P1-8 | Add `role="img"` + `aria-label` to the sidebar-rail logo and ARIA labels to icon-only links | shared CSS / each page | S |
| P1-9 | Add an `<address>` / linked footer mentioning publisher and contact email; link "FCC OPIF" to `https://publicfiles.fcc.gov/` | 7 files | S |
| P1-10 | Add a `<figure>` + `<figcaption>` wrapper around the MapLibre map describing what the map shows | `index.html` | S |
| P1-11 | Add `defer` to MapLibre script tag (`index.html:23`) and move it to end-of-body or use `defer`. Verify no race against `initMap()` boot. | `index.html` | S |
| P1-12 | Add `<meta property="article:author">`, `<meta property="article:section">` placeholders for future blog/news (gated until content exists). Skip unless P2-1 is approved. | n/a | n/a |
| P1-13 | Add `llms.txt` at site root summarizing site purpose, key data fields, and primary-source citations for AI retrievers | new file | XS |
| P1-14 | Add Google Analytics to `lur.html` for parity with other public pages | `lur.html` | XS |
| P1-15 | Add a custom `404.html` that links back to home and the four primary surfaces, with proper meta. GitHub Pages will serve it on miss. | new file | S |

### P2 — High impact, high effort. Proposals only — awaiting approval.

See "Proposed P2 — Awaiting Approval" section below.

### P3 — Marginal or speculative. Logged, not implemented.

| # | Recommendation | Why P3 |
|---|---|---|
| P3-1 | Add `humans.txt` at site root | Cosmetic; signals to no major retriever |
| P3-2 | Add `<meta name="theme-color">` and PWA manifest | Small UX polish; not a ranking signal |
| P3-3 | Add `dns-prefetch` in addition to `preconnect` | Marginal next to preconnect; preconnect already covers it |
| P3-4 | Add Schema.org `WebPage.speakable` annotations for voice search | Speculative; voice-search SEO returns are minimal in this niche |
| P3-5 | Per-state RSS feed of FCC filing updates | Real value but a content-product decision, not a ranking lever |
| P3-6 | Add `<meta name="rating" content="general">` | Effectively dead; modern search engines ignore it |

---

## 3. Proposed P2 — Awaiting Approval

These are higher-effort content / strategy decisions. Each is a
one-paragraph proposal — **do not implement without explicit
approval.**

### P2-1 — Per-state landing pages (`/state/[abbr]/`)

**Proposal:** generate 51 static landing pages (50 states + DC) at
`/state/al/`, `/state/ak/`, etc., each pre-rendered with that state's
FCC window dates, primary date, runoff date, declared Senate /
Governor / House candidates, and ballot measures, plus an inlined
`Event` + `Person` JSON-LD blob. Pages would be generated by a small
Node script that hits `api.politicalwindow.com` and writes static
HTML at build time, committed to the repo. This captures the entire
"[state] 2026 primary date" / "[state] 2026 ballot measures" /
"[state] 2026 senate candidates" long tail, which is high-volume and
has no national-level competitor that matches the site's data depth.
Tradeoff: introduces a (lightweight) build step the project
currently doesn't have. Effort: ~1-2 days. Impact: largest single
ranking lever after P0/P1.

### P2-2 — Per-candidate landing pages (`/candidate/[fec_id]/`)

**Proposal:** static page per candidate keyed by FEC ID, surfacing
total spend, top-N stations, top DMAs, party, office, district,
incumbency, plus `Person` JSON-LD. Captures "[candidate name] ad
spend" / "[candidate name] political ads" long tail — historically
high-conversion-intent for opposition-research and journalism use
cases. Same build-step tradeoff as P2-1. Effort: ~2-3 days
(thousands of pages, but mechanical). Impact: high for
journalist / industry traffic.

### P2-3 — FAQ section on `index.html` (with `FAQPage` schema)

**Proposal:** add a 6-8 question FAQ at the bottom of the homepage
covering "What is an FCC political window?", "What is the Lowest
Unit Rate?", "How do FCC public files work?", "When is my state's
2026 primary?", "Where does this data come from?", "How often is it
updated?". Each Q&A wrapped in `FAQPage` schema, which Google
rewards with rich-result eligibility. Visually a collapsible
accordion at the bottom of the page so it doesn't disrupt the data
density above. Effort: ~3-4 hours including copy. Impact: medium
for ranking, high for AI-search citation.

### P2-4 — `/methodology/` page

**Proposal:** single page describing data sources (FCC OPIF, FEC,
Ballotpedia), refresh cadence (FEC scraper daily 6 AM ET; FCC
filings ingested as posted), normalization rules (cable PSID →
brand mapping, candidate name de-dup, party affiliation
reconciliation), and known gaps (states behind on filings, missing
incumbency data, etc.). Add `Article` schema with author byline.
Effort: ~3 hours. Impact: feeds E-E-A-T directly; required
infrastructure for any future news / blog content.

### P2-5 — `/about/` page

**Proposal:** brief about page with publisher legal name, contact,
team / methodology summary, and source citations. Required to claim
publisher identity in `Organization` schema with non-stub values.
Effort: ~1 hour. Impact: low ranking, high E-E-A-T support.

### P2-6 — Pre-rendering / static snapshot of `index.html`

**Proposal:** at build time, fetch `/api/states`, `/api/candidates`,
`/api/ballot-measures` and inject the rendered HTML for KPIs,
state status chips, and candidate counts into the served
`index.html`. Today the HTML the crawler sees has empty `<div
id="kpiCands">0</div>` placeholders that only get populated after
JS runs. Pre-rendering closes the gap for crawlers and AI
retrievers without changing behavior for end users (the JS still
runs and overwrites with live data). Tradeoff: introduces a build
step. Effort: ~half a day. Impact: large for AI search; medium for
Google (Googlebot does render JS, but with delay).

### P2-7 — News / blog section

**Proposal:** add `/news/` directory for occasional explainer posts
(election-cycle news, methodology updates, notable LUR violations).
Each post `Article` schema, `NewsArticle` if timely. Builds
backlinks, captures news-search surfaces, supports E-E-A-T. **Big
content-strategy commitment** — only pursue if there's editorial
bandwidth. Effort: ongoing. Impact: large but compounds slowly.

### P2-8 — XML feed of "today's notable filings"

**Proposal:** an `/atom.xml` or `/feed.xml` that surfaces the day's
top FCC filings or the week's largest political ad spends. Aimed
at journalists and industry pros. Effort: ~half a day to wire to
the API. Impact: niche but sticky for industry-press relationships.

---

## 4. Specific copy proposals for P0-4 / P0-5

Drafts for review — not yet committed to files. All are factual,
specific, and avoid puffery per the prompt.

### `index.html`

- **Title (54 chars):** `2026 FCC Political Ad Windows & Election Map | PoliticalWindow`
- **Description (158 chars):** `Live tracker for 2026 FCC political broadcast windows, declared Senate / Governor / House candidates, ballot measures, and primary dates across all 51 states.`

### `candidate-tracker.html`

- **Title (60 chars):** `2026 Candidate Ad Spend Tracker — FEC Filings | PoliticalWindow`
- **Description (160 chars):** `Searchable database of declared 2026 federal and state candidates with FEC-sourced ad spend totals, station and DMA breakdowns. Filter by office, party, state.`

### `rates.html`

- **Title (60 chars):** `Political Ad Rate Intelligence by Station & DMA | PoliticalWindow`
- **Description (159 chars):** `Pivot, trend, and variance views of 2026 political broadcast ad rates by station, DMA, daypart, and candidate. Sourced from FCC OPIF political file invoices.`

### `public-file.html`

- **Title (58 chars):** `FCC Political Public File Viewer — 2026 Filings | PoliticalWindow`
- **Description (160 chars):** `Browse 2026 FCC political file contracts and invoices by station, candidate, advertiser, and DMA. Sourced daily from FCC OPIF public files. No login required.`

### `lur.html`

- **Title (59 chars):** `Lowest Unit Rate (LUR) Variance Monitor — 2026 | PoliticalWindow`
- **Description (159 chars):** `Identifies FCC Lowest Unit Rate variances where two or more candidates paid different rates for the same airtime. Side-by-side invoice comparison for compliance.`

### `explorer.html`

- **Title (58 chars):** `2026 Political Ad Rate Explorer by Advertiser | PoliticalWindow`
- **Description (160 chars):** `Pivot 2026 FCC political ad invoices by advertiser, station, DMA, party, and state. Line-item access to gross totals, unit rates, dayparts, and contract dates.`

### `admin.html`

- **Title:** `Admin — PoliticalWindow.com` (unchanged — page will be `noindex`)
- **Description:** none needed; `noindex,nofollow` is sufficient.

---

## 5. Implementation order (commits)

Once approved, the work lands on branch `feat/seo-overhaul` from
`main`, one logical concern per commit. No push, no merge.

1. `chore(seo): add robots.txt and sitemap.xml`
2. `seo: add canonical link to every page`
3. `seo: rewrite per-page titles for keyword-first targeting`
4. `seo: add per-page meta descriptions`
5. `seo: add meta robots; mark admin noindex`
6. `seo: add Organization + WebSite JSON-LD to every page`
7. `seo: add Open Graph and Twitter Card meta to every page`
8. `seo: promote page titles to h1; downgrade card titles to h2/h3`
9. `seo: add preconnect hints for API and CDN origins`
10. `seo: add BreadcrumbList JSON-LD to secondary pages`
11. `seo: inline static state-window Event JSON-LD on homepage` (P1-3)
12. `seo: inline static candidate Person JSON-LD on homepage` (P1-4)
13. `seo: inline static ballot-measure Event JSON-LD on homepage` (P1-5)
14. `seo: add expert-toned context blurbs (FCC windows / LUR / public file)` (P1-6, P1-7)
15. `seo: aria labels on icon-only nav; figcaption on map`
16. `seo: footer source-link and publisher metadata`
17. `seo: defer maplibre; add llms.txt; add 404.html`
18. `seo: GA4 on lur.html for parity`
19. `docs: update architecture.md + CHANGES.md for SEO overhaul`

After each commit: open the touched page(s) via `python3 -m
http.server` and confirm visual + console parity. JSON-LD validated
by hand against the schema.org spec.

---

## 6. Out-of-scope reminders

Per the prompt:
- No Search Console / Bing submissions (need credentials).
- No backlink work.
- No `politicalwindow-api` or `ratewindow-api` changes.
- No DNS / domain changes.
- No new analytics tools beyond GA4 parity.
- No paid keyword research.

---

## 7. Sign-off needed

Reply "approved" or with edits and I'll start the commit sequence
in section 5 against `feat/seo-overhaul`. Reply with answers to the
seven open questions in the TL;DR (or "skip / use placeholders") and
I'll fold them in where they affect Organization / Person / About
schemas without inventing values.

The **eight files I will create or modify on approval (P0 + P1, no
P2):**
- `robots.txt` (new)
- `sitemap.xml` (new)
- `llms.txt` (new)
- `404.html` (new)
- `og-image.svg` or similar (new — pending answer on Q5)
- `index.html` (head + body markup edits; no JS logic touched)
- `candidate-tracker.html` (same)
- `rates.html` (same)
- `public-file.html` (same)
- `lur.html` (same)
- `explorer.html` (same)
- `admin.html` (head only — meta robots noindex)
- `architecture.md` (P0/P1 closeout note)
- `CHANGES.md` (session entry)
