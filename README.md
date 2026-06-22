# ARIA — Aisle Retail Intelligence Assistant

**Built by BEL (Brand Experience Lab)**
A premium, conversational AI kiosk experience for the retail floor — wine and spirits, with an architecture designed to scale to grocery, convenience, and specialty retail.

---

## What this is

ARIA is a kiosk-ready conversational assistant that meets shoppers where they are — at the fixture, in the aisle, on their phone, or at an unattended terminal. It reduces abandonment at locked cabinets, replaces the friction of waiting for an associate, and turns a passive endcap into an active selling surface.

The current build is a single-file demo (`index.html`) that runs anywhere — touchscreen kiosks, tablets, phones, browsers. It contains the full design system, a 16-wine sample catalog with rich metadata, a pairing engine, a four-step "Build My Dinner" workflow, region exploration, scan-a-bottle, and mock adapters for the retail integration stack.

---

## File layout

```
/
├── index.html          — entire experience (HTML + CSS + JS, ~1900 lines)
└── README.md           — this file
```

The script section of `index.html` is partitioned into nine numbered modules so the eventual extraction into a build pipeline is trivial.

---

## Code modules

The `<script>` block in `index.html` is structured as follows:

1. **CONFIG** — brand identity, store metadata, scoring weights.
2. **CATALOG** — wine database (16 SKUs) and complementary pantry SKUs used by the shopping list builder.
3. **PAIRINGS** — food-dish keyword taxonomy, occasion modifiers, preference shortcuts, budget tiers.
4. **SCORER** — natural-language query parser plus a weighted ranking engine.
5. **SESSION** — user state (mode, history, cart, preferences, associate flag).
6. **INTEGRATIONS** — adapters with `async` signatures matching real retail endpoints: inventory, POS handoff, loyalty, QR handoff, voice, analytics, i18n.
7. **RENDER** — per-mode UI controllers (Discover, Dinner, Pair, Region, Scan) and the shared atoms used by all of them.
8. **ARIA** — the conversational micro-agent that composes natural-language explanations, dinner intros, and dish suggestions.
9. **BOOT** — wiring, event binding, mode switching, clock tick, analytics ping.

---

## Data model

### Wine record

Every bottle in `CATALOG.wines` exposes the full retail data model:

| Field | Example | Notes |
|---|---|---|
| `sku` | `JC-CAB-750` | Stable SKU for inventory + POS handoff. |
| `name`, `vintage`, `varietal` | `Josh Cellars Cabernet · 2021 · Cabernet Sauvignon` | Display. |
| `region`, `country` | `North Coast · USA` | Used by the Region explorer. |
| `tasting` | `{ acidity, tannin, sweetness, body, alcohol }` | 0–5 scale for the visual profile bars; `alcohol` is ABV. |
| `notes` | Long-form tasting text | Shown in card and in scan view. |
| `pairings` | `['beef','lamb','aged-cheese']` | Canonical tags that join with the food taxonomy. |
| `price`, `msrp` | `15.99`, `16.99` | Used by the price-fit scorer and the cart total. |
| `inventory` | `{ stock: 'in'|'low'|'out', count }` | Wired to `Integrations.inventory`. |
| `popularity` | `0.94` | 0–1, drives velocity bonus in the scorer. |
| `colorTint` | `['#5a1f2b','#2d0e14']` | Bottle SVG gradient (placeholder for real label art). |
| `aisle` | `A14 · Shelf 3` | Shown in every card so the shopper can find it. |
| `badges` | `['bestseller','premium-pick']` | Render as colored chips. |
| `tags` | `['bold-red','crowd-pleaser']` | Free-form discoverability. |

### Food taxonomy

`PAIRINGS.dishMap` maps regex-friendly dish patterns ("salmon\|trout\|halibut") to canonical tags. The same pattern model also covers `prefMap` (body/acidity/tannin shortcuts), `occasionMap` (party / weeknight / gift), and `budget` tiers.

A shopper can describe a meal any way they like — "salmon dinner under twenty bucks for a Tuesday" — and the parser produces a structured query with pairings, profile, modifiers, and a price cap that the scorer ranks against.

---

## Recommendation scoring

`Scorer.rank(query)` walks every wine in the catalog and computes a single `score` between 0 and 1 from six weighted axes (`CONFIG.scoringWeights`):

| Axis | Weight | What it captures |
|---|---|---|
| `pairing` | 0.40 | Overlap of `wine.pairings` with `query.pairings`. |
| `preference` | 0.25 | Distance between `wine.tasting` and the parsed user profile. |
| `price` | 0.15 | Reward for fitting under `maxPrice`, penalty for going over. |
| `inventory` | 0.10 | Bonus for in-stock, half-bonus for low-stock, zero for out. |
| `popularity` | 0.05 | Velocity nudge — doubled when the user signals they want a crowd-pleaser. |
| `premium` | 0.05 | Upsell handle for premium-pick badges. |

Modifier flags (`adventurous`, `premium`, `budget`) reshape the curve — adventurous favours less-popular wines, premium favours `premium-pick` and price ≥ $25, budget favours `best-value` and price ≤ $15.

The scorer is the single point where future ML can plug in: drop in a learned model for the `score` function and the rest of the system (UI, conversational copy, cart) needs no change.

---

## Conversational layer

`Aria.compose(input, query, ranked)` writes the natural-language intro the shopper sees above a recommendation set. It assembles fragments dynamically from the parsed query — pairings, budget, modifiers, sparkling intent — so the same explanation never appears twice.

`Aria.reason(wine, query)` writes the short italic "why we matched" line under each card.

`Aria.dinnerIntro(protein, style, wine)` produces protein-specific sommelier copy for the Build My Dinner flow.

`Aria.dishesForWine(wine)` powers the Wine → Food direction with a curated bank of dish suggestions.

All ARIA copy is concise, retail-aware, and confident — sommelier-warm rather than chatbot-cheery.

---

## Modes

| Mode | What it does |
|---|---|
| **Discover** | Conversational find-a-wine. Chips + free text. ARIA proposes three ranked bottles with rationales. |
| **Build My Dinner** | Four-step guided flow: protein → occasion → budget → plan. Produces top pick, alternative, premium upgrade, and a complete shopping list with aisle locations and a QR handoff. |
| **Pair** | Two-way pairing — Food → Wine (uses the scorer) or Wine → Food (uses a curated dish library). |
| **Explore (Region)** | Region cards that filter the catalog by region/country. |
| **Scan** | Tap a sample SKU and see the full tasting profile with animated bar chart. Hooks for a real barcode/vision scanner. |

---

## Integrations (mock adapters)

All adapters live under `Integrations.*` and match the shape of a real production endpoint. Swap each `Mock` for a `fetch()` and the rest of the app is unchanged.

| Adapter | Method | Real endpoint contract |
|---|---|---|
| `inventory` | `checkStock(sku)` | `GET /api/inventory/:sku` returning `{ stock, count }`. |
| `pos` | `addToCart(sku)` | `POST /api/cart/add` returning `{ ok, sku, ts }`. |
| `loyalty` | `profile()` | `GET /api/loyalty/:memberId` returning `{ tier, preferences }`. |
| `qr` | `createHandoff(payload)` | `POST /api/handoff` returning `{ id, url, payload }`. |
| `voice` | `listen()` | Web Speech API hook. |
| `analytics` | `track(event, props)` | `POST /api/telemetry`. |
| `i18n` | `t(str)` | Locale catalog lookup. |

A status panel in the right-hand sidebar shows each integration's wire-up state (`Live` vs `Mock`) so operators can see at a glance what's hooked up in a given deployment.

---

## Design system

A single `:root` variable block defines the entire visual language so the experience can be re-skinned for different retailers without touching component code.

| Token | Value | Use |
|---|---|---|
| `--bg`, `--surface`, `--surface-2/3` | Near-black to charcoal | Page + card surfaces. |
| `--ink`, `--ink-soft`, `--ink-mute`, `--ink-faint` | Soft white scale | All typography. |
| `--gold`, `--gold-bright` | `#c9a861`, `#e2c184` | Primary accent — buttons, badges, ARIA bubble. |
| `--wine`, `--wine-deep` | `#8a2f3f`, `#5a1f2b` | Secondary accent for wine-specific UI. |
| `--font-serif` | Fraunces | Editorial headlines, wine names, panel prompts. |
| `--font-sans` | Inter | UI, copy. |
| `--r-pill`, `--r-lg`, `--r-xl` | 999px / 20px / 28px | Apple-Store style radii. |
| `--shadow-md`, `--glow-gold` | Soft + warm | Depth + warmth on hover. |

The aesthetic target — modern, premium, conversational — sits at the intersection of Apple Store retail, Eataly, and an upscale wine bar. Generous touch targets (≥44px), kiosk-friendly spacing, and a hard ceiling on visual density.

---

## Extending to other categories

The architecture is deliberately category-agnostic. To extend ARIA to (e.g.) cheese, coffee, or whisky:

1. Add records to `CATALOG.wines` (rename or duplicate to `CATALOG.cheeses`, etc.) with the same data shape — tasting axes can be relabelled (`acidity` → `funk`, `tannin` → `sharpness`).
2. Update `PAIRINGS.dishMap` with the new domain's pairing vocabulary.
3. Adjust mode pills and copy in `Discover`, `Pair`, and `Region`.
4. The scorer, ARIA conversational layer, and rendering atoms are reused as-is.

The same chassis can host any guided-discovery retail problem where a shopper needs a confident recommendation grounded in inventory and a few preferences.

---

## What's intentionally placeholder

The following are **mock** in this build and explicitly noted in the right-hand "Retail Stack" panel:

- Live inventory sync — replace `Integrations.inventory.checkStock` with the real endpoint.
- POS handoff — replace `Integrations.pos.addToCart`.
- Loyalty personalization — return real preferences from `Integrations.loyalty.profile`.
- QR handoff URL — currently a placeholder host (`aria.bel.io/handoff/...`).
- Voice — the adapter is wired but the recognizer is not started in this demo.
- Multilingual — the locale pill cycles `EN → ES → FR → JA` and emits an analytics event; the i18n catalog is a passthrough.
- Associate mode — the toggle flips a session flag and is wired for future associate-only UI affordances.

Each is a single-line swap from mock to live.

---

## How to run

Open `index.html` in any modern browser. No build step, no server.

For kiosk deployment, recommend launching in fullscreen kiosk mode (`--kiosk --noerrdialogs` in Chromium) at 1920×1080 or 1080×1920.

---

## Brand positioning

Throughout the experience BEL's positioning is reinforced subtly: the top bar carries the BEL + ARIA co-brand, the footer reads *"Powered by BEL + ARIA · Retail Intelligence by Brand Experience Lab,"* the version line marks the build as a kiosk concept, and the right-hand panel telegraphs the production-grade integration surface that BEL brings to deployments. The intent is for any pilot site visit to read as *"a premium AI-powered retail intelligence kiosk built by BEL and powered by ARIA."*
