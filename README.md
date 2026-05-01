# Live Traffic Dashboard

Multi-city traffic incident map with a simulated data feed and live-API fallback cascade.

**Live demo:** https://d1t5py05a4uugi.cloudfront.net

## Overview

The Live Traffic Dashboard displays active traffic incidents across five US cities on an interactive Leaflet map. It attempts to pull real incident data from TxDOT 511 and SF511, falling back to a stable simulated dataset when those APIs are unavailable (CORS restrictions block direct calls from a static host without a proxy). Incidents are color-coded by severity, filterable by type, and linked between the map and a scrollable sidebar.

## Features

- **Interactive map** — Leaflet.js + OpenStreetMap; custom SVG markers color-coded by incident type and severity
- **Five cities** — Scottsdale AZ, Denver CO, Salt Lake City UT, Nashville TN, Dallas/Fort Worth TX
- **50-incident pool** — 10 per city; 25 active at any time; pool rotates 2–4 incidents on each refresh
- **Data cascade** — attempts TxDOT 511 → SF511 → Demo Mode (simulated); source displayed in a badge
- **Filter pills** — exclusive selection per incident type (accident, closure, hazard, construction); count updates live
- **Linked map/sidebar** — click a map marker to filter the sidebar to that city and zoom in; click a sidebar card to zoom to the exact incident; Reset View restores continental US view
- **Auto-refresh** — every 5 minutes; manual refresh button available
- **Mobile-responsive** — stacked layout at ≤768px

## Tech stack

| Layer | Technology |
|---|---|
| Markup / style / logic | Vanilla HTML5, CSS3, JavaScript |
| Map | Leaflet.js 1.9.4 + OpenStreetMap |
| Fonts | Google Fonts (Inter) |
| Hosting | AWS S3 + CloudFront |

No npm, no bundler, no framework.

## Architecture

The entire application is a single `index.html` file. On load, `fetchIncidents()` tries TxDOT 511, then SF511, then falls back to the local `SimulatedData` module. The active incident pool is seeded once and rotated each refresh to simulate real-world churn. `MapManager` owns the Leaflet instance and marker layer; `SidebarManager` owns the incident list; both read from the same shared state. A `setInterval` countdown drives auto-refresh at 300-second intervals. A Lambda proxy to bypass CORS on the live APIs is planned but not yet deployed.

## Local development

```bash
python3 -m http.server 8080
```

Open `http://localhost:8080`. No build step, no dependencies to install. Live API calls (TxDOT, SF511) may still be blocked by CORS depending on the browser; the app falls back to Demo Mode automatically.

## Deployment

Deploys automatically on push to main via GitHub Actions (.github/workflows/deploy.yml). S3 sync + CloudFront invalidation handled by the workflow.

**S3 bucket:** `jimmy-traffic-dashboard` (us-east-1)

## Project structure

```
traffic-dashboard/
├── index.html          # Complete application — HTML, CSS, and JS in one file
└── diagrams/
    ├── traffic-dashboard-architecture.svg
    └── traffic-dashboard-prod-architecture.svg
```

## Roadmap

- Lambda proxy to enable live TxDOT / SF511 data on CloudFront without CORS errors
- Additional cities and live data sources

## License

MIT — see [LICENSE](LICENSE)

## Author

Jimmy Hubbard — [github.com/jhubb88](https://github.com/jhubb88)

---

*Part of [jhubb88's portfolio](https://jimmyhubbard2.cc)*
