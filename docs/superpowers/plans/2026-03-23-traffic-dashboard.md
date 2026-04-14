# Texas Traffic Dashboard Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` traffic incident dashboard with Leaflet.js map, color-coded markers, left sidebar, and a TxDOT → SF511 → Simulated data cascade, ready to deploy to S3.

**Architecture:** All code lives in one `index.html` file with inline `<style>` and `<script>` blocks. The app is structured as discrete JS modules (functions/objects) within that script block: `DataSources`, `MapManager`, `SidebarManager`, `FilterState`, `RefreshManager`. Each has a clear interface and no shared global state except the normalized incident array.

**Tech Stack:** HTML5, CSS3, Vanilla JS (ES6+), Leaflet.js 1.9.x (CDN), OpenStreetMap tiles, Inter font (Google Fonts CDN)

---

## File Structure

```
traffic-dashboard/
└── index.html          # Single deliverable — all CSS + JS inline
```

Everything below is a section within `index.html`. Tasks map to logical sections of the file built in order.

---

### Task 1: HTML Shell + CSS Layout

**Files:**
- Create: `index.html` (scaffold only — no JS yet)

- [ ] **Step 1: Create the HTML skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Texas Traffic Dashboard — Jimmy Hubbard</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
  <style>/* CSS here */</style>
</head>
<body>
  <div id="app">
    <div id="topbar"></div>
    <div id="body">
      <div id="sidebar"></div>
      <div id="map"></div>
    </div>
    <div id="footer"></div>
  </div>
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>/* JS here */</script>
</body>
</html>
```

- [ ] **Step 2: Add CSS — reset, layout, colors**

Add inside `<style>`:

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #0d1117;
  --surface: #161b22;
  --border: #30363d;
  --text: #e2e8f0;
  --muted: #94a3b8;
  --dim: #64748b;
  --red: #ef4444;
  --orange: #f97316;
  --yellow: #eab308;
  --blue: #3b82f6;
}

body {
  background: var(--bg);
  color: var(--text);
  font-family: 'Inter', 'Segoe UI', sans-serif;
  height: 100vh;
  overflow: hidden;
}

#app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

/* Top bar */
#topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: var(--surface);
  border-bottom: 1px solid var(--border);
  padding: 10px 16px;
  flex-shrink: 0;
  gap: 12px;
}

/* Body */
#body {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* Sidebar */
#sidebar {
  width: 280px;
  min-width: 280px;
  background: var(--surface);
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* Map */
#map {
  flex: 1;
  background: #1a2332;
}

/* Footer */
#footer {
  background: var(--surface);
  border-top: 1px solid var(--border);
  padding: 6px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.7rem;
  color: var(--dim);
  flex-shrink: 0;
}

/* Responsive */
@media (max-width: 768px) {
  #body { flex-direction: column; }
  #sidebar { width: 100%; min-width: unset; height: 45vh; border-right: none; border-bottom: 1px solid var(--border); }
  #map { flex: 1; min-height: 0; }
  body { overflow: auto; }
  #app { height: auto; min-height: 100vh; }
}
```

- [ ] **Step 3: Verify layout in browser**

Open `index.html` in a browser. You should see:
- Dark `#0d1117` background fills the page
- A slightly lighter `#161b22` top bar
- Left column (280px) + right column (flex:1) in the body
- Footer bar at the bottom
- No JS errors in console

- [ ] **Step 4: Commit**

```bash
cd /mnt/c/Users/jimmy/Desktop/ClaudeCode/traffic-dashboard
git init
git add index.html
git commit -m "feat: scaffold HTML layout and CSS design system"
```

---

### Task 2: Leaflet Map Setup

**Files:**
- Modify: `index.html` — add map init JS

- [ ] **Step 1: Add topbar and footer static HTML**

Inside `#topbar`:
```html
<div id="topbar-left" style="display:flex;align-items:center;gap:10px;">
  <span style="font-size:1.2rem;">🚦</span>
  <span style="font-weight:600;font-size:0.9rem;">Texas Traffic Dashboard</span>
</div>
<div id="topbar-right" style="display:flex;align-items:center;gap:12px;">
  <span id="source-badge"></span>
  <span id="countdown" style="font-size:0.75rem;color:var(--dim);"></span>
  <button id="refresh-btn" onclick="manualRefresh()" style="background:var(--bg);border:1px solid var(--border);color:var(--muted);font-size:0.75rem;padding:4px 12px;border-radius:4px;cursor:pointer;">↻ Refresh</button>
</div>
```

Inside `#footer`:
```html
<span>Data: TxDOT 511 Public API · Map: Leaflet.js + OpenStreetMap</span>
<a href="https://jimmyhubbard2.cc/projects/" style="color:var(--dim);text-decoration:none;">← Back to Projects</a>
```

- [ ] **Step 2: Initialize Leaflet map in `<script>`**

```js
// ── Map ──────────────────────────────────────────────────────────────
const MapManager = (() => {
  const TEXAS_CENTER = [30.2672, -97.7431];
  const DEFAULT_ZOOM = 7;
  let map, markersLayer;

  function init() {
    map = L.map('map', { zoomControl: true }).setView(TEXAS_CENTER, DEFAULT_ZOOM);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
      maxZoom: 18
    }).addTo(map);
    markersLayer = L.layerGroup().addTo(map);
  }

  function getMap() { return map; }
  function getMarkersLayer() { return markersLayer; }

  return { init, getMap, getMarkersLayer };
})();

MapManager.init();
```

- [ ] **Step 3: Verify map renders**

Open in browser. You should see:
- OpenStreetMap tiles centered on Austin/Texas
- Zoom controls top-right
- Map fills available space
- No JS errors

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: initialize Leaflet map centered on Texas"
```

---

### Task 3: Simulated Data Engine

**Files:**
- Modify: `index.html` — add `SimulatedData` module

- [ ] **Step 1: Define the incident pool**

Add before `MapManager`:

```js
// ── Simulated Data ───────────────────────────────────────────────────
const SimulatedData = (() => {
  const POOL = [
    { id:'sim-001', type:'accident',     title:'Multi-vehicle accident',   location:'IH-35 N @ Exit 240, Austin',       severity:'major',    lat:30.3674, lng:-97.7198, details:'2 lanes blocked' },
    { id:'sim-002', type:'construction', title:'Road construction',        location:'US-290 W @ Loop 360, Austin',      severity:'moderate', lat:30.2987, lng:-97.8012, details:'Until 5:00 PM' },
    { id:'sim-003', type:'hazard',       title:'Debris on roadway',        location:'TX-1 @ Mopac Expwy SB, Austin',    severity:'minor',    lat:30.3215, lng:-97.7498, details:'' },
    { id:'sim-004', type:'closure',      title:'Road closure',             location:'IH-10 E @ Beltway 8, Houston',     severity:'major',    lat:29.7604, lng:-95.1835, details:'Full closure' },
    { id:'sim-005', type:'accident',     title:'Rear-end collision',       location:'IH-35 S @ Slaughter Ln, Austin',   severity:'moderate', lat:30.1883, lng:-97.7841, details:'Right shoulder blocked' },
    { id:'sim-006', type:'construction', title:'Bridge work',              location:'IH-10 W @ Loop 1604, San Antonio', severity:'major',    lat:29.5341, lng:-98.6316, details:'Single lane until midnight' },
    { id:'sim-007', type:'hazard',       title:'Stalled vehicle',          location:'US-183 @ Airport Blvd, Austin',    severity:'minor',    lat:30.3005, lng:-97.6890, details:'' },
    { id:'sim-008', type:'closure',      title:'Flooded roadway',          location:'TX-71 @ Bastrop River Rd',         severity:'major',    lat:30.1105, lng:-97.3140, details:'Both directions closed' },
    { id:'sim-009', type:'accident',     title:'Jackknifed truck',         location:'IH-35 N @ Round Rock, Williamson', severity:'major',    lat:30.5085, lng:-97.6789, details:'3 lanes blocked, use alternate' },
    { id:'sim-010', type:'construction', title:'Lane widening project',    location:'SH-45 @ IH-35 Interchange',        severity:'moderate', lat:30.4215, lng:-97.7341, details:'Nightly closures 9pm–5am' },
    { id:'sim-011', type:'hazard',       title:'Animal on roadway',        location:'US-290 E @ Manor, Austin',         severity:'minor',    lat:30.3411, lng:-97.5561, details:'' },
    { id:'sim-012', type:'accident',     title:'Rollover accident',        location:'IH-45 N @ Greenspoint, Houston',   severity:'major',    lat:29.9293, lng:-95.4144, details:'All lanes blocked, delays 45 min' },
    { id:'sim-013', type:'construction', title:'Pavement resurfacing',     location:'Loop 360 @ Bee Caves Rd, Austin',  severity:'minor',    lat:30.2651, lng:-97.8214, details:'Intermittent closures' },
    { id:'sim-014', type:'hazard',       title:'Spilled load',             location:'IH-10 W @ El Paso',                severity:'moderate', lat:31.7619, lng:-106.4850,details:'Right 2 lanes blocked' },
    { id:'sim-015', type:'closure',      title:'Planned lane closure',     location:'US-77 @ Corpus Christi',           severity:'moderate', lat:27.8006, lng:-97.3964, details:'Until 3:00 PM' },
    { id:'sim-016', type:'accident',     title:'Sideswipe collision',      location:'SH-183 @ DFW Airport Rd',          severity:'minor',    lat:32.8998, lng:-97.0403, details:'Shoulder only' },
    { id:'sim-017', type:'construction', title:'Overpass construction',    location:'IH-635 @ US-75, Dallas',           severity:'major',    lat:32.9094, lng:-96.7674, details:'Reduced to 2 lanes' },
    { id:'sim-018', type:'hazard',       title:'Ice on bridge',            location:'IH-20 @ Abilene',                  severity:'major',    lat:32.4487, lng:-99.7331, details:'Drive with caution' },
    { id:'sim-019', type:'closure',      title:'Emergency road closure',   location:'IH-27 @ Lubbock',                  severity:'major',    lat:33.5779, lng:-101.8552,details:'Avoid area' },
    { id:'sim-020', type:'accident',     title:'Head-on collision',        location:'US-84 @ Waco',                     severity:'major',    lat:31.5493, lng:-97.1467, details:'All lanes blocked' },
    { id:'sim-021', type:'construction', title:'Utility work',             location:'SH-6 @ Sugar Land, Houston',       severity:'minor',    lat:29.6197, lng:-95.6349, details:'1 lane closed' },
    { id:'sim-022', type:'hazard',       title:'Wrong-way driver reported',location:'IH-35 @ Georgetown',               severity:'major',    lat:30.6382, lng:-97.6789, details:'Use extreme caution' },
    { id:'sim-023', type:'closure',      title:'Bridge inspection closure',location:'US-90 @ Laredo',                   severity:'moderate', lat:27.5306, lng:-99.4803, details:'Closed 8am–noon' },
    { id:'sim-024', type:'accident',     title:'Motorcycle accident',      location:'TX-130 @ Pflugerville',            severity:'moderate', lat:30.4394, lng:-97.6200, details:'Shoulder blocked' },
    { id:'sim-025', type:'construction', title:'Median barrier install',   location:'IH-35 @ New Braunfels',            severity:'moderate', lat:29.7030, lng:-98.1245, details:'Until end of month' },
    { id:'sim-026', type:'hazard',       title:'High wind warning',        location:'IH-10 @ Kerrville',                severity:'minor',    lat:30.0474, lng:-99.1403, details:'Gusts to 55mph' },
    { id:'sim-027', type:'accident',     title:'Commercial truck accident',location:'IH-20 @ Odessa',                   severity:'major',    lat:31.8457, lng:-102.3676,details:'Hazmat team responding' },
    { id:'sim-028', type:'closure',      title:'Scheduled maintenance',    location:'SH-130 @ Seguin',                  severity:'minor',    lat:29.5688, lng:-97.9642, details:'Nightly 10pm–4am' },
    { id:'sim-029', type:'hazard',       title:'Fog advisory',             location:'US-59 @ Victoria',                 severity:'minor',    lat:28.8053, lng:-97.0036, details:'Visibility under 1/4 mile' },
    { id:'sim-030', type:'accident',     title:'Fender bender',            location:'Loop 1 @ 51st St, Austin',         severity:'minor',    lat:30.3174, lng:-97.7484, details:'Blocking right lane' },
    { id:'sim-031', type:'construction', title:'Drainage project',         location:'SH-71 @ Bastrop',                  severity:'moderate', lat:30.1105, lng:-97.3140, details:'1 lane each direction' },
    { id:'sim-032', type:'hazard',       title:'Tire debris on roadway',   location:'IH-410 @ San Antonio',             severity:'minor',    lat:29.4241, lng:-98.4936, details:'' },
    { id:'sim-033', type:'closure',      title:'Water main repair',        location:'Loop 12 @ Dallas',                 severity:'moderate', lat:32.7767, lng:-96.8467, details:'Closed until further notice' },
    { id:'sim-034', type:'accident',     title:'Minor collision',          location:'US-183 @ Cedar Park',              severity:'minor',    lat:30.5052, lng:-97.8203, details:'' },
    { id:'sim-035', type:'construction', title:'Traffic signal upgrade',   location:'IH-35 @ Kyle',                     severity:'minor',    lat:29.9888, lng:-97.8772, details:'Intermittent stops' },
    { id:'sim-036', type:'accident',     title:'Pedestrian incident',      location:'IH-35 Frontage Rd @ 6th St, Austin',severity:'major',  lat:30.2669, lng:-97.7428, details:'Emergency responders on scene' },
    { id:'sim-037', type:'hazard',       title:'Flash flood watch',        location:'FM 620 @ Lake Travis',             severity:'moderate', lat:30.3838, lng:-97.9141, details:'Do not cross flooded roads' },
    { id:'sim-038', type:'construction', title:'Interchange rebuild',      location:'IH-35 @ SH-130 Split, Georgetown', severity:'major',    lat:30.6382, lng:-97.6100, details:'Major delays expected' },
    { id:'sim-039', type:'closure',      title:'Oversize load escort',     location:'US-281 @ Johnson City',            severity:'minor',    lat:30.2774, lng:-98.4105, details:'Temporary 10-min closures' },
    { id:'sim-040', type:'accident',     title:'Abandoned vehicle fire',   location:'IH-10 @ Beaumont',                 severity:'major',    lat:30.0802, lng:-94.1266, details:'All lanes blocked, fire dept. on scene' },
  ];

  // Stable active set — persists across refreshes
  let activeIds = new Set();

  function getActiveIncidents() {
    // On first call, seed with 25 random incidents
    if (activeIds.size === 0) {
      const shuffled = [...POOL].sort(() => Math.random() - 0.5);
      shuffled.slice(0, 25).forEach(i => activeIds.add(i.id));
    } else {
      // Each refresh: remove 2-4 and add 2-4 different ones
      const activeArr = [...activeIds];
      const toRemove = activeArr.sort(() => Math.random() - 0.5).slice(0, Math.floor(Math.random() * 3) + 2);
      toRemove.forEach(id => activeIds.delete(id));
      const inactive = POOL.filter(i => !activeIds.has(i.id));
      const toAdd = inactive.sort(() => Math.random() - 0.5).slice(0, toRemove.length);
      toAdd.forEach(i => activeIds.add(i.id));
    }
    return POOL
      .filter(i => activeIds.has(i.id))
      .map(i => ({ ...i, updatedAt: new Date().toLocaleTimeString() }));
  }

  return { getActiveIncidents };
})();
```

- [ ] **Step 2: Verify simulated data in browser console**

Open browser console and run:
```js
console.log(SimulatedData.getActiveIncidents());
```
Expected: Array of ~25 objects each with id, type, title, location, severity, lat, lng, details, updatedAt.

Run it twice — some incidents should change between calls (2–4 different IDs).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add simulated incident data engine with stable IDs"
```

---

### Task 4: Data Source Cascade

**Files:**
- Modify: `index.html` — add `DataSources` module

- [ ] **Step 1: Add normalization helpers + TxDOT source**

```js
// ── Data Sources ─────────────────────────────────────────────────────
const DataSources = (() => {
  const TIMEOUT_MS = 3000;

  function fetchWithTimeout(url, options = {}) {
    const controller = new AbortController();
    const id = setTimeout(() => controller.abort(), TIMEOUT_MS);
    return fetch(url, { ...options, signal: controller.signal })
      .finally(() => clearTimeout(id));
  }

  // Map TxDOT severity strings to internal scale
  function mapSeverity(raw) {
    if (!raw) return 'minor';
    const r = String(raw).toLowerCase();
    if (r.includes('major') || r.includes('high') || r === '3') return 'major';
    if (r.includes('moderate') || r.includes('medium') || r === '2') return 'moderate';
    return 'minor';
  }

  function mapType(raw) {
    if (!raw) return 'hazard';
    const r = String(raw).toLowerCase();
    if (r.includes('accid') || r.includes('crash') || r.includes('collision')) return 'accident';
    if (r.includes('construct') || r.includes('roadwork') || r.includes('work')) return 'construction';
    if (r.includes('clos')) return 'closure';
    return 'hazard';
  }

  function normalizeTxDOT(raw) {
    // TxDOT 511 returns an array of event objects
    if (!Array.isArray(raw)) return null;
    return raw
      .map(e => ({
        id: String(e.ID || e.id || Math.random()),
        type: mapType(e.EventType || e.eventType || ''),
        title: e.EventDescription || e.eventDescription || e.EventType || 'Incident',
        location: e.RoadwayName ? `${e.RoadwayName} @ ${e.DirectionOfTravel || ''}`.trim() : (e.location || ''),
        severity: mapSeverity(e.Severity || e.severity || ''),
        lat: parseFloat(e.Latitude || e.latitude || 0),
        lng: parseFloat(e.Longitude || e.longitude || 0),
        details: e.LanesAffected || e.lanesAffected || '',
        updatedAt: e.LastUpdated || e.lastUpdated || new Date().toLocaleTimeString()
      }))
      .filter(i => i.lat !== 0 && i.lng !== 0);
  }

  function normalizeSF511(raw) {
    // SF 511 returns { events: { event: [...] } }
    const events = raw?.events?.event || raw?.Events?.Event || raw || [];
    if (!Array.isArray(events)) return null;
    return events
      .map(e => ({
        id: String(e.id || e.ID || Math.random()),
        type: mapType(e.event_type || e.EventType || ''),
        title: e.headline || e.Headline || e.event_type || 'Incident',
        location: e.roads_affected || e.RoadsAffected || e.id || '',
        severity: mapSeverity(e.severity || e.Severity || ''),
        lat: parseFloat(e.geography?.coordinates?.[1] || e.Latitude || 0),
        lng: parseFloat(e.geography?.coordinates?.[0] || e.Longitude || 0),
        details: e.description || e.Description || '',
        updatedAt: e.updated || new Date().toLocaleTimeString()
      }))
      .filter(i => i.lat !== 0 && i.lng !== 0);
  }

  async function tryTxDOT() {
    const endpoints = [
      'https://www.drivetexas.org/api/traffic/getIncidents',
      'https://traffic.511tx.org/api/v2/get/event',
    ];
    for (const url of endpoints) {
      try {
        const res = await fetchWithTimeout(url);
        if (!res.ok) continue;
        const data = await res.json();
        const normalized = normalizeTxDOT(data);
        if (normalized && normalized.length > 0) return { incidents: normalized, source: 'txdot' };
      } catch (_) { /* continue */ }
    }
    return null;
  }

  async function trySF511() {
    // Free public key — acceptable for client-side portfolio project
    const API_KEY = '2f474470-ca0b-4821-a880-1c0cbf1c1f5d';
    const url = `https://api.511.org/traffic/events?api_key=${API_KEY}&format=json`;
    try {
      const res = await fetchWithTimeout(url);
      if (!res.ok) return null;
      const data = await res.json();
      const normalized = normalizeSF511(data);
      if (normalized && normalized.length > 0) return { incidents: normalized, source: 'sf511' };
    } catch (_) { /* continue */ }
    return null;
  }

  async function fetchIncidents() {
    const result = await tryTxDOT() || await trySF511();
    if (result) return result;
    // Demo Mode — guaranteed
    return { incidents: SimulatedData.getActiveIncidents(), source: 'demo' };
  }

  return { fetchIncidents };
})();
```

- [ ] **Step 2: Verify cascade in browser console**

```js
DataSources.fetchIncidents().then(r => console.log(r.source, r.incidents.length));
```
Expected: logs `"demo"` (CORS will block live APIs on local file) and a number between 20–30.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add TxDOT→SF511→Demo data source cascade with normalization"
```

---

### Task 5: Render Markers + Source Badge

**Files:**
- Modify: `index.html` — add marker rendering, source badge update

- [ ] **Step 1: Add marker color config and custom icon factory**

```js
// ── Marker Rendering ─────────────────────────────────────────────────
const INCIDENT_COLORS = {
  accident:     '#ef4444',
  construction: '#f97316',
  hazard:       '#eab308',
  closure:      '#3b82f6'
};

function makeIcon(type) {
  const color = INCIDENT_COLORS[type] || '#94a3b8';
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 18 18">
    <circle cx="9" cy="9" r="7" fill="${color}" stroke="white" stroke-width="2"/>
  </svg>`;
  return L.divIcon({
    html: `<div style="filter:drop-shadow(0 0 4px ${color}88)">${svg}</div>`,
    className: '',
    iconSize: [18, 18],
    iconAnchor: [9, 9],
    popupAnchor: [0, -12]
  });
}
```

- [ ] **Step 2: Add `renderIncidents(incidents)` function**

```js
let currentIncidents = [];
let activeFilters = new Set(['accident', 'construction', 'hazard', 'closure']);

function renderIncidents(incidents) {
  currentIncidents = incidents;
  renderMap();
  renderSidebar();
  updateCounts();
}

function getFiltered() {
  return currentIncidents.filter(i => activeFilters.has(i.type));
}

function renderMap() {
  const layer = MapManager.getMarkersLayer();
  layer.clearLayers();
  getFiltered().forEach(incident => {
    const marker = L.marker([incident.lat, incident.lng], { icon: makeIcon(incident.type) })
      .bindPopup(makePopupHTML(incident))
      .on('click', () => highlightSidebarCard(incident.id));
    marker._incidentId = incident.id;
    layer.addLayer(marker);
  });
}

function makePopupHTML(i) {
  const color = INCIDENT_COLORS[i.type] || '#94a3b8';
  return `
    <div style="font-family:Inter,sans-serif;min-width:160px;">
      <div style="font-weight:600;font-size:0.8rem;margin-bottom:4px;">
        <span style="color:${color}">●</span> ${i.title}
      </div>
      <div style="font-size:0.72rem;color:#94a3b8;">${i.location}</div>
      <div style="font-size:0.72rem;color:#64748b;margin-top:2px;text-transform:capitalize;">${i.severity}${i.details ? ' · ' + i.details : ''}</div>
    </div>`;
}
```

- [ ] **Step 3: Add source badge updater**

```js
function updateSourceBadge(source) {
  const el = document.getElementById('source-badge');
  const styles = {
    txdot: { bg:'#1e2d1e', border:'#22c55e', color:'#22c55e', label:'● LIVE — TxDOT 511' },
    sf511: { bg:'#1e2d1e', border:'#22c55e', color:'#22c55e', label:'● LIVE — SF Bay 511' },
    demo:  { bg:'#2d2a1e', border:'#eab308', color:'#eab308', label:'◎ DEMO MODE' }
  };
  const s = styles[source] || styles.demo;
  el.style.cssText = `background:${s.bg};border:1px solid ${s.border};color:${s.color};font-size:0.7rem;padding:2px 10px;border-radius:10px;font-weight:600;`;
  el.textContent = s.label;
}
```

- [ ] **Step 4: Wire up initial load**

```js
async function loadData() {
  const { incidents, source } = await DataSources.fetchIncidents();
  updateSourceBadge(source);
  renderIncidents(incidents);
}

// Boot
MapManager.init();
loadData();
```

- [ ] **Step 5: Verify in browser**

Open `index.html`. You should see:
- Colored circular markers on the Texas map
- Source badge in top bar showing "DEMO MODE" (yellow)
- No JS errors

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: render color-coded markers and source badge"
```

---

### Task 6: Sidebar — Counts, Filters, Incident List

**Files:**
- Modify: `index.html` — add sidebar HTML structure + `renderSidebar`, `updateCounts` functions, CSS

- [ ] **Step 1: Add sidebar static HTML structure**

Replace `<div id="sidebar"></div>` with:

```html
<div id="sidebar">
  <div id="counts-section">
    <div class="section-label">Incident Summary</div>
    <div id="counts-grid"></div>
    <div id="total-count"></div>
  </div>
  <div id="filter-section">
    <div class="section-label">Filter</div>
    <div id="filter-pills"></div>
  </div>
  <div id="incident-list"></div>
</div>
```

- [ ] **Step 2: Add sidebar CSS**

```css
.section-label {
  font-size: 0.68rem;
  text-transform: uppercase;
  letter-spacing: 1.2px;
  color: var(--muted);
  margin-bottom: 8px;
}

#counts-section {
  padding: 12px;
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
}

#counts-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 6px;
  margin-bottom: 8px;
}

.count-card {
  border-radius: 6px;
  padding: 8px;
  text-align: center;
  border: 1px solid;
}

.count-card .count-num { font-size: 1.2rem; font-weight: 700; }
.count-card .count-lbl { font-size: 0.68rem; color: var(--muted); margin-top: 2px; }

#total-count {
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 6px;
  text-align: center;
  font-size: 0.78rem;
  color: var(--muted);
}

#total-count strong { color: var(--text); margin-right: 4px; }

#filter-section {
  padding: 10px 12px;
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
}

#filter-pills { display: flex; flex-wrap: wrap; gap: 6px; }

.filter-pill {
  font-size: 0.7rem;
  padding: 3px 10px;
  border-radius: 10px;
  cursor: pointer;
  border: 1px solid transparent;
  font-weight: 600;
  transition: opacity 0.15s;
}

.filter-pill.inactive { opacity: 0.35; }

#incident-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.incident-card {
  background: #1c2128;
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 9px 10px;
  cursor: pointer;
  transition: border-color 0.15s, background 0.15s;
}

.incident-card:hover { background: #1e2530; }
.incident-card.active { border-color: var(--blue); background: #1a2535; }

.incident-card .card-header {
  display: flex;
  align-items: center;
  gap: 7px;
  margin-bottom: 3px;
}

.incident-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  flex-shrink: 0;
}

.incident-card .card-title { font-size: 0.78rem; font-weight: 600; }
.incident-card .card-location { font-size: 0.7rem; color: var(--muted); }
.incident-card .card-detail { font-size: 0.68rem; color: var(--dim); margin-top: 2px; text-transform: capitalize; }

.empty-list {
  text-align: center;
  color: var(--dim);
  font-size: 0.8rem;
  padding: 32px 16px;
}

/* Scrollbar styling */
#incident-list::-webkit-scrollbar { width: 4px; }
#incident-list::-webkit-scrollbar-track { background: transparent; }
#incident-list::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }
```

- [ ] **Step 3: Add `updateCounts()` function**

```js
const COUNT_CONFIG = [
  { type: 'accident',     color: '#ef4444', bg: '#1f1117', border: '#3d1515', label: 'Accidents' },
  { type: 'construction', color: '#f97316', bg: '#1f1a0f', border: '#3d2c0a', label: 'Construction' },
  { type: 'hazard',       color: '#eab308', bg: '#1f1f0a', border: '#3d3800', label: 'Hazards' },
  { type: 'closure',      color: '#3b82f6', bg: '#0a1520', border: '#0a2540', label: 'Closures' },
];

function updateCounts() {
  const filtered = getFiltered();
  const grid = document.getElementById('counts-grid');
  grid.innerHTML = COUNT_CONFIG.map(c => {
    const n = filtered.filter(i => i.type === c.type).length;
    return `<div class="count-card" style="background:${c.bg};border-color:${c.border};">
      <div class="count-num" style="color:${c.color}">${n}</div>
      <div class="count-lbl">${c.label}</div>
    </div>`;
  }).join('');
  document.getElementById('total-count').innerHTML =
    `<strong>${filtered.length}</strong> Total Active Incidents`;
}
```

- [ ] **Step 4: Add filter pills rendering**

```js
function renderFilterPills() {
  const container = document.getElementById('filter-pills');
  const pillDefs = [
    ...COUNT_CONFIG.map(c => ({ type: c.type, color: c.color, label: c.label })),
    { type: 'all', color: '#30363d', label: 'All' }
  ];
  container.innerHTML = pillDefs.map(p => {
    const isActive = p.type === 'all' ? activeFilters.size === 4 : activeFilters.has(p.type);
    const textColor = p.type === 'hazard' ? '#000' : '#fff';
    return `<button class="filter-pill ${isActive ? '' : 'inactive'}"
      style="background:${p.color};color:${textColor};"
      onclick="toggleFilter('${p.type}')">${p.label}</button>`;
  }).join('');
}

function toggleFilter(type) {
  if (type === 'all') {
    activeFilters = new Set(['accident', 'construction', 'hazard', 'closure']);
  } else {
    if (activeFilters.has(type) && activeFilters.size === 1) return; // keep at least one
    if (activeFilters.has(type)) activeFilters.delete(type);
    else activeFilters.add(type);
  }
  renderFilterPills();
  renderMap();
  renderSidebar();
  updateCounts();
}
```

- [ ] **Step 5: Add `renderSidebar()` and `highlightSidebarCard()`**

```js
const SEVERITY_ORDER = { major: 0, moderate: 1, minor: 2 };

function renderSidebar() {
  renderFilterPills();
  const list = document.getElementById('incident-list');
  const filtered = getFiltered()
    .slice()
    .sort((a, b) => (SEVERITY_ORDER[a.severity] ?? 2) - (SEVERITY_ORDER[b.severity] ?? 2));

  if (filtered.length === 0) {
    list.innerHTML = '<div class="empty-list">No incidents to display</div>';
    return;
  }

  list.innerHTML = filtered.map(i => {
    const color = INCIDENT_COLORS[i.type] || '#94a3b8';
    return `<div class="incident-card" id="card-${i.id}" onclick="onCardClick('${i.id}')">
      <div class="card-header">
        <span class="incident-dot" style="background:${color}"></span>
        <span class="card-title">${i.title}</span>
      </div>
      <div class="card-location">${i.location}</div>
      <div class="card-detail">${i.severity}${i.details ? ' · ' + i.details : ''}</div>
    </div>`;
  }).join('');
}

function highlightSidebarCard(id) {
  document.querySelectorAll('.incident-card').forEach(el => el.classList.remove('active'));
  const card = document.getElementById(`card-${id}`);
  if (card) {
    card.classList.add('active');
    card.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  }
}

function onCardClick(id) {
  highlightSidebarCard(id);
  const incident = currentIncidents.find(i => i.id === id);
  if (!incident) return;
  const map = MapManager.getMap();
  map.setView([incident.lat, incident.lng], Math.max(map.getZoom(), 11), { animate: true });
  // Open popup for that marker
  MapManager.getMarkersLayer().eachLayer(layer => {
    if (layer._incidentId === id) layer.openPopup();
  });
}
```

- [ ] **Step 6: Verify sidebar renders**

Open browser. Should see:
- 4 count cards (2×2) with numbers
- Filter pills (Accidents, Construction, Hazards, Closures, All)
- Sorted incident cards in scrollable list
- Clicking a card pans map and opens popup
- Clicking a marker highlights sidebar card

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: sidebar with counts, filter pills, incident list, bidirectional sync"
```

---

### Task 7: Auto-Refresh + Countdown Timer

**Files:**
- Modify: `index.html` — add `RefreshManager`

- [ ] **Step 1: Add `RefreshManager`**

```js
// ── Refresh Manager ──────────────────────────────────────────────────
const RefreshManager = (() => {
  const INTERVAL = 45;
  let countdown = INTERVAL;
  let timer = null;

  function updateDisplay() {
    const el = document.getElementById('countdown');
    if (el) el.textContent = `Refreshing in ${countdown}s`;
  }

  async function refresh() {
    countdown = INTERVAL;
    // Close any open popup before replacing markers
    MapManager.getMap().closePopup();
    const { incidents, source } = await DataSources.fetchIncidents();
    updateSourceBadge(source);
    renderIncidents(incidents);
  }

  function start() {
    updateDisplay();
    timer = setInterval(async () => {
      countdown--;
      updateDisplay();
      if (countdown <= 0) {
        countdown = INTERVAL;
        await refresh();
      }
    }, 1000);
  }

  function reset() {
    countdown = INTERVAL;
    updateDisplay();
  }

  return { start, reset, refresh };
})();
```

- [ ] **Step 2: Add `manualRefresh()` and wire up final boot sequence**

⚠️ **Replace** the temporary boot block added at the end of Task 5 Step 4 (`MapManager.init(); loadData();`) with the following. Do not append — calling `MapManager.init()` twice will throw a Leaflet error.

```js
async function manualRefresh() {
  RefreshManager.reset();
  await RefreshManager.refresh();
}

// Boot sequence (replaces Task 5 temporary boot)
MapManager.init();
loadData().then(() => RefreshManager.start());
```

- [ ] **Step 3: Verify auto-refresh**

Open browser. In the top bar:
- "Refreshing in 45s" countdown ticks down every second
- At 0, data reloads and counter resets to 45
- Manual Refresh button reloads data and resets countdown immediately
- Map viewport (zoom/center) is preserved across refreshes
- Previously open popups are closed on refresh

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: auto-refresh every 45s with countdown timer and manual refresh"
```

---

### Task 8: Leaflet Popup Dark Theme Overrides

**Files:**
- Modify: `index.html` — CSS overrides for Leaflet popup

- [ ] **Step 1: Add Leaflet popup CSS overrides**

```css
/* Leaflet popup dark theme */
.leaflet-popup-content-wrapper {
  background: #0f172a !important;
  border: 1px solid #334155 !important;
  border-radius: 8px !important;
  box-shadow: 0 4px 16px rgba(0,0,0,0.6) !important;
  color: var(--text) !important;
}

.leaflet-popup-tip {
  background: #0f172a !important;
}

.leaflet-popup-close-button {
  color: var(--muted) !important;
}

.leaflet-control-zoom a {
  background: var(--surface) !important;
  color: var(--muted) !important;
  border-color: var(--border) !important;
}

.leaflet-control-zoom a:hover {
  background: var(--bg) !important;
  color: var(--text) !important;
}

.leaflet-control-attribution {
  background: rgba(13,17,23,0.8) !important;
  color: var(--dim) !important;
}

.leaflet-control-attribution a { color: var(--dim) !important; }
```

- [ ] **Step 2: Verify popup appearance**

Click a marker. Popup should have dark background, light text, no white box.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: dark theme overrides for Leaflet popup and controls"
```

---

### Task 9: Polish + Responsive Check

**Files:**
- Modify: `index.html` — fade-in animation, final polish

- [ ] **Step 1: Add fade-in CSS for markers**

```css
@keyframes markerFadeIn {
  from { opacity: 0; transform: scale(0.5); }
  to   { opacity: 1; transform: scale(1); }
}

.leaflet-marker-icon {
  animation: markerFadeIn 0.3s ease-out;
}
```

- [ ] **Step 2: Add page title polish to topbar**

Ensure the title area has the back-link in the footer and attribution are styled correctly. Do a final visual pass:
- Title and badge sit nicely in the top bar on desktop
- Sidebar scrolls smoothly
- Count cards and filter pills are not clipped

- [ ] **Step 3: Test responsive layout at 768px**

In browser DevTools, set viewport to 768px wide. Verify:
- Sidebar stacks above the map
- Map is still visible and usable
- Incident list is scrollable
- No horizontal overflow

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "style: marker fade-in animation and responsive polish"
```

---

### Task 10: Deploy to S3

**Files:**
- No code changes — deployment only

- [ ] **Step 1: Create S3 bucket (run in WSL terminal)**

```bash
aws s3 mb s3://jimmy-traffic-dashboard --region us-east-1
```

- [ ] **Step 2: Enable static website hosting**

```bash
aws s3 website s3://jimmy-traffic-dashboard \
  --index-document index.html \
  --error-document index.html
```

- [ ] **Step 3: Set public read policy**

```bash
aws s3api put-bucket-policy \
  --bucket jimmy-traffic-dashboard \
  --policy '{
    "Version":"2012-10-17",
    "Statement":[{
      "Sid":"PublicRead",
      "Effect":"Allow",
      "Principal":"*",
      "Action":"s3:GetObject",
      "Resource":"arn:aws:s3:::jimmy-traffic-dashboard/*"
    }]
  }'
```

- [ ] **Step 4: Upload the file**

```bash
aws s3 cp /mnt/c/Users/jimmy/Desktop/ClaudeCode/traffic-dashboard/index.html \
  s3://jimmy-traffic-dashboard/index.html \
  --content-type "text/html"
```

- [ ] **Step 5: Verify live URL**

Open in browser:
```
http://jimmy-traffic-dashboard.s3-website-us-east-1.amazonaws.com
```
Expected: Full dashboard loads, DEMO MODE badge visible, markers appear.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "chore: deploy to S3 jimmy-traffic-dashboard"
```

---

### Task 11: Add as Card #6 on Projects Page

**Files:**
- Modify: WordPress Projects page (ID: 43) via REST API or WP Admin

- [ ] **Step 1: Get current Projects page content**

```bash
curl -s "https://jimmyhubbard2.cc/wp-json/wp/v2/pages/43" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['content']['rendered'])" | head -100
```

- [ ] **Step 2: Identify the "Coming Soon" card in the HTML**

Find the div that contains the "Coming Soon" placeholder and note its structure.

- [ ] **Step 3: Replace Coming Soon with Traffic Dashboard card**

The new card HTML (matching existing card pattern — use `div` + `onclick`, not `<a>` tags):

```html
<div class="project-card" onclick="window.open('http://jimmy-traffic-dashboard.s3-website-us-east-1.amazonaws.com','_blank')" style="cursor:pointer;">
  <div class="project-icon">🚦</div>
  <h3>Texas Traffic Dashboard</h3>
  <p>Live traffic incident map for Texas built with Leaflet.js and TxDOT 511 public data. Color-coded markers, real-time filtering, and auto-refresh.</p>
  <div class="project-tags">
    <span>Leaflet.js</span><span>JavaScript</span><span>AWS S3</span><span>REST API</span>
  </div>
</div>
```

Update the page via WP Admin Site Editor or REST API with Application Password.

- [ ] **Step 4: Verify Projects page**

Open `https://jimmyhubbard2.cc/projects/` — the 3×2 grid should now show all 6 cards with no Coming Soon slot.

- [ ] **Step 5: Update memory**

Update `/home/jimmy/.claude/projects/-mnt-c-Users-jimmy/memory/project_website.md` to add Traffic Dashboard as project #6.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: complete Texas Traffic Dashboard — deployed to S3 and added to portfolio"
```
