# Live Traffic Dashboard — Master Reference Document

*Operational reference + recruiter overview. Update when AWS resources change, after each phase, or when a key decision is revisited.*
*Last updated: 2026-05-14*

---

## 1. TL;DR

The Live Traffic Dashboard displays active traffic incidents across five US metro areas (Scottsdale AZ, Denver CO, Salt Lake City UT, Nashville TN, Dallas-Fort Worth TX) on an interactive Leaflet map. The frontend tries live 511 data sources (TxDOT then SF Bay 511), but both are CORS-blocked from a static host, so a built-in simulated dataset renders today. A Lambda proxy on the roadmap will unblock the live paths. Stack: single-file vanilla HTML/CSS/JS (~36 KB), Leaflet.js + OpenStreetMap tiles, hosted on CloudFront + S3.

---

## 2. Live demo & repository

| Field | Value |
|---|---|
| Live URL | https://traffic-dashboard.jimmyhubbard2.cc |
| Repository | https://github.com/jhubb88/traffic-dashboard |
| Status | Working end-to-end (visible source badge currently reads "DEMO — Simulated Data" — see Section 4 Open items and Future Enhancement E2) |
| Last deploy | 2026-05-10 (H1 audit closure) |
| CI trigger | Push to `main` (S3 sync + CloudFront invalidation; no Lambda, no smoke test — there is no backend) |

---

## 3. Architecture

The entire application is a single `index.html` file with inlined CSS and JavaScript. On load, `fetchIncidents()` tries TxDOT 511, then SF Bay 511, then falls through to the built-in `SimulatedData` module. The simulated pool seeds once at startup and rotates 2-4 incidents per refresh to mimic real-world churn. `MapManager` owns the Leaflet instance and marker layer; `SidebarManager` owns the incident list; both read from the same shared state object. A `setInterval` countdown drives auto-refresh every 300 seconds. Both live API paths are currently CORS-blocked from a static host — the simulated fallback is what renders for every visit today. The planned post-proxy architecture is committed to the repo as `diagrams/traffic-dashboard-future-architecture.svg` with a NOT-DEPLOYED banner inside the SVG itself.

### Stack

| Layer | Technology |
|---|---|
| Markup / style / logic | Vanilla HTML5, CSS3, JS (single `index.html`, ~870 lines, inlined styles + scripts) |
| Map library | Leaflet.js 1.9.4 (CDN-loaded) |
| Tile source | OpenStreetMap (public tile servers) |
| Data source cascade | TxDOT 511 → SF Bay 511 → built-in `SimulatedData` module |
| Hosting | S3 (`jimmy-traffic-dashboard`) + CloudFront (`E3H1V6C42HG9P1`) |
| TLS | ACM cert, TLSv1.2_2021 minimum at CloudFront |
| CI/CD | GitHub Actions on push to `main` — S3 sync + CloudFront invalidation (33-line workflow, no Lambda step) |
| Repo-only artifacts | `diagrams/` (current + future-state SVG), `docs/`, `.superpowers/` — all excluded from S3 sync |
| Fonts | Google Fonts (Inter) |
| License | MIT |

### Data flow

```
┌─────────┐  GET /        ┌────────────┐  GET *     ┌──────────────────┐
│ Browser │ ─────────────►│ CloudFront │ ─────────► │ S3 (frontend)    │
└────┬────┘               │ E3H1V...   │            │ jimmy-traffic-...│
     │                    └────────────┘            └──────────────────┘
     │
     │  index.html cascade (5-min auto-refresh):
     │
     ├─► TxDOT 511    https://traffic.511tx.org/api/v2/get/event
     │                 ✗ CORS-blocked from browser
     │
     ├─► SF Bay 511   https://api.511.org/traffic/events?api_key=...
     │                 ✗ CORS-blocked from browser
     │
     └─► SimulatedData  Built into index.html
                        50-incident pool / 25 active / rotates 2-4 per refresh
                        ★ Currently 100% of rendering
```

The planned future state (with a Lambda proxy in front of the live APIs, removing the CORS wall) is documented in `diagrams/traffic-dashboard-future-architecture.svg` with an in-SVG NOT-DEPLOYED banner. See Future Enhancement E2.

---

## 4. Current state

### Works end-to-end

- Interactive Leaflet map with custom SVG markers for five US metro areas; 25 active incidents at any time from a 50-incident pool that rotates 2-4 entries per refresh.
- Color-coded by incident type (accident, closure, hazard, construction) and severity. Exclusive-select filter pills with live counts.
- Linked map and sidebar: click a marker to filter the sidebar to that city and zoom in; click a sidebar card to zoom to the exact incident; Reset View restores continental US.
- 5-minute auto-refresh with a manual refresh button available.
- Mobile-responsive at ≤768 px viewport (stacked layout).
- CI auto-deploys on every push to `main`. Two-step workflow: S3 sync + CloudFront invalidation.

### Open items

- Live data path is currently always failing. Both TxDOT 511 and SF Bay 511 are CORS-blocked from a static host. The visible source badge reads "DEMO — Simulated Data" on every visit. Tracked as Future Enhancement E2.
- GitHub Actions still pinned to v4 majors (`actions/checkout@v4`, `aws-actions/configure-aws-credentials@v4`). Other portfolio projects bumped to v6 in May 2026; this repo missed the bump. Tracked as E1.
- SF511 API key embedded as a plaintext literal in `index.html` line 478. Free-tier public-developer key, lower impact than other portfolio key exposures. Tracked as E5.
- No CloudWatch alarms scoped to this project. Tracked as E3.
- `portfolio-user` CLI principal lacks `cloudwatch:GetMetricStatistics` and `ce:GetCostAndUsage` permissions. Tracked as E4.
- Frontend bucket has no lifecycle configuration. Noncurrent versions accumulate; low impact at this traffic level. Not promoted to its own enhancement entry.

---

## 5. Build history

### Phase 1 — Initial scaffold + docs (2026-04-14 → 2026-04-22)

Migrated from a monorepo into a standalone repository (commit `3b065e1`). First end-to-end working version: single-file vanilla HTML/CSS/JS, Leaflet.js + OpenStreetMap, the full data-cascade logic with built-in `SimulatedData`, and all 50 incidents seeded across the five cities. Favicons added (`096e835`). README + LICENSE + initial live demo link committed (`c90133b`). At end of phase: working static site, no CI, manually deployed.

### Phase 2 — CI/CD + subdomain wiring (2026-05-01 → 2026-05-08)

Deploy workflow added (`c36ba5b`). 33-line two-step pipeline — S3 sync (with excludes for `.git`, `.github`, `*.md`, `LICENSE`, `.gitignore`, `.superpowers/*`, `bucket-policy.json`, `diagrams/*`, `docs/*`) followed by CloudFront invalidation. No Lambda step (no backend exists). Subdomain switched from the raw CloudFront URL to `traffic-dashboard.jimmyhubbard2.cc` via cPanel CNAME → CloudFront (`6e92181`). The subdomain uses the full project name rather than an abbreviated form.

### Portfolio-wide infrastructure note — OAC migration (2026-05-07)

Frontend bucket `jimmy-traffic-dashboard` migrated to Origin Access Control as part of the portfolio-wide OAC standardization across all eight project distributions. No source-code commit in this repo (OAC is a CloudFront + bucket policy change, not a build-artifact change). See Key Decision 4.

### Phase 3 — H1 audit closure (2026-05-10, commit `d503e09`)

Self-audited the repo, scored findings by severity, fixed mediums in commit `d503e09`. Two medium-severity issues closed:

- **M1 — misleading documentation.** A SVG architecture diagram was named `traffic-dashboard-prod-architecture.svg` but depicted a Lambda proxy that wasn't deployed. A recruiter clicking through the repo could infer the production architecture included that proxy. Renamed to `traffic-dashboard-future-architecture.svg` and added a NOT-DEPLOYED banner inside the SVG itself, so the diagram is honest about its status.
- **M2 — stale infrastructure artifact.** A `bucket-policy.json` template lived in the repo from before the portfolio-wide OAC migration. After the migration, the in-repo template no longer matched the live bucket policy. Removed from the repo so what's checked in matches what's deployed.

---

## 6. Operational reference

### S3 bucket

| Bucket | Purpose | PAB | Encryption | Lifecycle | Project tag |
|---|---|---|---|---|---|
| jimmy-traffic-dashboard | Static frontend | all 4 ON | AES256 | (none) | traffic-dashboard |

### CloudFront

| Field | Value |
|---|---|
| Distribution ID | `E3H1V6C42HG9P1` |
| Status | Deployed |
| Aliases | `traffic-dashboard.jimmyhubbard2.cc` |
| CloudFront domain | `d1t5py05a4uugi.cloudfront.net` |
| Origin | S3 REST endpoint → `jimmy-traffic-dashboard.s3.us-east-1.amazonaws.com` (OAC-fronted) |
| OAC ID | `E28Q5IINAEOULO` |
| Viewer cert source | ACM |
| ACM cert ARN | `arn:aws:acm:us-east-1:603509861186:certificate/598151c8-e0e7-4b46-acf0-4da54e5bce38` |
| Min protocol version | TLSv1.2_2021 |
| Price class | PriceClass_100 (NA + EU) |
| Custom error 403 → | `/index.html` (200) |
| Custom error 404 → | `/index.html` (200) |
| Default root object | `index.html` |
| WAF / WebACL | none |
| Cache behaviors | (DefaultCacheBehavior only) |

### CloudWatch alarms

None currently scoped to this project. Tracked as Future Enhancement E3.

### Cost & utilization

`portfolio-user` CLI principal does not have `cloudwatch:GetMetricStatistics` or `ce:GetCostAndUsage` permissions. 30-day request counts and per-project cost are not queryable from the CLI right now. The `Project=traffic-dashboard` tag is present on every resource for Cost Explorer attribution, but reading the data requires either Console access or an IAM permission grant. Tracked as E4.

---

## 7. Key decisions

### 1. Vanilla HTML/CSS/JS in a single file, no framework, no build step

The complete application is one ~36 KB `index.html` with inlined `<style>` and `<script>`. Leaflet.js is the only external dependency, CDN-loaded at runtime. React, Vue, Svelte, or even a multi-file split with ES modules were all viable alternatives. A single-file vanilla approach is easy for a recruiter to read top-to-bottom in any editor, has zero supply-chain attack surface (no `npm install` step in CI), and deploys in seconds via S3 sync. The CI workflow is correspondingly 33 lines — no bundler, no toolchain, no `npm run build`. Trade-off: harder to maintain if the file grows past ~100 KB. Today's 36 KB is comfortable.

### 2. Data source cascade with simulated fallback

The initial implementation tried direct calls from the browser to TxDOT 511 and SF Bay 511. Both were CORS-blocked from a static host; neither API serves `Access-Control-Allow-Origin` headers permissive enough for a browser running on a different domain. Rather than block the project on the planned Lambda proxy work, I built `SimulatedData` into `index.html` (50 incidents, 25 active, 2-4 rotating per refresh) and added the cascade — `tryTxDOT()` → `trySF511()` → simulated — so the cascade exits cleanly to the fallback on every visit. The architecture lands in the same place as a designed-for-resilience cascade would have: live data when available, simulated data when not. The difference is the path: this one was added after the CORS wall was already known, not before.

### 3. Leaflet.js + OpenStreetMap (over Mapbox or Google Maps)

Leaflet is open-source, no API key required, no usage cost, no quota. OpenStreetMap tiles are free at the public-tile-server level for this kind of traffic. Mapbox would have added an API key in the client (this project already has too many of those — see E5) plus a usage-based pricing surface. Google Maps would have added per-load billing through Google Cloud. For a portfolio demo with five cities, ~25 markers visible at a time, and no expectation of high traffic, Leaflet + OSM is the right cost-and-complexity floor. Trade-off: fewer built-in features than Mapbox or Google Maps (no native traffic overlay, no native incident styling), but neither feature would have replaced the cascade logic anyway. The custom SVG markers and the cascade pattern are what make this project distinct, and Leaflet doesn't constrain either.

### 4. OAC over OAI for the static-site bucket

The frontend bucket is fronted by Origin Access Control rather than the older Origin Access Identity pattern. Bucket policy scoped to `cloudfront.amazonaws.com` service principal with a `SourceArn` condition pinned to distribution `E3H1V6C42HG9P1`; CloudFront can read, nothing else can. All four Public Access Block settings on. Static website hosting disabled (the bucket uses the S3 REST endpoint via OAC). SPA-style routing handled at CloudFront with `CustomErrorResponses` mapping 403 and 404 to `/index.html` with a 200 response. OAC is the AWS-recommended pattern since 2022: SigV4 signing, support for SSE-KMS bucket encryption (not used here but available), and a more auditable trust model than OAI's anonymous-reader-by-CF-only convention. Migrated 2026-05-07 as part of the portfolio-wide OAC standardization across all eight project distributions.

---

## 8. Future enhancements

### E1 — Bump GitHub Actions to v6 majors (priority: medium)

`.github/workflows/deploy.yml` still pins `actions/checkout@v4` and `aws-actions/configure-aws-credentials@v4`. These v4 actions run on Node.js 20, which is deprecated on the GitHub Actions runner. Two upcoming deadlines from GitHub's deprecation timeline:

- **2026-06-02** — Actions are forced to run on Node.js 24 by default. v4 actions whose internal JS is not Node-24-compatible may behave differently from this date.
- **2026-09-16** — Node.js 20 is removed from the runner entirely. v4 actions will fail to start from this date.

Resume-matcher bumped to v6 in `ebd2cee` (2026-05-12); log-analyzer in `6218f76` (2026-05-12). traffic-dashboard, ntcip-simulator, and FieldIQ's prewarm.yml are the remaining v4 holdouts in the portfolio. Recommended bump before 2026-06-02 to stay ahead of the force-default. Fix is a two-line edit per workflow file (`@v4` → `@v6` for both actions).

### E2 — Lambda proxy to enable live 511 data (priority: medium)

This is the project's most impactful future change. Today, the frontend's cascade always falls through to `SimulatedData` because both live 511 APIs are CORS-blocked from a static host. The visible source badge reads "DEMO — Simulated Data" on every visit, and the README explicitly acknowledges this as a roadmap item.

The fix: deploy a small Lambda + API Gateway pair that proxies requests to TxDOT 511 and SF Bay 511, returning the responses with `Access-Control-Allow-Origin` headers set to the dashboard's CloudFront origin. The frontend's `tryTxDOT()` and `trySF511()` functions get pointed at `/api/txdot` and `/api/sf511` on the same domain (via a CloudFront `/api/*` cache behavior), removing the CORS wall.

Downstream effects:

- The source badge flips between "● LIVE — TxDOT 511" and "● LIVE — SF Bay 511" depending on which feed responded.
- The SF511 API key moves into the Lambda environment, out of the client-side code (resolves E5).
- `diagrams/traffic-dashboard-future-architecture.svg` becomes the current-architecture diagram, and the NOT-DEPLOYED banner gets removed.

Cost estimate at portfolio traffic: a few cents per month in Lambda + API Gateway charges.

### E3 — CloudWatch alarm for CloudFront 5xx error rate (priority: medium)

Zero alarms exist for this project. The most meaningful metric for a static site is CloudFront 5xx error rate — an origin failure (S3 unreachable, OAC misconfigured, bucket deleted) would surface here. Suggested baseline: `5xxErrorRate >= 1% in 5 min` on distribution `E3H1V6C42HG9P1`. Less noise than alarming on raw error counts at low traffic. SNS topic with email subscription. One-time setup.

### E4 — Grant `portfolio-user` CLI observability reads (priority: low)

Same enhancement carried over from the other portfolio PROJECT_MASTERs. `cloudwatch:GetMetricStatistics` and `ce:GetCostAndUsage` both return AccessDenied for the `portfolio-user` IAM principal. Today this means scripted observability reads fall back to the AWS Console. Granting either inline or via attached managed policies (`CloudWatchReadOnlyAccess` + a Cost Explorer policy) unblocks observability across all eight portfolio projects at once.

### E5 — Move SF511 API key out of client-side code (priority: low)

The SF511 API key (511.org free-tier public-developer key) is embedded as a plaintext literal in `index.html` line 478. Same bug class as resume-matcher's `unafcros47` and log-analyzer's embedded `x-api-key`, with a meaningful difference: the SF511 free-tier key is functionally a low-rate-limit identifier rather than a paid credential, so the impact is lower. Tracked here for consistency with other portfolio docs. **Resolves automatically when E2 ships** — the Lambda proxy holds the key in its environment variables and the client doesn't need it.

---

## 9. Files & structure

```
traffic-dashboard/                     ← Lives on Windows Desktop
├── .github/
│   └── workflows/
│       └── deploy.yml                 ← CI/CD — S3 sync + CloudFront invalidation (33 lines)
├── diagrams/                          ← Repo-only; excluded from S3 sync
│   ├── traffic-dashboard-architecture.svg              ← Current state
│   └── traffic-dashboard-future-architecture.svg      ← Post-proxy plan, NOT-DEPLOYED banner
├── docs/                              ← Repo-only; excluded from S3 sync
├── .superpowers/                      ← Claude Code tooling artifact; excluded from S3 sync
├── LICENSE                            MIT
├── README.md                          Recruiter-facing summary (~77 lines)
├── index.html                         Complete app — HTML + CSS + JS in one file (~36 KB)
├── apple-touch-icon.png
├── favicon-16.png
├── favicon-32.png
├── favicon.ico
├── icon-192.png
└── icon-512.png
```
