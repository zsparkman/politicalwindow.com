# Theme tokens

Edit any value in this list (in every page's `:root` block — the file is
duplicated across `admin.html`, `candidate-tracker.html`, `explorer.html`,
`index.html`, `lur.html`, `public-file.html`, `rates.html`) to retheme
the site. Nothing else needs to change.

## Existing tokens (already in the codebase)

| Token | Default | What it controls |
|---|---|---|
| `--navy` | `#E2E6ED` | Page background fill |
| `--navy-mid` | `#F8F9FB` / `#FFFFFF` | Mid-tone alternative bg |
| `--navy-card` | `#FFFFFF` | Card body background |
| `--navy-border` | `#C1CBDB` | Card border |
| `--blue` | `#1D4ED8` | Primary blue (KPIs, active sidebar item, primary buttons) |
| `--blue-light` | `#2563EB` | Hover/lighter blue |
| `--blue-glow` | `rgba(29,78,216,0.10)` | Blue tint background |
| `--amber` | `#D97706` | Amber accent |
| `--red` | `#DC2626` | Red accent / danger |
| `--green` | `#059669` | Green accent / success / spend |
| `--slate` | `#94A3B8` | Slate text/icon |
| `--dem` | `#2563EB` (varies) | Democrat party color |
| `--rep` | `#EF4444` / `#DC2626` | Republican party color |
| `--ind` | `#8B5CF6` | Independent party color |
| `--text` | `#0F172A` | Primary body text |
| `--text-muted` | `#475569` | Secondary text |
| `--text-faint` | `#A0AEBF` | Faint/caption text |
| `--mono`, `--sans`, `--display` | font families | Typography |
| `--shadow-sm/md/lg` | drop shadows | Elevation |

## New themable tokens (added on top of the baseline)

| Token | Default | What it controls |
|---|---|---|
| `--surface` | `#FFFFFF` | Card surface (alias of --navy-card; future semantic name) |
| `--surface-alt` | `#F8FAFC` | Subtle alternate surface (table stripes, inputs, panels) |
| `--surface-faint` | `#F1F5F9` | Faint surface tint |
| `--surface-table` | `#FAFBFC` | Table header background |
| `--border` | `#E2E8F0` | Default 1px border |
| `--border-strong` | `#CBD5E1` | Stronger border / scrollbar thumb |
| `--border-faint` | `#E3E8EF` | Very faint border lines (chart grids, table dividers) |
| `--text-mute2` | `#64748B` | Mid-tone muted text (between --text-muted and --slate) |
| `--header-bg` | `#1E293B` | Top header bar background |
| `--header-border` | `#334155` | Top header bar bottom border |
| `--header-fg` | `#F1F5F9` | Top header bar foreground (logo text, page title) |
| `--header-fg-muted` | `rgba(255,255,255,0.7)` | Muted text in the top header |
| `--brand-grad-start` | `#FF642D` | Sidebar logo gradient — start |
| `--brand-grad-end` | `#FF8F6B` | Sidebar logo gradient — end |
| `--link` | `#2B6CB0` | Inline link / table cell link / chart bar primary |
| `--blue-soft` | `#93C5FD` | Pale blue (badge backgrounds) |

## How to retheme

Pick the token, change its hex value in every page file's `:root`. Example —
warm the surfaces:

```css
--surface:       #FFF8EE;
--surface-alt:   #F6EFE3;
--surface-faint: #EFE5D2;
```

Or swap the header to navy gold:

```css
--header-bg:    #0B1B36;
--header-border:#142A52;
--header-fg:    #F8E5BC;
```

There is no build step — save the file and reload.
