# PIT STOP

**The Road Trip Planner that Knows the Way · by BEL**

A mobile-first concept mock of an AI road-trip planning experience. Built as a single self-contained HTML file — no build step, no server. Drop into any static host (GitHub Pages, Vercel, Netlify) and it runs.

---

## What's in here

```
pitstop/
├── index.html   — the entire app (HTML + CSS + JS, ~3,200 lines)
└── README.md    — this file
```

## How to ship to GitHub Pages

Add the folder to a GitHub repo and turn on Pages — that's it.

```
# new repo route
git init pitstop && cd pitstop
cp /path/to/index.html .
git add . && git commit -m "PIT STOP mock"
git push origin main
# then enable Pages on the main branch in repo Settings
```

To add it alongside the existing ARIA site:

```
# in the alexandrazelcer.github.io/ARIA repo
mkdir pitstop
cp /path/to/index.html pitstop/
git add pitstop && git commit -m "Add PIT STOP mock"
git push
# → live at alexandrazelcer.github.io/ARIA/pitstop/
```

## What's in the demo

**Five modes:**

- **Discover** — conversational *"where should we go?"* free text + chip shortcuts.
- **Build My Trip** — 4-step guided flow (type → vibe → distance → budget) that outputs a destination, alternative, premium upgrade, 3-day itinerary, **snack pack**, and playlist.
- **Pair** — match a destination with the right snacks, soundtrack, and timing.
- **Routes** — iconic American road trips (Pacific Coast Highway, Route 66, Blue Ridge Parkway, Maine Coast, Door County, Texas Hill Country, Oregon Coast, plus the featured *All Roads Lead to Hershey*).
- **Plot My Route** — enter Start + Destination, app maps the route with curated treat-stops, scenic detours, and passport-scannable locations.

**Featured at the top of every screen:**

- **All Roads Lead to Hershey** — a dedicated route card that plots a full Hershey getaway from any starting point.
- **Today's Pick alert bar** — urgency banner with a curated weekend deal.
- **Today's Featured Adventures rail** — horizontal scrollable strip of destinations with critic scores, promotions, and per-day pricing.

**Pit Stop Passport (top-right pill):**

- Sticker book of every scanned location (3 sample stamps pre-populated).
- Reward tier ladder — Welcome Bundle → Hershey itinerary → Custom snack pack → Hersheypark Fast-Pass voucher.
- **Demo "Scan +1" button** simulates an in-store QR scan and adds a stamp live.
- Passport markers also appear on Plot My Route stops, showing which locations along a planned drive are scan-eligible.

## Data model

Sixteen destinations in `CATALOG.destinations`. Each carries:

```
sku · name · region · state · category · scene (landscape SVG type) ·
colorTint · vibe{scenic, family, adventure, cultural, romantic, relaxed} 0-5 ·
tags[] · bestTime · availability{state, label} · duration · cost · popularity ·
ratings[] (T+L, CNT, AAA, NatGeo, LP, TA) · badges[] · pairings[] ·
notes · promotion
```

Six snack-pack categories — Drive · Cooler · Backseat · Campfire · Hotel · Refreshments — each with icon, description, sample items, and aisle placeholder. Eight curated iconic routes, including the featured Hershey route.

The Scorer module ranks destinations on six weighted axes (vibe, duration, budget, season, family, popularity) with modifiers for adventurous/family intent. The same engine powers Discover, Build My Trip, and Pair.

## Design system

Warm cream road-map aesthetic, all driven from CSS custom properties at the top of `<style>`:

- Background `#faf4e8` (warm cream) · surface white · ink `#2a1d14` (deep espresso)
- Primary accent **terracotta** `#c25a3a` · honey amber `#d4a24a`
- Vintage road-sign red `#a52f2a` for promos · deep ocean `#1f3b52` for trust elements
- Cocoa accent `#6e2820` reserved exclusively for the *All Roads Lead to Hershey* feature
- Fraunces serif for editorial headlines + Inter for UI
- Inline-SVG landscape backgrounds per destination (mountain, coast, coast-pine, forest, plains, desert, fall-mountain variants) — colored by the destination's `colorTint`

Fully responsive: mobile-first at 360–480px, tablet at 768px, laptop at 1024px+, ultra-wide kiosk-ready. Reduced-motion + touch-target sizing respected.

## What this is not (yet)

- No backend — saves and shares are toast notifications today
- No real geolocation, weather, or route API — Plot My Route uses curated sample stops
- QR scanning is simulated through the Passport modal's *Scan +1* button
- No real i18n catalog — locale pill cycles labels only

All of those are single-adapter swaps when the program goes live (see `Integrations` patterns in the ARIA build for the same architecture).

## Why this is built the way it is

The architecture is deliberately CPG-channel friendly. Every Pit Stop in a plotted route maps cleanly to a c-store, travel center, or retail location. The Passport stamps are designed to attribute physical in-store scans to digital app engagement. The Snack Pack output suggests treat categories without naming brands, making it a drop-in mechanism for a CPG sponsor's product lineup to populate by category.

## Brand positioning

The footer reads *"Powered by BEL + PIT STOP — Experiential Intelligence by Brand Experience Lab"* so anyone evaluating the mock sees who built it. The conversational entity, the brand mark, and the cohesion across modes are all the BEL platform proof — PIT STOP is the consumer-facing skin on top.

---

*v1.0 · Concept · Mock build for client review*
