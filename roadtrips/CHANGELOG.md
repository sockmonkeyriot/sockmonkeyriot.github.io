# Road Trips — Changelog

Version history for the `/roadtrips` hub and the itinerary template. One entry per
meaningful change to the **structure or format** (the landing page, the template
engine, the manifest schema). Adding or editing a *trip* doesn't need an entry
unless it changed the format to do it.

Versioning: `MAJOR.MINOR.PATCH` —
- **MAJOR** — template redesign or breaking change to the `TRIP` data schema
- **MINOR** — new feature, section, card field, or landing-page capability
- **PATCH** — fixes, styling tweaks, copy

The current version lives in `trips.json` (`"version"`) and is displayed in the
landing-page footer.

---

## v0.5.0 — 2026-06-10

Day maps now show the full day: START → numbered stops → END.

**Changed**
- Each day map gets a **START pill** (amber) where the day actually begins —
  the previous day's overnight, or the new `meta.start` (home) on Day 1 — and
  an **END pill** (ink) on the day's final stop. Intermediate stops are
  numbered 1…N between them. The route line (and OSRM geometry) now starts at
  START instead of the day's first stop.
- New schema field `meta.start: { nm, ll, map? }` — the trip's origin point.
  It also feeds the leg-link chain, so the trip's first stop now gets a leg
  link too (previously it had none).
- Leg-link label shortened to `↪ Just this leg` so both card buttons sit on
  one line (the origin is implicit: the previous stop).
- Stamping a stop now **collapses its card** to just the slot + name;
  unstamping expands it back. Done stops stop shouting, the day gets shorter
  as you live it.
- SLC→Chicago: removed the Route A return note from the Trip Summary panel
  (the engine's optional `summary.note` field remains for trips that want a
  closing line).
- **Timeline markers are now numbered** (1…N, last stop = END) instead of
  emoji, mirroring the day map exactly — list and map read as one. The `ic`
  emoji stays in the data as unrendered metadata.
- **Cover gains a trip-overview map**: all days' routes end to end, with an
  amber day-number badge at each day's stopping point — tap a badge or its
  segment to jump to that day. A START pill marks the trip origin and the
  final day folds into the END pill ("6 · END", tappable). Reuses the same
  cached per-day OSRM geometry (no extra router calls).
- Dropped the amber "first stop" marker — START replaces that role.

## v0.4.0 — 2026-06-10

Embedded day-route maps and per-leg directions links.

**Added**
- **Per-day route map** on every day panel (between the day note and the
  timeline): Leaflet + OpenStreetMap, numbered markers for the day's stops in
  order (first stop amber), tap a marker for a popup with the stop name and
  its Open in Maps link. Legs draw as a dashed connector immediately; the real
  driving geometry is fetched once per day from the public OSRM router and
  **cached in localStorage**, so repeat loads are instant, make no router
  calls, and keep the true route shape offline. Router unreachable → the
  dashed connector stays.
- **Leg links on every stop card**: an outline `↪ Leg from {previous stop}`
  button next to Open in Maps, opening Google Maps directions for exactly that
  leg (prev stop → this stop). Chains across day boundaries (Day 2's first
  leg starts at Day 1's overnight). The trip's first stop has no leg link.
- **Schema:** stops gain `ll: [lat, lng]` (geocoded via Nominatim). Stops
  without `ll` render normally but don't appear on the map.

**Offline posture**
- Leaflet loads from the unpkg CDN with graceful degradation: no network →
  no map blocks, everything else works exactly as before. Same posture as
  Google Fonts.

## v0.3.0 — 2026-06-10

Trip-page (template) simplification, matching the v0.2.0 hub cleanup.
Applied to `template/index.html` and rolled into `2026-slc-to-chicago/`.

**Changed**
- Cover: two-line title in one font ("Salt Lake" / "→ Chicago") — dropped the
  italic-amber connector word. Subtitle (the trip's one-line thesis) removed;
  that sentence already lives nowhere else the page needs it.
- Stop cards: the badge row is gone — no more colored type pills. The ★ Google
  rating and ◆ landmark flag now append to the timing line
  (`Opens 11 AM · ★ 4.7 · ◆ landmark`), so each card is name → one meta
  line → the why.

**Schema**
- `cover.titleJoin` and `cover.sub` removed from the TRIP schema.
- Stop `t` (category) retained in the data as unrendered metadata — the emoji
  marker carries the category visually.

## v0.2.0 — 2026-06-10

Landing-page simplification (per Daniel's review).

**Changed**
- Hero: title is plain "Road Trips" in one font (dropped the italic-amber
  second word); subtitle removed; band tightened.
- Trip cards collapsed into compact one-line rows: serif trip name, a single
  mono meta line (`dates · days · miles`), amber chevron. Whole row is the link.

**Removed**
- Per-trip taglines on the landing page (still stored in `trips.json` as
  metadata; just not rendered).
- Chips row, stops count, status badges (section headers already convey
  status), and the "Open itinerary" button.
- Unplanned trips now say `… · In planning` in the meta line and render
  unlinked at reduced opacity.

## v0.1.0 — 2026-06-09

Initial conversion from one-off app to a structure.

**Added**
- `index.html` — landing page at `/roadtrips/`: mobile-first scrollable trip list
  with three sections (Current / Upcoming / Archived). Status is derived
  automatically from each trip's start/end dates, so trips move between sections
  on their own as time passes.
- `trips.json` — trip manifest (same pattern as the blog's `index.json`). The
  landing page renders entirely from this file; adding a trip = adding an entry.
- `template/index.html` — the itinerary app abstracted into a reusable template.
  All trip-specific content now lives in a single `TRIP` data object at the top
  of the script; everything below it is a generic engine (cover, swipe deck,
  day tabs, stop cards, summary). Ships with a 2-day demo trip that exercises
  every card field.
- `TEMPLATE.md` — written spec for the format: data schema, section anatomy,
  stop types, design language, interaction model. The doc to iterate on when
  changing the format independent of any trip.
- `2026-slc-to-chicago/` — the SLC→Chicago trip (Jun 12–17, 2026) migrated onto
  the template engine. Content unchanged from the original app; `localStorage`
  key preserved so existing visited-stamps survive.
- Chicago→SLC return trip added to the manifest as an unplanned placeholder
  (shows in Upcoming as "in planning"; no itinerary page yet).

**Engine changes vs. the original one-off app**
- Tabs, day labels, nav captions, counters, and the summary panel are now
  generated from `TRIP` instead of hardcoded (the original broke past 9 panels).
- Document title, header, and cover render from data.
- Optional per-trip theme overrides via `TRIP.meta.theme`.

**Open questions for future versions**
- Shared engine file vs. self-contained copies per trip (currently:
  self-contained, so archived trips never break; tradeoff is engine fixes don't
  propagate backward).
- Redirect or retire the old live URL `/2026slctochicago` once this structure
  is pushed.
