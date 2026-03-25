# Artificial Reefs of the World — Dashboard

An open, interactive world map documenting artificial reef sites across ocean regions and six continents. Built by the Octopolis team as a public resource for researchers, divers, conservationists, artists, and anyone drawn to what's happening beneath the surface.

See [ABOUT.md](ABOUT.md) for the public-facing description, current reef count, and last-updated date.

## Why This Exists

Over 30,000 artificial reef sites exist worldwide, yet no centralized global registry has ever been created — the knowledge is scattered across journals, government reports, and isolated project pages. This dashboard is a first step toward making that global picture visible, as part of [Octopolis](https://www.gofundme.com/f/octopolis-come-to-life), a large-scale participatory art installation that will be permanently submerged as a living coral reef.

## Architecture

Single-file SPA (`index.html`) with no build step. All HTML, CSS, and JavaScript live in one file. Data is loaded at runtime from a co-located CSV via `fetch()` + PapaParse. No server-side logic — entirely static.

**Data flow:** `fetch(CSV)` → PapaParse → `REEFS[]` array → Leaflet CircleMarkers + MarkerCluster → DOM

## Tech Stack

| Concern | Library | Version |
|---|---|---|
| Map rendering | [Leaflet](https://leafletjs.com/) | 1.9.4 |
| Marker clustering | [Leaflet.markercluster](https://github.com/Leaflet/Leaflet.markercluster) | 1.5.3 |
| CSV parsing | [PapaParse](https://www.papaparse.com/) | 5.4.1 |
| Tile layer | [CARTO Dark](https://carto.com/basemaps/) (dark_all) | — |
| Typography | Google Fonts — DM Serif Display, DM Sans | — |

All dependencies loaded from CDN (unpkg / Google Fonts). Zero local node_modules.

## UI Components

The layout uses CSS Grid with three rows and two columns:

```
grid-template-rows:    auto  auto  1fr
grid-template-columns: 260px 1fr
                       ↑ sidebar  ↑ map
```

**Header** (`grid-column: 1/-1`) — Logo, tagline ("Big Art for the Living Ocean"), and CTA link to GoFundMe. Full-width gradient bar with a subtle radial glow.

**Sub-header** (`grid-column: 1/-1`) — Displays `"x of n reefs selected"` as a live count synced with filter/search state.

**Sidebar** — Search input, Select All / Clear All controls, and a scrollable list of 15 category buttons. Each button shows a color swatch and per-category count. Footer with project attribution and version.

**Map** — Dark-themed Leaflet map. Markers are `L.circleMarker` with per-category fill color, grouped via `L.markerClusterGroup`. Clicking a marker opens a styled popup card showing name, organization, location, region, category badge, image (if available), description, size, design, and an outbound link.

**Responsive** — At `≤860px`, sidebar converts to a fixed-position drawer toggled by a hamburger button, with a backdrop overlay. Popup dimensions and font sizes scale down.

## State Management

**Category selection** — `activeCats: Set<string>`. Starts empty on init (ghost state). Toggled by category button clicks, Select All, and Clear All.

**Ghost / inactive state** — A separate `L.layerGroup` of gray (`#4b5563`) circle markers with `interactive: false`. Shown when `activeCats` is empty (count === 0); removed from map when any category is active. Mutually exclusive with the cluster layer — they never coexist visually.

**Search** — `searchTerm: string`. Filters across reef name, location, organization, and region. On first keystroke, a snapshot (`catsBeforeSearch: Set`) captures the current category state. Search then auto-activates only categories containing matching reefs while locking the category list (`pointer-events: none`, grayscale on non-matching swatches). On clear, the snapshot is restored.

**`syncCatButtons()`** — Single source of truth for button visual state: toggles `.active` class per button and `.search-active` class on the category list container.

## Data — CSV Schema

The dashboard reads `artificial_reefs_worldwide.csv` with the following columns:

| Column | Type | Description |
|---|---|---|
| `Name` | string | Reef project name |
| `Organization` | string | Managing entity |
| `Location` | string | Human-readable location |
| `Latitude` | float | Decimal degrees |
| `Longitude` | float | Decimal degrees |
| `Size` | string | Scale / area description |
| `Design` | string | Construction approach |
| `PrimaryCategory` | string | One of 15 categories (must match a key in `CAT_COLORS`) |
| `Description` | string | Project summary |
| `URL` | string | Outbound link |
| `ImageURL` | string | Optional thumbnail |
| `Region` | string | `"Continent / Water Body"` (e.g., `"Asia / Bali Sea"`) |

## Deployment

Hosted on **Cloudflare Pages** as a static site. No build command — the repo root is the publish directory.

Requirements for any static host:

- `index.html` and `artificial_reefs_worldwide.csv` must be in the same directory.
- Must be served over HTTP(S) — opening `index.html` from `file://` will fail the CSV `fetch()` due to CORS.

## Local Development

```bash
# Any local HTTP server works. Examples:
python3 -m http.server 8000
# or
npx serve .
```

Then open `http://localhost:8000`.
