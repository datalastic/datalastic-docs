# Datalastic API Documentation Site

## What this is

A professional developer documentation site for the Datalastic Maritime API, built as a **single `index.html` file** deployed on GitHub Pages at `docs.datalastic.com`.

Source reference: `API Reference - Datalastic.html` (the original datalastic.com/api-reference/ page, saved locally)

---

## Deployment

- **Live URL:** https://docs.datalastic.com/
- **GitHub repo:** https://github.com/datalastic/datalastic-docs (main branch, root)
- **GitHub Pages:** enabled — source: main branch / root folder, custom domain: `docs.datalastic.com`, Enforce HTTPS: on
- **CNAME file:** present in repo — contains `docs.datalastic.com`
- **SiteGround DNS:** CNAME record `docs` → `datalastic.github.io` (added in Site Tools → Domain → DNS Zone Editor)
- **GoDaddy:** domain registrar only — nameservers point to SiteGround, no DNS changes needed there
- **No build step required** — open `index.html` directly in a browser to preview

### How to push updates
Git is configured globally as `Datalastic Maritime Data API` / `87026634+datalastic@users.noreply.github.com`. The gh CLI has both `datalastic` (active) and `jsoncargo` accounts. From this directory, always commit and push as datalastic — no special steps needed.

```bash
git add index.html
git commit -m "your message"
git push
```

Only `index.html`, `CNAME`, and `Datalastic_logo.png` are deployed. Everything else (CLAUDE.md, API Reference source files) stays local only.

---

## CDN dependencies

- Google Fonts: Inconsolata + Khula
- highlight.js 11.9.0 from cdnjs — **full bundle only** (`highlight.min.js`)
- highlight.js CSS theme: `atom-one-dark.min.css` from cdnjs

---

## Architecture

Single self-contained `index.html` (~120KB). All CSS and JS are inline. No frameworks, no npm, no build tools.

### Key data structures (in the JS)

- **`NAV`** — navigation group/item definitions. Add-on items have entries in `ADDON_NAV_IDS` set which auto-applies ➕ styling.
- **`ENDPOINTS`** — array of endpoint config objects. Each has: `id`, `method`, `base`, `path`, `title`, `desc`, `params`, `attrs`, `response`, `tryParams`, `examples` (optional override), `addon` (boolean).
- **`STATIC_SECTIONS`** — array of hand-authored HTML sections (Introduction, Authentication, Credit System, etc.)
- **`renderEndpoint(ep)`** — generates the full endpoint card HTML from an ENDPOINTS config object. Uses `tryItContent()` internally.
- **`tryItContent(key, ep)`** — generates the inline (non-collapsible) Try It form HTML. Only called for GET endpoints.
- **`buildNav()`** — renders the sidebar nav. No in-group dividers; group separation is CSS-only via `.nav-group + .nav-group`.
- **`ADDON_NAV_IDS`** — Set of endpoint IDs that are add-ons. Drives `addon-item` CSS class and ➕ pip in nav links. No longer inserts dividers.
- **`bulkReportSection(id, title, type, cost, desc, contains, body, addon=false)`** — helper that generates async report section HTML. Pass `addon=true` for Add-On reports to render the orange callout banner.

### Tab system (endpoint cards)
- `.ep-tabbar` — flex tab bar with bottom border (Parameters / Code Examples / Response / Try It ⚡)
- `.ep-tab[data-eptab]` — tab buttons; `data-eptab` values: `params`, `code`, `response`, `tryit`
- `.ep-tab.active` — active tab has purple underline
- `.ep-pane[data-eptabpane]` — tab panels, `display:none` by default
- `.ep-pane.active` — visible panel, `display:block`
- `.tryit-inline[data-endpoint]` — Try It form container inside the `tryit` pane; `data-endpoint` holds the endpoint key
- Tab switching is handled in the `[data-eptab]` branch of the body click event delegation
- `sendTry()` finds the panel via `btn.closest('.tryit-inline')` and sends `x-api-key` as a fetch header

### Base URLs used
- Core endpoints: `https://api.datalastic.com/api/v0/` (constant `BASE_V0`)
- Extended endpoints (SAT-E, Sea Routes): `https://api.datalastic.com/api/ext/` (constant `BASE_EXT`)
- Maritime report data: `https://api.datalastic.com/api/maritime_reports/` (constant `BASE_MR`)

---

## Design system

| Token | Value |
|-------|-------|
| `--color-dark` | `#0B0A18` |
| `--color-purple` | `#6046AF` |
| `--color-cyan` | `#92DDEA` |
| `--color-pink` | `#FFA5D8` |
| `--color-soft-purple` | `#BE9DDF` |
| `--color-blue` | `#7EB8DA` |
| `--color-bg` | `#ffffff` |
| `--color-surface` | `#f7f7fb` |
| `--color-border` | `#ebebf5` |

- **Error codes (correct):** 400, 401, 402 Payment Required (credits exhausted), 404, 429 Too Many Requests (rate limit only), 500. There is NO second 429 for monthly quota — that's 402.
- **Headings/code:** Inconsolata
- **Body/prose:** Khula
- **Buttons:** border-radius 16px
- **Add-on accent:** `#e65100` (orange), background `#fff8f0`, border `#ffcc80`
- **Nav group separator:** `.nav-group + .nav-group { border-top: 1px solid #ffcc80; padding-top: 4px; }` — orange line between every sidebar group, CSS-only, no JS divider elements

---

## Plan structure (important — get this right)

There are **6 plans** in two tiers:

| Base plans | Add-Ons plans |
|------------|---------------|
| Starter | Starter + Add-Ons |
| Experimenter | Experimenter + Add-Ons |
| Developer Pro | Developer Pro + Add-Ons |

**Key facts:**
- Add-On endpoints are **not purchasable separately** — the user must be on an Add-Ons plan tier.
- To get Add-Ons, users must **upgrade/switch** to the corresponding +Add-Ons plan (e.g. Starter → Starter+Add-Ons).
- The **API key stays the same** — it automatically unlocks Add-On endpoints when the plan includes them.
- Never describe Add-Ons as "purchasable on top of any base plan" — they require a specific plan tier.

---

## SEO

All SEO signals are set in the `<head>`:

- **Title:** `Datalastic API Documentation: Vessel Tracking, AIS & Maritime Data`
- **Meta description:** `Datalastic API Documentation for the maritime API. Query live AIS vessel positions, historical ship data, ports, and sea routes across 20 endpoints with clean JSON responses and global coverage.`
- **Canonical:** `https://docs.datalastic.com/`
- **Robots:** `index, follow`
- **Open Graph:** og:type, og:title, og:description, og:url, og:site_name, og:image
- **Twitter Card:** summary card with title, description, image
- **og:image / twitter:image:** `https://docs.datalastic.com/Datalastic_logo.png`
- **Favicon:** `<link rel="icon" type="image/png" href="Datalastic_logo.png">`
- **JSON-LD:** TechArticle schema with name, description, url, author (Datalastic org)
- **h1:** `Datalastic API Documentation` (exact primary keyword)

### Writing style rules (enforced)
- No em dashes (— or &mdash;) anywhere in user-visible text
- No AI buzzwords: "robust", "seamlessly", "comprehensive", "powerful", "cutting-edge", "leverage", "utilize", "streamline", "unlock", "harness", "elevate", "transform", "revolutionize"
- No filler openers: "Whether you're...", "From X to Y", "In today's..."
- Keep it direct and human — short sentences, active voice

---

## What's been built

### Layout
- Three-column desktop: 260px fixed sidebar / 780px main content / 200px sticky right rail
- Mobile: hamburger drawer sidebar, single column, right rail hidden
- IntersectionObserver drives active nav highlight and right-rail sub-section tracking (debounced 250ms)

### Sections implemented (all complete)

**Getting Started**
- Introduction — hero banner, 3 stats (1M vessels / 22K ports / Global), plan overview cards (Data Feed vs ➕ Add-Ons)
- Authentication — recommends `x-api-key` header as primary method; `?api-key=` query param documented as convenience alternative; all 3 base URLs documented
- Account & Usage — `/stat` endpoint: check API key status and monitor credit consumption
- Quickstart — 3-step guide
- Rate Limits & Credits — plan tiers + rate limit info

**Core Concepts**
- Credit System — full cost table split into Data Feed / ➕ Add-Ons sections
- Async Reports — full submit→poll→download lifecycle with cURL + Python examples
- Field Reference — searchable table of all AIS fields
- Vessel Types — live-filtered tag list (main types + subtypes)
- Error Codes — 6 entries: 400, 401, 402 Payment Required, 404, 429, 500

**API Endpoints (all 20, full cards)**

Core Data Feed:
1. Basic Vessel Tracking — GET /vessel
2. Pro Vessel Tracking — GET /vessel_pro
3. Bulk Vessel Tracking — GET /vessel_bulk
4. Area Traffic — GET /vessel_inradius
5. Vessel History — GET /vessel_history
6. Historical Area Scan — POST /report (inradius_history)
7. Vessel Info — GET /vessel_info
8. Vessel Finder — GET /vessel_find
9. Port Finder — GET /port_find
10. Port Info — GET /port (params: uuid, unlocode only — name/lat/lon/radius removed)

➕ Add-Ons:
11. SAT-E (Satellite Estimator) — GET /vessel_pro_est (base: /api/ext/)
12. Sea Routes — GET /route (base: /api/ext/)
13. Dry Dock Dates — GET /dry_dock_dates
14. Casualties — GET /casualty
15. Inspections & Detentions — GET /inspections
16. Sales & Demolitions — GET /spd
17. Ownership — GET /ownership
18. Classification Society — GET /class_society
19. Vessel Engines — GET /engine
20. Maritime Companies — GET /companies

**Bulk Reports (all 12 sections)**

Core Data Feed:
- Vessel List (5,000 credits)
- Port List (500 credits)
- Historical Area Scan (inradius_history)
- API Usage Report (free)

➕ Add-On Reports (all use `bulkReportSection(..., addon=true)`):
- Dry Dock Dates — `report_type: "dry_dock_dates"`
- Ship Casualties — `report_type: "casualty"` (supports from/to date params)
- Ship Inspections — `report_type: "inspections"` (supports from/to date params)
- Sales & Demolitions — `report_type: "sales_purchase_demolitions"` (note: different from endpoint ID `spd`)
- Vessel Ownership — `report_type: "ownership"`
- Classification Society — `report_type: "class_society"`
- Vessel Engines — `report_type: "engine"`
- Maritime Companies — `report_type: "companies"`

**Port Terminals (static section, not an endpoint card)**
- Nav ID: `port_terminals`, title: "Port Terminals" — appears in Ports & Routes group between Port Info and Sea Routes
- Renders a static section (via `staticSection()`) with `GET /api/v0/port` URL bar, cyan callout linking to Port Info, terminal fields table, and example response snippet
- Not in `ENDPOINTS` — uses the same `/port` endpoint as Port Info; the section explains terminals are part of the port response when present, not all ports have them

**SDKs & Libraries** — Python, Node.js, Go — all "Coming Soon" with placeholder GitHub links

### Per-endpoint card features
Each endpoint card has:
1. Method badge (GET purple / POST cyan) + URL bar with copy button
2. Description
3. Tab bar: **Parameters / Code Examples / Response / Try It ⚡** (Try It tab only on GET endpoints)
4. Parameters tab — params table (required badges, type pills)
5. Code Examples tab — 7-language switcher: cURL, Python, Node.js, Go, Ruby, PHP, Java — syntax highlighted via highlight.js; tabs have full ARIA (`role="tablist"`, `role="tab"`, `aria-selected`)
6. Response tab — collapsible attributes tree + JSON response example
7. Try It tab (GET only) — fetch()-based live tester, API key persisted in localStorage, inline (no collapsible wrapper); amber callout warns that the request is live and deducts credits. `sendTry()` sends the key as `x-api-key` header (not query param)

Add-on cards additionally have:
- `➕ Add-On` badge pill on the card header
- Orange callout banner explaining the Add-Ons plan tier requirement

### Global features
- Live search (Cmd/Ctrl+K) — filters sidebar nav; shows "No results" when nothing matches
- Copy buttons: "Copied!" on success, "Failed" on error or empty content — 2-second feedback
- Smooth scroll with 80px offset
- localStorage-persisted API key synced across all Try It panels (guarded against SecurityError in restricted contexts)
- `navigation_status` field value is `"Under way using engine"` (lowercase w — this is correct per AIS standard)
- IntersectionObserver URL hash updates are debounced 250ms and wrapped in try/catch against browser rate-limit SecurityError
- Right rail hides with `display:none` (not `visibility:hidden`) to avoid ghost column
- Mobile sidebar has keyboard focus trap (Tab/Shift+Tab wrap, Escape closes)

### CTAs
- Sidebar logo: links to `https://datalastic.com` (opens in new tab)
- Introduction: hero banner → https://datalastic.com/pricing/
- Authentication: "Get API Key" button
- Sidebar bottom: "Get API Key →" always visible
- After every endpoint: "Don't have an API key yet? Get one →"
- Note: **"Free" was intentionally removed** — CTAs say "Get API Key" not "Get Free API Key" (Datalastic has no free tier)
- Logo: `Datalastic_logo.png` embedded as base64 data URL in `.brand` (icon + "datalastic" text)

---

## Pending / known gaps

### SDK GitHub links
All three SDK links point to placeholder GitHub URLs that don't exist yet:
- `https://github.com/datalastic/datalastic-python`
- `https://github.com/datalastic/datalastic-node`
- `https://github.com/datalastic/datalastic-go`

**These repo names are also squattable on PyPI and npm.** Register and create all three repos (even empty) and claim the package names on PyPI and npm as soon as possible — the docs are live and indexed.

### Security items to address
These were flagged in a security audit and have not yet been fixed:
1. **Logo PNG XMP metadata** — `Datalastic_logo.png` contains an employee name and Facebook ad attribution IDs in embedded XMP metadata. Strip with `exiftool -all= Datalastic_logo.png` before re-embedding.
2. **No SRI hashes** — the cdnjs `highlight.min.js` and CSS have no `integrity=` attribute. A compromised CDN could exfiltrate API keys from localStorage.
3. **Real report UUID in examples** — verify `4ddc14e0-9059-9b86-1156-8efd93f0cfe6` is not a real accessible report at `https://report.datalastic.com/dl/`. Replace with a clearly synthetic UUID if so.
4. **Real contact details in sample responses** — example JSON contains real company emails, phone numbers, and a named individual's LinkedIn URL. Replace with fictional placeholders.

---

## Critical gotchas — read before editing

### `tryItPanel` does not exist
The old collapsible Try It panel function (`tryItPanel`) was replaced by `tryItContent()` when Try It became a tab. **`tryItPanel` is gone.** Any call to it anywhere in the file will throw a `ReferenceError` and crash the entire page (no content renders at all). Never re-add `tryItPanel` calls.

### POST endpoints never get a Try It tab
The `renderEndpoint()` function checks `isGet = ep.method !== "POST"`. If POST, the tabbar has three tabs only (Parameters, Code Examples, Response). Do not add Try It to POST sections.

### Logo is embedded AND deployed
`Datalastic_logo.png` is used in two ways:
1. **Embedded as base64** in the `<img>` tag inside `.brand` (sidebar logo)
2. **Deployed as a file** in the repo — referenced by `<link rel="icon">` (favicon) and `og:image`

If the logo changes, update both: re-embed the base64 in the `<img>` tag AND push the new PNG file to the repo.

To re-embed:
```python
import base64
with open('Datalastic_logo.png','rb') as f: print('data:image/png;base64,'+base64.b64encode(f.read()).decode())
```

### Error codes: 402 not 429 for quota
There is exactly ONE `429` response code — rate limit (600 req/min). Credits exhausted returns `402 Payment Required`. There is no second 429.

### highlight.js: full bundle only
Use `highlight.min.js` (the full bundle) from cdnjs. **Do not switch to `core.min.js` + individual language files** — `core.min.js` does not exist on cdnjs (returns 404), which silently breaks all syntax highlighting. The full bundle is the only correct approach.

### highlight.js on hidden tab panes
`hljs.highlightAll()` is called once at DOMContentLoaded and processes all `<pre><code>` blocks including those in hidden `.ep-pane` divs. This works fine — hljs doesn't skip hidden elements.

### Authentication: header is primary, query param is secondary
All code examples use `x-api-key: YOUR_API_KEY` as a request header. The `?api-key=` query parameter still works and is documented in the Authentication section as a convenience alternative, but it is NOT the recommended approach and must not appear in code examples. `genGetExamples()` builds URLs without `api-key` in the query string (uses `urlNoKey`).

### Add-Ons plan messaging
Never say Add-Ons are "purchasable on top of any base plan." The correct model: users must be on an Add-Ons plan tier (Starter+Add-Ons, Experimenter+Add-Ons, or Developer Pro+Add-Ons).

---

## How to make changes

**Add a new endpoint:**
1. Add a config object to the `ENDPOINTS` array with: `id`, `method`, `base`, `path`, `title`, `desc`, `params[]`, `attrs[]`, `response` (JSON string), `tryParams[]`
2. If it's an add-on, add `addon: true` and add the `id` to `ADDON_NAV_IDS`
3. Add a nav entry in the `NAV` array under the appropriate group

**Add a static section:**
Add a new `case "id":` in the `staticSection()` switch function returning a `<section>` element. Add the id to `STATIC_IDS` set and add a nav entry in `NAV`.

**Add an Add-On bulk report section:**
Call `bulkReportSection(id, title, report_type, cost, desc, contains, extraBody, true)` inside `staticSection()`. Add the id to `STATIC_IDS`, `ADDON_NAV_IDS`, and `NAV` under the "Add-On Reports" group.

**Change the credit cost table:**
Find the credit-system static section HTML. The table has two `credit-section-head` rows splitting Data Feed from ➕ Add-Ons. Use `colspan="2"` on section header rows.

**Change branding colors:**
Update the CSS variables in the `:root` block at the top of the `<style>` tag.

---

## Files in this directory

| File | Purpose |
|------|---------|
| `index.html` | The documentation site — deployed |
| `CNAME` | GitHub Pages custom domain config — deployed |
| `Datalastic_logo.png` | Logo file — deployed (used as favicon + og:image); also embedded as base64 inside index.html for the sidebar |
| `CLAUDE.md` | This file — local only, not deployed |
| `API Reference - Datalastic.html` | Original source page from datalastic.com/api-reference/ — reference only, not deployed |
| `API Reference - Datalastic_files/` | Assets from the saved original page — reference only, not deployed |
