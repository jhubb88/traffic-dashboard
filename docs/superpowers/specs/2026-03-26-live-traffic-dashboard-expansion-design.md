# Live Traffic Dashboard — Multi-City Expansion

**Date:** 2026-03-26
**Status:** Approved

## Summary

Expand the Traffic Dashboard from Texas-only simulated data to 5 cities across 4 states. Rename the project from "Texas Traffic Dashboard" to "Live Traffic Dashboard". Update the map default view to show all 5 cities at once.

## Cities

| City | State | Lat | Lng | Key Roads |
|------|-------|-----|-----|-----------|
| Scottsdale | AZ | 33.4942 | -111.9261 | Loop 101, SR-51, SR-202, US-60, AZ-87 |
| Denver | CO | 39.7392 | -104.9903 | I-25, I-70, I-225, US-36, E-470, C-470 |
| Salt Lake City | UT | 40.7608 | -111.8910 | I-15, I-80, I-215, SR-201, SR-89 |
| Nashville | TN | 36.1627 | -86.7816 | I-24, I-40, I-65, SR-840, US-70, I-440 |
| Dallas/Fort Worth | TX | 32.7767 | -96.7970 | I-30, I-35E, I-20, SH-114, US-75, I-635 |

## Changes

### 1. Rename
- `<title>`: `Texas Traffic Dashboard — Jimmy Hubbard` → `Live Traffic Dashboard — Jimmy Hubbard`
- Topbar label: `Texas Traffic Dashboard` → `Live Traffic Dashboard`
- Footer: `Data: TxDOT 511 Public API` → `Data: Multi-State 511 APIs`

### 2. Map Default View
- Center: `[36.8, -102.8]` (geographic midpoint of all 5 cities)
- Zoom: `5` (was `7`) — fits all 5 cities in one view
- Constant name: `TEXAS_CENTER` → `MAP_CENTER`

### 3. Simulated Incident Pool
- Replace all 40 Texas-only incidents with 50 new incidents (10 per city)
- Each city gets a realistic mix: ~3 accidents, ~3 construction, ~2 hazards, ~2 closures
- Severities spread across major/moderate/minor
- No changes to the shuffle/active-set/refresh engine

### 4. No Logic Changes
The following are unchanged:
- `getActiveIncidents()` shuffle logic (seeds 25 of 50, rotates 2-4 per refresh)
- Filter pills, search, sidebar rendering, map marker rendering
- Data source cascade (TxDOT → SF511 → Demo)
- Refresh timer (45s interval)
