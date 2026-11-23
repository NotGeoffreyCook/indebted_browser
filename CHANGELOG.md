# Changelog

## 2026-11-23
- Store Slice (`index.html`): the level now runs vertically. You start in the Entrance at the bottom and move up through Seg 1-3 to the Arena at the top. The layout is the same plan rotated 90° (`tp`/`tr`/`toPlan` helpers), so the FIRE brand space now sits on the right of Seg 2.
- Store Slice: 300 px of extra floor below the Entrance so the bottom HUD doesn't cover the start; practice targets moved to a column on the Entrance's right wall.
- Store Slice: the four brand stock tiles now sit in one row under the full-width news ticker; progress panel, DEBUG button and debug panels hang below the row.
- GRN enemy reaction bias changed from Task Force (regroup) to **Scatter & Flank**: shot or near-missed, a GRN enemy hops sideways off your aim line (groups alternate sides), then paths to a spot 60–120° off your facing and re-engages from there. Falls back to Break Sightline if no flank spot is found; counts toward the 3-repositioner cap (over the cap it only scatters); 6 s cooldown; not used by Flankers or Couriers. Weights: Flank +40, Sightline +10, Hesitate −15, Push −25. New tuning values replace the Task Force ones; badge is FL.
- Synced `docs/design/design-reference-v2.md` to the current Design Reference (2026-11-23), which also brings in the 2026-11-22 market fixes pass and the Enemy Reactions section.
- Updated the Store Slice Prototype (`index.html`) to the latest artifact version: pixel-art level effects, damage-only enemy numbers colored by ammo, room progress tracking, and the Director halting in cleared rooms.

## 2026-11-20
- Initial import of the Store Slice Prototype from its Claude artifact (version of 2026-11-19).
- Added 15 earlier browser prototypes under `prototypes/`.
- Added design docs (Design Reference v2, AI Director v0.2) and specs (Market, Credit Card, Mechanics Inventory, Level Design Guide) as Markdown.
- Added brand logos, tone boards and the rules sheet under `assets/brand/`.
