# Stoten Cost Intelligence v2 — Claude Code Handoff

Comprehensive handoff document for Claude Code / Codex rebuild of the Stoten Cost Intelligence prototype. Covers everything built, what works, outstanding bugs, attempted fixes, data model, design system, and next steps.

## Project Overview

## What This Is

**Stoten Cost Intelligence** is an interactive construction cost management dashboard built for Stoten's construction management team. It is a fully self-contained single HTML file (~120KB) with no backend, no database, and no external dependencies except Chart.js and Google Fonts loaded from CDN.

It was built as a **working prototype / functional spec** to show Stoten's CMs what a production tool would look and feel like. All data is dummy/illustrative. The immediate goal was CM review and feedback before an enterprise rebuild.

## Current Status
- **Prototype is functionally complete** — all 11 panels built, all dummy data wired
- **Outstanding bug:** Three panels (AI Query, Data Foundation, New Estimate) display blank on navigation — scroll position doesn't reset to top after switching from a longer panel. This is the primary reason for handoff to Claude Code.
- **File locations:**
  - GitHub (private): `https://github.com/djegan97-oss/stoten-cost-intelligence` (branch: `main`, file: `prototype/stoten-cost-intelligence-v2.html`)
  - Live public URL: `https://pub.hyperagent.com/p/7kaTQ7s8VAwHvnET2R9H1A`
  - Artifact v16 (Hyperagent platform)

## Context
- Built for **Stoten** — a CRE development company. The CMs are Jack and one other.
- This is a **demo prototype**, not a production tool. No backend exists.
- When this goes live, the stack will be: **Next.js + Supabase + Railway**
- GitHub account: `djegan97-oss` (all repos private)
- The operator is **not a developer** — do not assume CLI familiarity in any communications back to him

## The Outstanding Bug — Fix This First

## Symptom
When navigating TO any of these three panels — **AI Query**, **Data Foundation**, **New Estimate** — after having scrolled down on a longer panel (e.g. Portfolio View with its long project table), the page appears blank. Scrolling up reveals the content was there all along, hidden above the viewport.

## Root Cause (best diagnosis)
The page uses natural scroll (the document itself scrolls). When switching dashboards, the scroll position from the previous panel is not being reset. Shorter panels (AI Query, Data Foundation, New Estimate) have content only in the first ~400-600px. If the user scrolled Portfolio View down to ~800px and then clicks AI Query, the page is still scrolled to 800px — past where AI Query's content ends — so only blank space is visible.

## What Has Already Been Tried (16 versions)
All of these approaches were tried and failed to fix the issue:

1. **Removing `max-width` constraints** on those three panels — those were causing a different blank panel issue, fixed
2. **Switching from viewport-lock layout** (`height:100vh; overflow:hidden` on `.app`, `overflow-y:auto` on `.main`) to natural page scroll — changed the behavior but didn't fix the core issue
3. **`window.scrollTo(0,0)`** in the navigation handler
4. **`window.parent.scrollTo(0,0)`** — tried to scroll the parent frame (the Hyperagent wrapper)
5. **Removing `min-height: 100vh`** from `.app`
6. **`document.querySelector('.main').scrollTop = 0`** — scrolling the internal `.main` container
7. **`window.scrollTo(0,0) + document.documentElement.scrollTop = 0 + document.body.scrollTop = 0`** — all three methods simultaneously
8. **`requestAnimationFrame(() => { window.scrollTo(0,0); ... })`** — deferred scroll to fire after DOM render
9. **`history.scrollRestoration = 'manual'`** — prevent browser from restoring stale scroll
10. **`position: fixed` sidebar + `margin-left: 220px` on `.main`** — completely different layout model to decouple sidebar scroll from content scroll

None worked. The developer should diagnose fresh from the GitHub source.

## Current Layout CSS
```css
html, body { font-family: 'Inter', system-ui; font-size: 14px; background: #f9fafb; }
.app { display: block; }
.sidebar { position: fixed; top: 0; left: 0; width: 220px; height: 100vh; z-index: 100; overflow-y: auto; }
.main { margin-left: 220px; min-height: 100vh; background: #f9fafb; }
.dashboard { display: none; }
.dashboard.active { display: block; }
.dash-body { padding: 24px 28px; }
```

## Current Navigation JS
```javascript
document.querySelectorAll('.sb-item[data-dash]').forEach(item => {
  item.addEventListener('click', () => {
    const id = item.dataset.dash;
    document.querySelectorAll('.sb-item[data-dash]').forEach(i => i.classList.remove('active'));
    item.classList.add('active');
    document.querySelectorAll('.dashboard').forEach(d => d.classList.remove('active'));
    document.getElementById('dash-' + id).classList.add('active');
    requestAnimationFrame(() => {
      window.scrollTo(0,0);
      document.documentElement.scrollTop = 0;
      document.body.scrollTop = 0;
    });
    if (id === 'regional') renderComparison();
    if (id === 'trade') renderTradeCharts();
  });
});
```

## Key Constraint
The file is served by Hyperagent's CDN at `pub.hyperagent.com/p/...`. Hyperagent **injects a script wrapper** before our HTML in the served file. The injected script includes error reporting and a custom script loader. This may be interfering with scroll behavior. The developer should test both with and without this injection by serving the raw HTML locally.

## All 11 Panels — What Each Does

The app has a dark sidebar (220px wide) with navigation. Clicking any sidebar item shows that panel and hides all others.

## Sidebar Structure
```
DASHBOARDS
  ▦ Portfolio View     [data-dash="portfolio"]  — default/active on load
  ⊞ Regional Compare   [data-dash="regional"]
  ◈ Trade Pricing      [data-dash="trade"]

DATA
  ⊟ Cost Database      [data-dash="costdb"]
  ⊡ Project History    [data-dash="history"]
  ⬆ Data Ingest        [data-dash="ingest"]
  ⇌ Bid Compare        [data-dash="bidcompare"]
  ⬡ Data Foundation    [data-dash="foundation"]

INTELLIGENCE
  ✦ AI Query           [data-dash="aiquery"]
  ◎ New Estimate       [data-dash="estimate"]
```

Plus a dynamic panel:
  **Project Detail**    [id="dash-project"] — no sidebar item; accessed by clicking any project row

---

## 1. Portfolio View (`#dash-portfolio`)
National portfolio overview. Shows 4 KPI cards (projects, data points, avg variance, regions), a Budget vs. Actual grouped bar chart by project, a region donut chart, and a full 10-project table. Clicking any table row opens Project Detail.

## 2. Regional Compare (`#dash-regional`)
Side-by-side project comparison tool. Has:
- Region filter tabs (All / Southwest / South Central / Southeast / Midwest) that filter the project dropdowns
- Two project selectors (A and B) — any two of 10 projects
- Default: Katherines Crossing vs. Wolf Rd Distribution
- Comparison table: Project Profile, Building Specs, Cost Summary, Normalized Metrics (Cost/SF, Height-Adjusted Cost/SF, Normalized to 100K SF, Cost/Dock Door, Cost/Site Acre)
- Normalization notes panel explaining the methodology

## 3. Trade Pricing (`#dash-trade`)
Commodity pricing trends. Has:
- 4 sparkline cards: Dock Doors ($/door), Tilt-Up Panel ($/SF), Structural Steel ($/ton), Site Work ($/SF)
- Region filter buttons (All Regions / Southwest / South Central / Southeast / Midwest) — regional multipliers applied
- Tabbed table section: "All Commodities" (aggregate QoQ) + 4 commodity tabs each showing all 4 regions side by side with period-over-period deltas
- 9 quarters of data: Q1 2023 – Q1 2025

## 4. Cost Database (`#dash-costdb`) ← [BROKEN: blank on nav]
Full cost breakdown table. 10 projects × 10 cost categories (Concrete/Foundations, Structural Steel, Tilt-Up Panels, Roofing, Mechanical, Electrical, Plumbing/FP, Dock Equipment, Site Work/Civil, GC/Fee) with $M values and $/SF per category. Portfolio average footer row. Rows are clickable — open Project Detail.

**Note: Cost Database was NOT reported as broken by the operator. Only AI Query, Data Foundation, and New Estimate are the confirmed broken panels.**

## 5. Project History (`#dash-history`)
Card grid of all 10 projects sorted newest-first. Each card shows: name, type, market, GC, year, SF, actual cost, cost/SF, clear height, budget variance. Clicking a card opens Project Detail.

## 6. Data Ingest (`#dash-ingest`)
Illustration of future upload workflow. Has amber disclaimer banner ("not yet connected"). Upload drag-drop area (non-functional), manual entry form (non-functional), recent ingests log, database stats. Buttons labeled "Coming Soon" show an alert when clicked.

## 7. Bid Compare (`#dash-bidcompare`)
GC bid leveling tool. 3-project dropdown (Katherines Crossing / Wolf Rd / Nashville Industrial Park). Switching the dropdown re-renders the bid table via `renderBidTable(key)`. Each project has 3 GC bids with 10 scope line items, TOTAL BID row, and Leveled Total row.

## 8. Data Foundation (`#dash-foundation`) ← [BROKEN: blank on nav]
Schema/methodology documentation. Two-column grid: Project-Level Fields table (12 fields) + Cost Category Fields table (10 categories with scope descriptions). Normalization Standards. Data Collection Sources (3 cards: GC Closeout, AIA G702, Manual Entry). All static HTML.

## 9. AI Query (`#dash-aiquery`) ← [BROKEN: blank on nav]
Demo AI query interface. Has amber disclaimer banner ("no live LLM connected"). Query textarea + Run Query button. 5 pre-seeded sample query buttons (hardcoded responses in `AI_RESPONSES` object). Recent Queries section (3 static items). `runAIQuery()` matches input to `AI_RESPONSES` or returns generic fallback.

## 10. New Estimate (`#dash-estimate`) ← [BROKEN: blank on nav]
Illustration of future estimate generator. Parameter form (8 fields: name, type, market, region, SF, clear height, dock doors, site acres). `runEstimate()` shows a static cost range output (Low/Mid/High) with category breakdown. No real computation.

## 11. Project Detail (`#dash-project`) — Dynamic
Opened by clicking any project in Portfolio, Cost Database, or Project History. Populated dynamically by `openProject(projectId, fromDash)`. Shows:
- 5 KPI cards (budget, actual, variance %, all-in $/SF, cost/dock door)
- Full-width cost breakdown table (10 categories × budget/$/SF/actual/$/SF/Δ$M/Δ%/% of total)
- Horizontal grouped bar chart (Budget vs Actual by category)
- Project specs strip (GC, market, region, SF, clear height, site acres)
- Portfolio ranking note (ranks by $/SF and variance magnitude)
- "← Back" button → returns to origin panel
- "⇌ Compare to Another Project" button → goes to Regional Compare with this project as Project A, highlights Project B dropdown

## Data Model

## 10 Projects

All data is dummy/illustrative. Stored in `const PROJECTS = [...]` array in JS.

```javascript
{
  id: 1-10,
  name: string,
  type: 'Industrial' | 'Flex' | 'Cold Storage' | 'Mixed-Use',
  region: 'Southwest' | 'South Central' | 'Southeast' | 'Midwest',
  market: string,  // e.g. "Chicago, IL"
  gc: string,      // General Contractor name
  sf: number,      // Building square footage
  clear_ht: number, // Clear height in feet
  dock_doors: number,
  site_acres: number,
  budget: number,  // Total budget in dollars
  actual: number,  // Total actual cost in dollars
  year: number,
  cost_cats: {     // All 10 values sum to actual cost, in $M
    foundations: number,
    steel: number,
    panels: number,
    roofing: number,
    mechanical: number,
    electrical: number,
    plumbing_fp: number,
    dock_equip: number,
    site_civil: number,
    gc_fee: number
  }
}
```

### The 10 Projects
| # | Name | Type | Region | Market | GC | SF | Budget | Actual |
|---|------|------|--------|--------|-----|-----|--------|--------|
| 1 | Phoenix Distribution Ctr | Industrial | Southwest | Phoenix, AZ | Hensel Phelps | 148K | $18.2M | $19.1M |
| 2 | Austin Industrial Flex | Flex | South Central | Austin, TX | Swinerton | 85K | $11.4M | $10.9M |
| 3 | Denver Cold Storage | Cold Storage | Southwest | Denver, CO | JE Dunn | 64K | $14.2M | $15.8M |
| 4 | Dallas Last Mile | Industrial | South Central | Dallas, TX | Austin Industries | 122K | $14.1M | $13.7M |
| 5 | Nashville Industrial Park | Industrial | Southeast | Nashville, TN | Skanska USA | 96K | $12.8M | $13.3M |
| 6 | Atlanta Flex Center | Flex | Southeast | Atlanta, GA | DPR Construction | 71K | $9.2M | $9.6M |
| 7 | Charlotte Mixed-Use | Mixed-Use | Southeast | Charlotte, NC | Balfour Beatty | 52K | $10.4M | $11.1M |
| 8 | San Antonio Warehouse | Industrial | South Central | San Antonio, TX | Kiewit | 110K | $13.1M | $12.8M |
| 9 | Katherines Crossing | Industrial | Midwest | Chicago, IL | Power Construction | 135K | $16.4M | $17.2M |
| 10 | Wolf Rd Distribution | Industrial | Midwest | Chicago, IL | Pepper Construction | 118K | $15.1M | $15.8M |

## Trade Pricing Data

Stored in `const TRADE_BASE` and `const REGION_ADJ`. 9 quarters (Q1 2023 – Q1 2025).

```javascript
const QUARTERS = ['Q1 23','Q2 23','Q3 23','Q4 23','Q1 24','Q2 24','Q3 24','Q4 24','Q1 25'];

const TRADE_BASE = {
  dock:  [4800, 5100, 5350, 5200, 5400, 5650, 5800, 6050, 6200],  // $/door
  tilt:  [42.0, 43.5, 44.0, 43.0, 44.5, 46.0, 47.5, 49.0, 50.5], // $/SF
  steel: [2800, 2950, 3100, 2900, 3050, 3200, 3350, 3500, 3650],  // $/ton
  site:  [8.50, 8.80, 9.10, 8.90, 9.30, 9.60, 9.90, 10.20, 10.50] // $/SF
};

// REGION_ADJ: per-quarter multipliers per region per commodity
// Midwest runs ~5-12% above national; South Central 6-9% below
// See REGION_ADJ constant in the source for full values
```

## Bid Data

Stored in `const BID_DATA` with keys: `'katherines'`, `'wolfrd'`, `'nashville'`.

Each entry has: `title`, `sub`, `gcs` (3 GC names), `rows` (10 line items × [scope, bidA, bidB, bidC, notes]), `totals`, `leveled`, `spread`, `levelNote`.

## Normalization Methodology

Used in Regional Compare and Project Detail:
- **Cost/SF**: actual / building_sf
- **Height-Adjusted Cost/SF**: (actual/sf) - (max(0, clear_ht - 28) × 0.45)
- **Normalized to 100K SF**: (actual/sf) × 100000
- **Cost/Dock Door**: actual / dock_doors
- **Cost/Site Acre**: actual / site_acres
- **Per-category budget estimate**: actual × (total_budget / total_actual) — proportional split

## Design System

## Key Design Principles

1. **Do NOT make it look AI-generated.** No purple/blue gradients, no glowing effects, no icon grids, no uniform eyebrow labels on every element.
2. **Data-dense, not visual-heavy.** Tables and numbers are the primary UI. Charts are auxiliary.
3. **Editorial aesthetic** — clean white cards, subtle borders, professional typography. Think Bloomberg terminal or McKinsey report, not SaaS landing page.

## Color Palette
```
Sidebar bg:       #111827
Sidebar section:  #374151 (labels), #9ca3af (inactive items)
Active sidebar:   #f9fafb text, #1f2937 bg, #3b82f6 left border

Page bg:          #f9fafb
Card bg:          #fff
Card border:      1px solid #e5e7eb
Table alt rows:   #f9fafb (hover)

Text primary:     #111827
Text secondary:   #374151
Text muted:       #6b7280
Text disabled:    #9ca3af

Accent blue:      #3b82f6 (active states, links)
Over budget:      #d97706 (amber)
Under budget:     #059669 (green)

Demo badge:       #92400e text, #fffbeb bg, #fcd34d border
Warning banner:   same as demo badge
Actual cost highlight: #f8fbff bg, #0369a1 text
```

## Typography
```
Font: 'Inter' (Google Fonts) → system-ui → sans-serif fallback
Base size: 14px

Dash title:       15px, font-weight: 600
Table headers:    10-11px, font-weight: 700, uppercase, letter-spacing: .06em
Table data:       12-13px
KPI values:       24-26px, font-weight: 700
KPI labels:       10px, font-weight: 700, uppercase
Monospace fields: 'SF Mono' / 'Cascadia Code' / 'Roboto Mono' (for $, numbers)
```

## Component Patterns

**KPI Cards**: `.kpi-card` — white card, border, 16px padding. Label (10px uppercase) + Value (24px bold) + Note (12px muted).

**Table Cards**: `.tcard` — white card with overflow:hidden. Header `.tcard-hdr` has title + subtitle. `<table>` inside with `<th>` at 10px uppercase + `<td>` at 12-13px.

**Dash Header**: Fixed header at top of each panel. Title + subtitle on left. Demo badge on right. White bg, bottom border.

**DEMO Badge**: Amber pill on every panel. Text: "Demo · Sample Data". Never remove this.

**Warning Banners**: Amber background, ⚠️ icon, title + body text. Used on Data Ingest and AI Query to communicate non-functional status.

## CDN Dependencies
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js@4/dist/chart.umd.min.js"></script>
```

## Chart.js Usage
- Portfolio: Bar chart (Budget vs Actual) + Doughnut (region split)
- Project Detail: Horizontal bar chart (Budget vs Actual by category)
- Trade Pricing: 4 line charts (one per commodity, sparkline style at 90-140px height)
- Trade Detail tables: no charts, all tabular

All charts use Chart.js 4. Canvas elements with explicit heights. Colors follow the palette above.

## JavaScript Architecture

The entire app is ~2000 lines of vanilla JavaScript in a single `<script>` block at the bottom of the HTML. No frameworks, no modules, no build tools.

## Key Functions

### Navigation
```javascript
// Sidebar navigation (all sb-items with data-dash)
document.querySelectorAll('.sb-item[data-dash]').forEach(item => { ... });

// Programmatic navigation (back button, compare button, project card clicks)
function navigateTo(dashId, activeSidebarDash) { ... }
```

### Dashboard 2: Regional Compare
```javascript
function populateSelectors(region)  // Populates project A/B dropdowns filtered by region
function renderComparison()         // Reads selectors, computes normalized metrics, renders table
```

### Dashboard 3: Trade Pricing
```javascript
function getTradeData(key, market)         // Returns price array for a commodity/region
function getRegionalSeries(key, region)    // Per-quarter regional data using REGION_ADJ
function buildTradeSummaryTable(market)    // Aggregate all-commodities QoQ table
function buildCommodityDetailTable(key, bodyId, fmtFn)  // Per-commodity regional breakdown
function renderTradeCharts(market)         // Renders all 4 sparkline charts
```

### Cost Database
```javascript
function buildCostDbTable()  // Builds 10-project × 10-category table with click handlers
```

### Project History
```javascript
function buildHistoryCards()  // Builds 3-column card grid sorted by year desc
```

### Project Detail
```javascript
function openProject(projectId, fromDash)  // Populates dash-project panel dynamically
// Creates/destroys projChart (Chart.js instance) on each call
// Computes per-category budget estimates proportionally from total budget
```

### Portfolio Charts
```javascript
function buildPortfolioCharts()  // Creates bvaChart (bar) and regionChart (doughnut)
// These are persistent — only created once on load
```

### Bid Compare
```javascript
const BID_DATA = { katherines: {...}, wolfrd: {...}, nashville: {...} }
function renderBidTable(key)  // Renders full bid leveling table for selected project
function fmtDollar(v)         // Formats number as "$X,XXX,XXX"
```

### AI Query
```javascript
const AI_RESPONSES = { 'Highest $/SF project?': '...', ... }  // 5 hardcoded Q&A pairs
function runAIQuery()  // Matches input to AI_RESPONSES or returns generic fallback
function runEstimate() // Returns static estimate output HTML
```

### Utility Formatters
```javascript
const fmt$ = (v, dec=0) => '$' + v.toLocaleString(...)  // $1,234
const fmtM = v => '$' + (v/1e6).toFixed(1) + 'M'       // $18.2M
const fmtK = v => (v/1000).toFixed(0) + 'K'            // 148K
const fmtPct = v => (v>=0?'+':'') + v.toFixed(1) + '%'  // +4.9%
const fmtSF2 = v => '$' + v.toFixed(2)                  // $50.50
const fmt$2 = same as fmt$ with dec=2
function variance(p) { return (p.actual - p.budget) / p.budget * 100; }
function typeBadge(type)   // Returns HTML <span> with type badge styling
function varClass(v)       // Returns 'var-over' or 'var-under'
```

## Initialization (bottom of script)
```javascript
if ('scrollRestoration' in history) history.scrollRestoration = 'manual';
window.scrollTo(0, 0);
buildPortfolioTable();
buildPortfolioCharts();
populateSelectors('all');
renderComparison();
buildCommodityDetailTable('dock', 'detail-dock', v => fmt$(v));
buildCommodityDetailTable('tilt', 'detail-tilt', v => fmtSF2(v));
buildCommodityDetailTable('steel', 'detail-steel', v => fmt$(v));
buildCommodityDetailTable('site', 'detail-site', v => fmtSF2(v));
renderBidTable('katherines');
buildCostDbTable();
buildHistoryCards();
```

## Hyperagent Wrapper Injection

**Critical context for debugging the scroll bug.**

When the HTML file is served via `https://pub.hyperagent.com/p/7kaTQ7s8VAwHvnET2R9H1A`, Hyperagent injects a JavaScript wrapper **before** our HTML. The served file structure is:

```
<!DOCTYPE html>
<script>"use strict";
// ~112 lines of injected JS:
// 1. Error reporting (window.onerror, unhandledrejection → postMessage to parent)
// 2. A custom script loader that intercepts document.createElement("script")
//    and loads external scripts via fetch() instead of normal src loading
//    This affects how Chart.js loads!
// 3. Mobile browser detection and link rewriting
</script>
<!DOCTYPE html>
<html lang="en">
... our actual HTML ...
```

The custom script loader fetches external scripts (like Chart.js) asynchronously via `fetch()`. This means Chart.js may not be available when our initialization code runs.

**Implication:** When debugging locally (serving raw HTML without Hyperagent's injection), Chart.js loads synchronously and everything works. With Hyperagent's injection, Chart.js loads asynchronously — charts may fail silently, and timing-sensitive code (like scroll resets) may behave differently.

**Developer recommendation:** Test both with raw HTML (file:// or local server) AND via the Hyperagent URL to reproduce the exact user experience.

## What's Working vs What Needs Fixing

## ✅ Working
- All navigation (sidebar clicks, back buttons, compare buttons, project row clicks)
- Portfolio View: KPI cards, charts, project table, row click to project detail
- Regional Compare: region filter tabs, project selectors, comparison table, normalization
- Trade Pricing: sparkline cards, region filter, tabbed detail tables (all 4 commodities × 4 regions)
- Cost Database: full breakdown table with click handlers
- Project History: card grid with click handlers
- Data Ingest: layout and disclaimer (not functional by design)
- Bid Compare: dropdown switch between 3 projects, full bid leveling tables
- Project Detail: dynamic population, chart, back/compare navigation
- All formatting and content on all panels

## 🚨 Priority 1 Fix
**Blank panel on navigation** — AI Query, Data Foundation, New Estimate show blank white area when navigated to from a longer panel (e.g. Portfolio). Scroll position not resetting. See "The Outstanding Bug" section for full details.

## ⚠️ Non-Issues (by design)
- Data Ingest upload buttons do nothing (labeled "Coming Soon" — intentional)
- AI Query returns only 5 real answers + generic fallback (no LLM — intentional)
- New Estimate shows static output (no computation — intentional)
- All data is dummy (intentional — this is a prototype)

## 🔧 Nice-to-Have Improvements
These were not breaking issues but would improve the production rebuild:
1. Make the Bid Compare dropdown populate from the PROJECTS data array rather than hardcoded GC names
2. AI Query should compute real answers from the PROJECTS data (no LLM needed — just arithmetic)
3. The "% of Total" column in the Project Detail breakdown could be shown as a visual bar
4. Regional Compare could allow comparing projects across regions, not just within
5. Trade Pricing region filter could be linked to the user's project's region by default

## Production Rebuild Plan

When the prototype is ready for productionization, here is the agreed architecture:

## Stack
- **Frontend:** Next.js (React)
- **Database:** Supabase (managed Postgres)
- **Hosting:** Railway
- **Auth:** Supabase Auth
- **AI Query backend (future):** Claude API via Supabase Edge Function

## Route A (Fast Path)
Keep the single-file HTML as the functional spec. Replace the hardcoded `PROJECTS` array with Supabase client queries. Replace the static dummy data with real database reads/writes. This minimizes rebuild time.

1. Add Supabase JS client to existing HTML
2. Replace `const PROJECTS = [...]` with `await supabase.from('projects').select('*')`
3. Data Ingest forms now write to Supabase
4. Authentication via Supabase Auth (simple email login)
5. Deploy to Railway as a static file or Node.js server

## Database Schema (from Data Foundation panel)
```sql
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  type TEXT,  -- Industrial / Flex / Cold Storage / Mixed-Use
  region TEXT,  -- Southwest / South Central / Southeast / Midwest
  market TEXT,
  gc TEXT,
  sf INTEGER,
  clear_ht INTEGER,
  dock_doors INTEGER,
  site_acres DECIMAL,
  budget BIGINT,
  actual BIGINT,
  year INTEGER,
  -- 10 cost categories (in cents to avoid float issues)
  cost_foundations BIGINT,
  cost_steel BIGINT,
  cost_panels BIGINT,
  cost_roofing BIGINT,
  cost_mechanical BIGINT,
  cost_electrical BIGINT,
  cost_plumbing_fp BIGINT,
  cost_dock_equip BIGINT,
  cost_site_civil BIGINT,
  cost_gc_fee BIGINT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE trade_pricing (
  id SERIAL PRIMARY KEY,
  quarter TEXT,  -- e.g. 'Q1 2023'
  region TEXT,
  commodity TEXT,  -- dock / tilt / steel / site
  price_cents BIGINT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## Key Open Questions Before Rebuild
1. Who owns the Supabase project? (Stoten's account vs. operator's personal account)
2. User scope: just CMs Jack + one other, or broader Stoten team?
3. IP boundary confirmed? (This is Stoten work, not Stockbridge)
4. Commodity pricing source: manual entry, or automated feed from market data provider?
5. AI Query model: structured query engine (just arithmetic/SQL), free-form LLM, or hybrid?

## File Locations & Access

## GitHub
- **Repo:** `https://github.com/djegan97-oss/stoten-cost-intelligence` (private)
- **Account:** `djegan97-oss`
- **Branch:** `main`
- **File:** `prototype/stoten-cost-intelligence-v2.html`
- **README:** At root, describes the project and next steps
- **Current commit:** v16 (scroll timing fix)
- **Commit history:** 16 commits, all with descriptive messages explaining each fix

## Live URLs
- **Public (wrapper):** `https://hyperagent.com/s/7kaTQ7s8VAwHvnET2R9H1A`
- **Direct HTML:** `https://pub.hyperagent.com/p/7kaTQ7s8VAwHvnET2R9H1A`
- Both serve the same HTML via Hyperagent's CDN with the injected wrapper

## Local Testing
To test without Hyperagent injection:
```bash
git clone https://github.com/djegan97-oss/stoten-cost-intelligence
cd stoten-cost-intelligence
open prototype/stoten-cost-intelligence-v2.html
# or
python3 -m http.server 8080
# then navigate to http://localhost:8080/prototype/stoten-cost-intelligence-v2.html
```

## Operator Context
- **Who:** Managing Director at Stockbridge Capital Group, building tools for Stoten (a CRE development company run by friends)
- **Not a developer** — all communications should assume no CLI fluency
- **GitHub account:** `djegan97-oss` (all private repos)
- **This project is personal/independent work** — NOT Stockbridge time or resources. IP boundary is important.
- **Stack for all projects:** Railway + Supabase + GitHub
