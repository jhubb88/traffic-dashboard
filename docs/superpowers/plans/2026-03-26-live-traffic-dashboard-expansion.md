# Live Traffic Dashboard — Multi-City Expansion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [x]`) syntax for tracking.

**Goal:** Expand the Traffic Dashboard from Texas-only to 5 cities (Scottsdale AZ, Denver CO, Salt Lake City UT, Nashville TN, Dallas/Fort Worth TX), rename the project, and update the default map view.

**Architecture:** Single HTML file (`index.html`) — all changes are text replacements. No logic changes; only the POOL array, map center constant/zoom, title strings, and footer text are modified.

**Tech Stack:** HTML/CSS/JS, Leaflet.js, OpenStreetMap

---

### Task 1: Rename title and topbar

**Files:**
- Modify: `index.html:6` (title tag)
- Modify: `index.html:264` (topbar label)

- [x] **Step 1: Update `<title>` tag**

In `index.html` line 6, change:
```html
<title>Texas Traffic Dashboard — Jimmy Hubbard</title>
```
to:
```html
<title>Live Traffic Dashboard — Jimmy Hubbard</title>
```

- [x] **Step 2: Update topbar label**

In `index.html` line 264, change:
```html
<span style="font-weight:600;font-size:0.9rem;">Texas Traffic Dashboard</span>
```
to:
```html
<span style="font-weight:600;font-size:0.9rem;">Live Traffic Dashboard</span>
```

- [x] **Step 3: Verify**

Open `index.html` in a browser. Confirm:
- Browser tab shows "Live Traffic Dashboard — Jimmy Hubbard"
- Topbar shows "Live Traffic Dashboard"

- [x] **Step 4: Commit**

```bash
git add index.html
git commit -m "rename: Texas Traffic Dashboard → Live Traffic Dashboard"
```

---

### Task 2: Update footer attribution

**Files:**
- Modify: `index.html:291` (footer span)

- [x] **Step 1: Update footer text**

In `index.html` line 291, change:
```html
<span>Data: TxDOT 511 Public API · Map: Leaflet.js + OpenStreetMap</span>
```
to:
```html
<span>Data: Multi-State 511 APIs · Map: Leaflet.js + OpenStreetMap</span>
```

- [x] **Step 2: Verify**

Open `index.html` in browser. Confirm footer reads "Data: Multi-State 511 APIs · Map: Leaflet.js + OpenStreetMap".

- [x] **Step 3: Commit**

```bash
git add index.html
git commit -m "update footer attribution for multi-state coverage"
```

---

### Task 3: Update default map view

**Files:**
- Modify: `index.html:476-477` (MapManager constants)

- [x] **Step 1: Update map center and zoom**

In `index.html` lines 476–481, change:
```js
const TEXAS_CENTER = [30.2672, -97.7431];
const DEFAULT_ZOOM = 7;
let map, markersLayer;

function init() {
  map = L.map('map', { zoomControl: true }).setView(TEXAS_CENTER, DEFAULT_ZOOM);
```
to:
```js
const MAP_CENTER = [36.8, -102.8];
const DEFAULT_ZOOM = 5;
let map, markersLayer;

function init() {
  map = L.map('map', { zoomControl: true }).setView(MAP_CENTER, DEFAULT_ZOOM);
```

- [x] **Step 2: Verify**

Open `index.html` in browser. Confirm the map loads zoomed out enough to show the southwest/south-central US — all 5 cities should be visible before any incidents load.

- [x] **Step 3: Commit**

```bash
git add index.html
git commit -m "update map default center and zoom for 5-city view"
```

---

### Task 4: Replace simulated incident pool

**Files:**
- Modify: `index.html:299-340` (the POOL array inside SimulatedData)

- [x] **Step 1: Replace the POOL array**

In `index.html`, replace the entire POOL array (lines 299–340, from `const POOL = [` through the closing `];`) with the following 50-incident array (10 per city):

```js
  const POOL = [
    // ── Scottsdale, AZ ───────────────────────────────────────────────
    { id:'sc-001', type:'accident',     title:'Multi-vehicle collision',      location:'Loop 101 @ Scottsdale Rd, Scottsdale',    severity:'major',    lat:33.5062, lng:-111.9026, details:'2 lanes blocked' },
    { id:'sc-002', type:'construction', title:'Road widening project',        location:'SR-51 @ Shea Blvd, Scottsdale',           severity:'moderate', lat:33.5817, lng:-111.9752, details:'Nightly closures 9pm–5am' },
    { id:'sc-003', type:'hazard',       title:'Debris on roadway',            location:'US-60 @ Power Rd, Mesa',                  severity:'minor',    lat:33.3731, lng:-111.6952, details:'' },
    { id:'sc-004', type:'closure',      title:'Emergency road closure',       location:'AZ-87 @ McDowell Rd, Scottsdale',         severity:'major',    lat:33.4614, lng:-111.6282, details:'Avoid area' },
    { id:'sc-005', type:'accident',     title:'Rear-end collision',           location:'Loop 101 @ Indian School Rd, Scottsdale', severity:'moderate', lat:33.4942, lng:-111.9261, details:'Right shoulder blocked' },
    { id:'sc-006', type:'construction', title:'Bridge resurfacing',           location:'SR-202 @ Rural Rd, Tempe',                severity:'moderate', lat:33.4151, lng:-111.8271, details:'Single lane until midnight' },
    { id:'sc-007', type:'hazard',       title:'Stalled vehicle',              location:'Loop 101 @ Frank Lloyd Wright Blvd',      severity:'minor',    lat:33.6234, lng:-111.8894, details:'' },
    { id:'sc-008', type:'closure',      title:'Planned lane closure',         location:'US-60 @ Gilbert Rd, Gilbert',             severity:'moderate', lat:33.3554, lng:-111.7882, details:'Until 3:00 PM' },
    { id:'sc-009', type:'accident',     title:'Commercial truck accident',    location:'AZ-101 @ Pima Rd, Scottsdale',            severity:'major',    lat:33.6522, lng:-111.8981, details:'Hazmat team responding' },
    { id:'sc-010', type:'construction', title:'Utility work',                 location:'Scottsdale Rd @ Camelback Rd',            severity:'minor',    lat:33.5069, lng:-111.9264, details:'1 lane closed' },

    // ── Denver, CO ───────────────────────────────────────────────────
    { id:'den-001', type:'accident',     title:'Jackknifed truck',            location:'I-25 @ I-70 Interchange, Denver',         severity:'major',    lat:39.7621, lng:-104.9883, details:'3 lanes blocked, use alternate' },
    { id:'den-002', type:'construction', title:'Overpass construction',       location:'I-70 @ Wadsworth Blvd, Lakewood',         severity:'major',    lat:39.7463, lng:-105.0863, details:'Reduced to 2 lanes' },
    { id:'den-003', type:'hazard',       title:'Ice on roadway',              location:'I-70 @ Eisenhower Tunnel approach',        severity:'major',    lat:39.7047, lng:-105.4193, details:'Drive with caution' },
    { id:'den-004', type:'closure',      title:'Road closure',                location:'I-25 @ Colorado Blvd, Denver',            severity:'moderate', lat:39.7245, lng:-104.9625, details:'Full closure' },
    { id:'den-005', type:'accident',     title:'Rollover accident',           location:'C-470 @ Kipling St, Littleton',           severity:'major',    lat:39.5944, lng:-105.0793, details:'All lanes blocked, delays 45 min' },
    { id:'den-006', type:'construction', title:'Lane widening',               location:'US-36 @ Sheridan Blvd, Westminster',      severity:'moderate', lat:39.8271, lng:-105.0522, details:'Nightly closures 9pm–5am' },
    { id:'den-007', type:'hazard',       title:'Debris on roadway',           location:'I-225 @ Parker Rd, Aurora',               severity:'minor',    lat:39.6521, lng:-104.8343, details:'' },
    { id:'den-008', type:'closure',      title:'Scheduled maintenance',       location:'E-470 @ Arapahoe Rd, Aurora',             severity:'minor',    lat:39.5882, lng:-104.7843, details:'Nightly 10pm–4am' },
    { id:'den-009', type:'accident',     title:'Sideswipe collision',         location:'I-25 @ Broadway, Denver',                 severity:'minor',    lat:39.7041, lng:-104.9884, details:'Shoulder only' },
    { id:'den-010', type:'construction', title:'Drainage project',            location:'I-70 @ Quebec St, Denver',                severity:'moderate', lat:39.7634, lng:-104.9343, details:'1 lane each direction' },

    // ── Salt Lake City, UT ───────────────────────────────────────────
    { id:'slc-001', type:'accident',     title:'Multi-vehicle accident',      location:'I-15 @ 2100 S, Salt Lake City',           severity:'major',    lat:40.7185, lng:-111.8908, details:'2 lanes blocked' },
    { id:'slc-002', type:'construction', title:'Road construction',           location:'I-80 @ Bangerter Hwy, West Valley City',  severity:'major',    lat:40.7164, lng:-111.9789, details:'Until 5:00 PM' },
    { id:'slc-003', type:'hazard',       title:'Debris on roadway',           location:'I-215 @ 4500 S, Murray',                  severity:'minor',    lat:40.6893, lng:-111.8521, details:'' },
    { id:'slc-004', type:'closure',      title:'Emergency road closure',      location:'SR-201 @ Redwood Rd, Salt Lake City',     severity:'major',    lat:40.6963, lng:-111.9383, details:'Full closure' },
    { id:'slc-005', type:'accident',     title:'Rear-end collision',          location:'I-15 @ 600 N, Salt Lake City',            severity:'moderate', lat:40.7791, lng:-111.8883, details:'Right shoulder blocked' },
    { id:'slc-006', type:'construction', title:'Bridge work',                 location:'I-80 @ 1300 E, Salt Lake City',           severity:'moderate', lat:40.7483, lng:-111.8472, details:'Single lane until midnight' },
    { id:'slc-007', type:'hazard',       title:'Stalled vehicle',             location:'SR-89 @ 5400 S, Murray',                  severity:'minor',    lat:40.6603, lng:-111.8742, details:'' },
    { id:'slc-008', type:'closure',      title:'Planned lane closure',        location:'I-215 @ State St, Murray',                severity:'moderate', lat:40.6811, lng:-111.8892, details:'Until 3:00 PM' },
    { id:'slc-009', type:'accident',     title:'Jackknifed truck',            location:'I-15 @ Farmington, Davis County',         severity:'major',    lat:40.9891, lng:-111.8962, details:'3 lanes blocked, use alternate' },
    { id:'slc-010', type:'construction', title:'Pavement resurfacing',        location:'SR-201 @ 5600 W, West Valley City',       severity:'minor',    lat:40.7023, lng:-112.0342, details:'Intermittent closures' },

    // ── Nashville, TN ────────────────────────────────────────────────
    { id:'nas-001', type:'accident',     title:'Multi-vehicle accident',      location:'I-40 @ I-24 Merge, Nashville',            severity:'major',    lat:36.1653, lng:-86.7531, details:'2 lanes blocked' },
    { id:'nas-002', type:'construction', title:'Road construction',           location:'I-65 @ Old Hickory Blvd, Nashville',      severity:'moderate', lat:36.2981, lng:-86.7643, details:'Until 5:00 PM' },
    { id:'nas-003', type:'hazard',       title:'Debris on roadway',           location:'I-24 @ Harding Pl, Nashville',            severity:'minor',    lat:36.0841, lng:-86.7334, details:'' },
    { id:'nas-004', type:'closure',      title:'Emergency road closure',      location:'SR-840 @ Columbia Pike, Franklin',        severity:'major',    lat:35.9231, lng:-86.8341, details:'Full closure' },
    { id:'nas-005', type:'accident',     title:'Rear-end collision',          location:'I-40 @ White Bridge Rd, Nashville',       severity:'moderate', lat:36.1471, lng:-86.8543, details:'Right shoulder blocked' },
    { id:'nas-006', type:'construction', title:'Bridge work',                 location:'I-65 @ Trinity Ln, Nashville',            severity:'major',    lat:36.2241, lng:-86.7893, details:'Reduced to 2 lanes' },
    { id:'nas-007', type:'hazard',       title:'Stalled vehicle',             location:'I-440 @ Nolensville Pike, Nashville',     severity:'minor',    lat:36.1041, lng:-86.7341, details:'' },
    { id:'nas-008', type:'closure',      title:'Planned lane closure',        location:'US-70 @ Charlotte Ave, Nashville',        severity:'moderate', lat:36.1531, lng:-86.8741, details:'Until 3:00 PM' },
    { id:'nas-009', type:'accident',     title:'Jackknifed truck',            location:'I-24 @ Murfreesboro Pike, Nashville',     severity:'major',    lat:36.0941, lng:-86.6231, details:'3 lanes blocked, use alternate' },
    { id:'nas-010', type:'construction', title:'Utility work',                location:'I-40 @ Briley Pkwy, Nashville',           severity:'minor',    lat:36.1781, lng:-86.8121, details:'1 lane closed' },

    // ── Dallas / Fort Worth, TX ──────────────────────────────────────
    { id:'dfw-001', type:'accident',     title:'Multi-vehicle accident',      location:'I-35E @ I-30, Fort Worth',                severity:'major',    lat:32.7481, lng:-97.0883, details:'2 lanes blocked' },
    { id:'dfw-002', type:'construction', title:'Overpass construction',       location:'I-635 @ US-75, Dallas',                   severity:'major',    lat:32.9094, lng:-96.7674, details:'Reduced to 2 lanes' },
    { id:'dfw-003', type:'hazard',       title:'Stalled vehicle',             location:'I-20 @ Great SW Pkwy, Grand Prairie',     severity:'minor',    lat:32.6841, lng:-97.0654, details:'' },
    { id:'dfw-004', type:'closure',      title:'Water main repair',           location:'Loop 12 @ Buckner Blvd, Dallas',          severity:'moderate', lat:32.7767, lng:-96.8467, details:'Closed until further notice' },
    { id:'dfw-005', type:'accident',     title:'Rear-end collision',          location:'SH-114 @ DFW Airport Rd, Grapevine',      severity:'moderate', lat:32.8841, lng:-97.0423, details:'Shoulder blocked' },
    { id:'dfw-006', type:'construction', title:'Lane widening project',       location:'I-30 @ Cockrell Hill Rd, Dallas',         severity:'moderate', lat:32.7481, lng:-96.9323, details:'Nightly closures 9pm–5am' },
    { id:'dfw-007', type:'hazard',       title:'Debris on roadway',           location:'US-75 @ LBJ Freeway, Dallas',             severity:'minor',    lat:32.9111, lng:-96.7643, details:'' },
    { id:'dfw-008', type:'closure',      title:'Scheduled maintenance',       location:'I-20 @ Duncanville Rd, Duncanville',      severity:'minor',    lat:32.6521, lng:-96.9083, details:'Nightly 10pm–4am' },
    { id:'dfw-009', type:'accident',     title:'Commercial truck accident',   location:'I-35E @ I-635, Farmers Branch',           severity:'major',    lat:32.9243, lng:-96.9883, details:'Hazmat team responding' },
    { id:'dfw-010', type:'construction', title:'Median barrier install',      location:'SH-360 @ I-30, Arlington',                severity:'moderate', lat:32.7281, lng:-97.0723, details:'Until end of month' },
  ];
```

- [x] **Step 2: Update the seed count**

The pool is now 50 incidents. The seed picks 25 by default — this is still correct (50% active at any time), no change needed. Confirm line ~347 still reads:
```js
shuffled.slice(0, 25).forEach(i => activeIds.add(i.id));
```

- [x] **Step 3: Verify in browser**

Open `index.html` in browser. Confirm:
- ~25 incidents appear across all 5 cities
- Markers appear in Arizona, Colorado, Utah, Tennessee, and Texas
- Sidebar incident cards show city names in locations (e.g. "Salt Lake City", "Nashville", "Scottsdale")
- Clicking an incident card zooms to that marker correctly
- Filter pills (Accidents / Construction / Hazards / Closures) still work
- Search still works (try "Denver" or "Nashville")
- Refresh button cycles incidents in/out

- [x] **Step 4: Commit**

```bash
git add index.html
git commit -m "expand incident pool to 5 cities: Scottsdale, Denver, SLC, Nashville, DFW"
```

---

### Task 5: Deploy to S3 and verify live

**Files:**
- Deploy: `index.html` → S3 bucket `jimmy-traffic-dashboard`

- [x] **Step 1: Upload to S3**

Run in WSL terminal:
```bash
aws s3 cp /mnt/c/Users/jimmy/Desktop/ClaudeCode/traffic-dashboard/index.html \
  s3://jimmy-traffic-dashboard/index.html \
  --content-type "text/html"
```
Expected output: `upload: ./index.html to s3://jimmy-traffic-dashboard/index.html`

- [x] **Step 2: Verify live site**

Open: `http://jimmy-traffic-dashboard.s3-website-us-east-1.amazonaws.com`

Confirm:
- Browser tab: "Live Traffic Dashboard — Jimmy Hubbard"
- Topbar: "Live Traffic Dashboard"
- Map loads zoomed out showing south-central/southwest US
- Incidents visible across multiple states
- Footer: "Data: Multi-State 511 APIs · Map: Leaflet.js + OpenStreetMap"

- [x] **Step 3: Final commit**

```bash
git add index.html
git commit -m "deploy: live traffic dashboard multi-city expansion"
git push
```
