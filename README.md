# Hilton Head 2026 — Family Vacation Itinerary

A lightweight web app that displays a family vacation itinerary as interactive tiles, pulling live data from a cloud database. Built to explore a simple but complete full-stack workflow: structured data → cloud database → dynamic frontend → hosted URL.

**Live site:** https://offandrunning.github.io/HHISC/

---

## What It Does

The app renders 54 tiles — one for every combination of day (June 19–27) and meal/activity category (Breakfast, Lunch, Dinner, AM Activities, PM Activities, Kid Activities). Each tile shows the date, a color-coded category badge, and the planned activity. A filter bar at the top lets you view all tiles or narrow to a single category.

---

## Tech Stack

| Layer | Tool |
|-------|------|
| Database | [Supabase](https://supabase.com) (PostgreSQL, cloud-hosted) |
| Frontend | Vanilla HTML, CSS, JavaScript (no framework, no build step) |
| Data fetching | Supabase JS SDK loaded from CDN |
| Hosting | GitHub Pages |

---

## How It Was Built

### 1. Structured the source data

Started with a spreadsheet (`Vacation Tracker 2026 - Sheet1.csv`) laid out with dates as columns and meal/activity categories as rows. Identified that the right schema was a flat table — one row per tile — rather than keeping the spreadsheet's matrix shape.

### 2. Set up Supabase

Created a free Supabase project and ran the following SQL in the dashboard to create the `activities` table:

```sql
create table activities (
  id serial primary key,
  date text not null,
  category text not null,
  description text not null
);
```

Enabled Row-Level Security and added a public read policy so the frontend can query the data without authentication:

```sql
alter table activities enable row level security;

create policy "Public read" on activities
  for select using (true);
```

### 3. Seeded the data

Converted the CSV's matrix structure into 54 flat `INSERT` rows and ran them in the Supabase SQL Editor:

```sql
insert into activities (date, category, description) values
  ('6/19/26', 'Breakfast', 'Breakfast at Home'),
  ('6/19/26', 'Lunch', 'Fast food'),
  -- ... 52 more rows
```

### 4. Built the frontend

Wrote a single `index.html` file with no build tooling. The Supabase JS client is loaded directly from a CDN using an ES module import:

```js
import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js/+esm'

const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY)
const { data } = await supabase.from('activities').select('*').order('id')
```

On load, the script fetches all rows and dynamically builds the tile grid. Each tile is assigned a CSS class based on its category, which maps to a distinct pastel color scheme defined in CSS custom properties. The filter buttons toggle a `.hidden` class on tiles client-side — no additional network requests needed.

### 5. Deployed to GitHub Pages

Pushed the repo to GitHub and enabled Pages (Settings → Pages → Deploy from branch: main). The site is served directly from `index.html` at the repo root.

---

## Key Design Decisions

**No framework, no build step.** The scope is a single read-only page with static structure. A framework would add complexity without adding value here. Vanilla JS with ES modules is sufficient and keeps the project completely dependency-free locally.

**Flat table schema over a matrix.** Storing each cell as its own row makes querying, filtering, and rendering straightforward. It also makes the data easy to extend — adding a new category or date is a single INSERT, not a schema change.

**Anon key with RLS over open access.** The Supabase anon key is safe to expose in frontend code because Row-Level Security is enabled and the only policy is `select`. No writes are possible through this key.

---

## Project Structure

```
HHISC/
├── index.html                        # Entire frontend — markup, styles, and JS
├── Vacation Tracker 2026 - Sheet1.csv  # Original source data
└── README.md
```
