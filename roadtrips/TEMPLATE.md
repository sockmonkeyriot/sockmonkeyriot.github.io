# Road Trip Itinerary — Template Spec

This is the working document for the **format** of the itinerary web apps —
what a trip page is, what it contains, and how it behaves — independent of any
specific trip. Change the format here first, then implement in
`template/index.html`, then roll into trips. Log format changes in
`CHANGELOG.md`.

## The model

Each trip is a **self-contained, single-file web app** (`<trip-id>/index.html`,
vanilla JS, no build step, no dependencies beyond Google Fonts and Leaflet —
both CDN-loaded with graceful degradation: offline, the maps simply don't
render and everything else works). It's designed phone-first: full-bleed on
mobile, framed like a phone on desktop. It keeps state (visited stamps,
cached day-route geometry) in `localStorage`.

A trip file has two parts:

1. **The `TRIP` data object** — everything specific to the trip, in one block
   at the top of the `<script>`. Creating a new trip = copy
   `template/index.html`, replace `TRIP`.
2. **The engine** — everything below the data block: rendering, navigation,
   stamps. Identical across trips. The canonical copy lives in
   `template/index.html`.

> **Tradeoff (open question):** engine code is *copied* into each trip rather
> than shared, so archived trips never break when the format evolves. The cost
> is that engine fixes don't propagate backward. Revisit if trip count grows.

## Page anatomy

A trip page is a horizontal swipe deck of panels:

```
[ Cover ] [ Day 1 ] [ Day 2 ] … [ Day N ] [ Summary ]
```

- **Header (fixed):** trip kicker + panel counter, trip title, day tabs,
  amber progress bar.
- **Cover:** the poster. Kicker line, big two-line serif title in one font
  ("Origin" / "→ Destination" — the arrow lives in the data), the route line
  (every waypoint, mono), 2–4 headline stats (miles, days, span), swipe hint.
  Sunrise + ridge art, pine gradient. No subtitle.
- **Day panel:** day header (Day N + date, route as the headline, miles/time
  chips), an italic editor's note (the day's shape and any timing logic), an
  embedded **day route map** (Leaflet + OpenStreetMap: numbered markers for
  the day's stops in order, first stop amber; tap a marker for the stop name +
  Maps link; the leg line starts as a dashed connector and upgrades to the
  real driving shape from OSRM, cached in localStorage), then a vertical
  **timeline of stop cards** connected by a dashed route line.
- **Summary:** one row per day (date, destination, miles, where you're
  sleeping, the day's keystones), a total box, and an optional closing note
  (e.g., the return plan).
- **Bottom nav (fixed):** Prev / location caption / Next.

## Stop cards

Each stop is a card on the day's timeline with an emoji marker. Fields:

| Field  | Req | What it is |
|--------|-----|------------|
| `ic`   | ✓   | Emoji for the timeline marker (☕ 🍽️ 🍔 🍸 🦖 🅿️ 🚐 🏨 🚇 🏁 …) |
| `slot` | ✓   | Role in the day, small caps: `Rollout`, `Breakfast`, `Lunch`, `Stop`, `Stop · Optional`, `Drinks`, `Dinner`, `Dessert`, `Parking`, `Overnight · Free`, `Hotel · Optional`, `Logistics`, `Arrive` |
| `nm`   | ✓   | The name. Specific beats generic. |
| `t`    | ✓   | Category metadata: `coffee`, `lunch`, `dinner`, `drinks`, `dessert`, `stop`, `overnight`, `hotel`, `parking`, `logistics`. Stored but not currently rendered — the emoji marker carries the category visually. |
| `when` | ✓   | Timing line: arrival time, hours, *the catch* (closes early, sells out, $/vehicle) |
| `why`  | ✓   | 1–3 sentences on why this stop earns its place. Voice: confident, concrete, second person. |
| `ll`   |     | `[lat, lng]` — places the stop on the day map and anchors leg links. Without it the card renders normally but the stop is absent from the map. Geocoded once (Nominatim) when authoring. |
| `addr` |     | Street address (shown with pin icon) |
| `rate` |     | Google rating, appended to the timing line as ` · ★ 4.8` |
| `lm`   |     | `true` → ` · ◆ landmark` appended to the timing line (bucket-list road-food / Americana) |
| `map`  |     | Google Maps search URL → "Open in Maps" button |
| `warn` |     | `true` → button styled as a warning (van clearance, bans, gotchas) |
| `alt`  |     | "Also" footer: 1–3 genuinely different alternates, ` · ` separated, with ratings |

Card interactions: each card has a **stamp** (✓) to mark a stop visited —
strikes the title, fills the stamp, persists per-trip in `localStorage`.
Each card also carries two map actions: the filled **Open in Maps** button
(navigate to this stop) and an outline **↪ Leg from {previous stop}** link
(Google Maps directions for exactly that leg; the chain crosses day
boundaries, and the trip's first stop has none).

### Content conventions (from how Daniel travels)

- Every day ends with **where the van sleeps**: a free, legal, vetted
  `overnight` stop, with a backup in `alt`. Flag towns that ban overnight
  parking with `warn`/note. An optional `hotel` card when a reset makes sense
  (showers get a mention either way).
- **Parking is a stop.** "Park once, walk the night" — call out van-height
  limits (garages are the enemy), free street zones, surface lots.
- Meals carry the trip: one serious room and one road-food landmark beat
  three okay places. `alt` lists are ranked and real, not filler.
- Surface the catch in `when`: closed Mondays, opens at 3, sells out by noon.
- Drive math in the day header chips (`~334 mi`, `~6h drive`); timing
  anchors ("park by ~4:45 PM") in `when`/`why`.

## TRIP data schema

```js
const TRIP = {
  meta: {
    id: "2026-slc-to-chicago",   // also default localStorage key prefix
    storageKey: "slc2chi_done",  // optional override (legacy keys)
    docTitle: "Salt Lake → Chicago · Road Tour",  // <title>
    kicker: "◆ Eastbound · Jun 12–17",            // header top-left, mono
    title: "Salt Lake City → Chicago",            // header title line
    theme: { "--amber": "#d9802f" }  // optional CSS-var overrides, see below
  },
  cover: {
    kick: "A van road trip · Jun 12–17, 2026",
    titleTop: "Salt Lake",      // h1 line 1
    titleBottom: "→ Chicago",   // h1 line 2 — one font, arrow in the data
    routeline: "SLC → Vernal → … → Logan Square",
    stats: [ { n: "~1,580", l: "Total miles" }, … ],   // 2–4
    hint: "Swipe to begin · or tap a day above"        // optional
  },
  days: [
    {
      dow: "Friday · Jun 12",       // day header date line
      route: "SLC → Vernal → Steamboat",  // day headline
      miles: "~334 mi", time: "~6h drive",
      note: "Italic editor's note for the day.",
      tab: "1",                     // optional tab label (default: index)
      short: "Steamboat",           // bottom-nav caption → "Day 1 · Steamboat"
      sum: {                        // this day's row on the summary panel
        sd: "Fri 6/12",             // short date
        dest: "Steamboat Springs, CO",
        mi: "~334 mi",
        lo: "Rabbit Ears Pass dispersed (free)",  // lodging/overnight
        ks: "Keystones · separated · like this"
      },
      stops: [ { ic, slot, nm, t, ll?, when, why, addr?, rate?, lm?, map?, warn?, alt? }, … ]
    }, …
  ],
  summary: {
    total: { headline: "~1,580 miles · 6 days",
             sub: "Friday morning, Jun 12 → Wednesday evening, Jun 17" },
    note: "Optional italic closing note (return plan, etc.)"
  }
};
```

## Design language

- **Palette (CSS vars):** aged paper (`--paper #f1e7d5`, `--card #faf4e8`) on a
  deep pine backdrop; ink `#2a2017`; accents amber `#d9802f` (action/primary),
  clay `#b1442a` (timing/headers), pine `#1f4a3a` (map buttons). Paper-grain
  noise overlay on everything.
- **Type:** Fraunces (serif — titles, names, italic notes), Hanken Grotesk
  (sans — body), DM Mono (mono — labels, chips, timings, nav). Small-caps
  letter-spaced mono is the "wayfinding" voice; serif is the "magazine" voice.
- **Texture moves:** dashed route lines, stamp circles, chips, hand-offset
  card shadows (`2px 2px 0`), national-park-poster cover art.
- **Per-trip theming:** `meta.theme` can override any CSS var (e.g., shift the
  pine/amber pair for a desert or coastal trip) without touching the engine.

## Interaction model

- Horizontal scroll-snap deck; swipe, Prev/Next buttons, or day tabs.
- Progress bar + `NN / NN` counter in header; captions in bottom nav.
- Stop cards stagger-fade in when a panel becomes active.
- Visited stamps persist in `localStorage` under `meta.storageKey`.
- All map links are `https://www.google.com/maps/search/?api=1&query=…` URLs —
  they open the native Maps app on the phone.

## The hub (`/roadtrips/`)

`index.html` at the hub root renders the trip list from `trips.json` —
sections **Current / Upcoming / Archived**, derived from each trip's
`start`/`end` dates against today (no manual status flag). Each trip is a
compact one-line row: serif name + a single mono meta line
(`dates · days · miles`); the whole row links to the trip. Unplanned trips
(`path: null`) render unlinked at reduced opacity with `· In planning`
appended. Manifest entry fields: `id`, `name`, `origin`, `destination`,
`start`/`end` (ISO, null if unplanned), `dates` (display string), `days`,
`miles`, `stops`, `tagline`, `path` — `stops` and `tagline` are stored but
not currently rendered by the hub. Hub version in `trips.json → version`,
shown in the footer.
