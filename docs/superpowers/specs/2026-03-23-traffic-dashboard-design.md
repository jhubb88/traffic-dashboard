# Texas Traffic Dashboard — Design Spec
**Date:** 2026-03-23
**Status:** Approved

---

## Overview

A single self-contained `index.html` file that displays Texas traffic incident data on an interactive map. Designed as a portfolio project for jimmyhubbard2.cc, styled to match the existing dark portfolio aesthetic. Hosted on AWS S3.

The app attempts two live public APIs before falling back to realistic simulated data. Due to browser CORS restrictions on S3-hosted static files, **Demo Mode (simulated data) is the expected default runtime state.** The live API attempts are included to demonstrate the integration pattern and occasionally succeed in environments with permissive CORS.

---

## Layout

- **Full-viewport layout:** top bar + body (sidebar + map) + footer
- **Left sidebar (280px fixed):** incident counts, filter pills, scrollable incident list
- **Map area (flex: 1):** Leaflet.js map fills remaining space
- **Responsive breakpoint at 768px:** sidebar collapses below the map in a stacked layout

---

## Internal Incident Data Schema

All data sources — live or simulated — normalize to this internal structure before rendering:

```js
{
  id: String,           // unique incident identifier (stable across refreshes for simulated data)
  type: String,         // "accident" | "construction" | "hazard" | "closure"
  title: String,        // human-readable incident name
  location: String,     // road/highway description (e.g., "IH-35 N @ Exit 240, Austin")
  severity: String,     // "minor" | "moderate" | "major"
  lat: Number,          // latitude
  lng: Number,          // longitude
  details: String,      // extra info (lanes blocked, duration, etc.) — may be empty string
  updatedAt: String     // ISO timestamp or human-readable time string
}
```

Any incident missing `lat` or `lng` is dropped silently (not rendered on map, not shown in list).

---

## Top Bar

- App title: "Texas Traffic Dashboard" with traffic light emoji
- **Source indicator badge:** shows active data source with color-coded status
  - Green: "LIVE — TxDOT 511"
  - Green: "LIVE — SF Bay 511" (note: data is Bay Area, badge clarifies this)
  - Yellow: "DEMO MODE"
- **Countdown timer:** "Refreshing in Xs" counts down to next auto-refresh
- **Manual refresh button:** triggers immediate data reload and resets countdown

---

## Left Sidebar

### Summary Counts
4 color-coded stat cards (2×2 grid):
- 🔴 Accidents (`#ef4444`)
- 🟠 Construction (`#f97316`)
- 🟡 Hazards (`#eab308`)
- 🔵 Closures (`#3b82f6`)
- Total incidents count below the grid
- Counts reflect currently active filters (not total from source)

### Filter Pills
Toggle buttons for each incident type + "All" reset. Toggling a filter:
- Updates map (hidden-type markers are removed from map entirely, not greyed out)
- Updates incident list (hidden-type cards are removed from list)
- Updates count display

### Incident List
Scrollable cards sorted by severity (major → moderate → minor). Each card shows:
- Color dot + incident title
- Location string
- Severity + details line

**Clicking a card:** pans/zooms the map to that marker, opens the popup, highlights the card with a colored border.

**Active filter interaction:** cards for filtered-out types are not rendered in the list. No ambiguity.

---

## Map

- **Library:** Leaflet.js (CDN, no API key)
- **Tiles:** OpenStreetMap (free, no key)
- **Default view:** Texas (center ~30.2672, -97.7431 / Austin, zoom 7)
- **Markers:** Circular, color-coded, white border + glow ring matching type color
- **On marker click:** small popup appears on map (title, location, severity) AND sidebar scrolls to and highlights that card
- **On refresh:** map viewport (center + zoom) is preserved. Active popup is closed. New markers replace old markers.
- **Zero incidents:** map shows empty (no markers), sidebar shows "No incidents to display" message in the list area, counts show all zeros

---

## Data Sources (cascade A → B → C)

Each source is attempted with a 3-second timeout. First success wins. On failure the app tries the next source silently.

### A. TxDOT 511
- Attempts known public endpoints for Texas 511 data (tried in order, first CORS-success wins)
- No API key required on public endpoints
- Fields mapped to internal schema via a `normalizeTxDOT(raw)` function

### B. SF Bay Area 511
- Public API, free key hardcoded in the file (acceptable for a client-side portfolio project)
- Returns Bay Area incidents; source badge notes "SF Bay 511" so the geographic mismatch is visible
- Fields mapped via `normalizeSF511(raw)` function

### C. Simulated Data (guaranteed to succeed)
- ~25 incidents across Texas highways: IH-35, IH-10, US-290, TX-1, Beltway 8, Loop 360
- Incidents have **stable IDs** across refreshes — a fixed pool of ~40 incidents, with ~25 randomly active at any time. On each refresh, a few are added/removed to simulate live feed activity.
- This avoids every marker being "new" on every cycle, keeping animations meaningful
- Malformed response and network offline states are impossible in this path

### Error Handling
- **Malformed JSON from live API:** caught with try/catch, treated as a source failure → next source
- **Network offline:** `fetch` rejects → caught → treated as source failure → next source (Demo Mode always available offline)
- **All live sources fail:** Demo Mode loads automatically, no user-visible error needed
- **Refresh during open popup:** popup is closed before new markers are placed

---

## Auto-Refresh

- Interval: **45 seconds**
- Countdown visible in top bar
- On refresh: re-run source cascade → normalize data → update map markers → update sidebar → reset countdown
- Map viewport preserved across refreshes
- New/added incidents fade in; removed incidents fade out

---

## Visual Design

- Background: `#0d1117`
- Surface: `#161b22`
- Border: `#30363d`
- Body font: Inter (Google Fonts CDN)
- Muted text: `#64748b` / `#94a3b8`
- Primary text: `#e2e8f0`
- Incident colors: red `#ef4444`, orange `#f97316`, yellow `#eab308`, blue `#3b82f6`

---

## Footer

- Attribution: "Data: TxDOT 511 Public API · Map: Leaflet.js + OpenStreetMap"
- "← Back to Projects" link → `https://jimmyhubbard2.cc/projects/`

---

## Delivery

- Single file: `index.html`
- All CSS in `<style>` block, all JS in `<script>` block
- External CDN only: Leaflet.js, Inter font
- S3 bucket: `jimmy-traffic-dashboard` (us-east-1)
- Added to Projects page on jimmyhubbard2.cc as card #6 (replaces Coming Soon slot)
