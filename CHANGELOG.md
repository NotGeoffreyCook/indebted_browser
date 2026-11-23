# Changelog

## 2026-11-23
- GRN enemy reaction bias changed from Task Force (regroup) to **Scatter & Flank**: shot or near-missed, a GRN enemy hops sideways off your aim line (groups alternate sides), then paths to a spot 60–120° off your facing and re-engages from there. Falls back to Break Sightline if no flank spot is found; counts toward the 3-repositioner cap (over the cap it only scatters); 6 s cooldown; not used by Flankers or Couriers. Weights: Flank +40, Sightline +10, Hesitate −15, Push −25. New tuning values replace the Task Force ones; badge is FL.
- Synced `docs/design/design-reference-v2.md` to the current Design Reference (2026-11-23), which also brings in the 2026-11-22 market fixes pass and the Enemy Reactions section.
- Updated the Store Slice Prototype (`index.html`) to the latest artifact version: pixel-art level effects, damage-only enemy numbers colored by ammo, room progress tracking, and the Director halting in cleared rooms.

## 2026-11-20
- Initial import of the Store Slice Prototype from its Claude artifact (version of 2026-11-19).
- Added 15 earlier browser prototypes under `prototypes/`.
- Added design docs (Design Reference v2, AI Director v0.2) and specs (Market, Credit Card, Mechanics Inventory, Level Design Guide) as Markdown.
- Added brand logos, tone boards and the rules sheet under `assets/brand/`.
