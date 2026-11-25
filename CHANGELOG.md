# Changelog

## 2026-11-25 (branch `floor-mosaics`)
- **Enemy blood** in the enemy brand's contrast colour (GLD purple, RED coolant cyan, BLU arc orange, GRN bloom magenta). Every hit throws a few small speckles along the shot (more for bigger hits, `bloodHit` caps it); a death leaves a large pixel splotch with a darker core plus a ring of drops (`bloodDeath` sets its size). Drops stamp into stain canvases made only where fights happen, so stains last the whole run cheaply; landed blood is darkened so fresh drops read brighter. Drones and practice targets don't bleed. Toggle with `bloodOn`.
- Brand Wings: floors are now **ad-tile mosaics**. Every 20 px floor tile is a rented ad; colour follows location. The Entrance and Hub are public newsprint black and white; each wing uses only its brand's tone-board 950–600 (600 ink on 950/900 vinyl, alternating room to room), so step 500 stays reserved for things you can shoot, punch or pick up.
- Ink coverage per tile (five value bins) makes the tiles add up to a picture: each wing's room 1 and arena carry the brand logo (sampled from `assets/brand/`), and the Hub carries IDE with a smile under it (first pass used Ed's cracked smiley mask).
- Market link: below `mosVacAt` Worth a brand's tiles go vacant and fall back to public paper; at Floor the whole wing empties until Recovery. Brand tiles spill out of each Hub door, farther the higher that brand's Worth (`mosSpill`).
- Walls get a thin concrete rim and BLU's switched-off Beta panels a light outline, so both still read on the dark brand floors.
- Performance: tiles are baked into 320 px chunk canvases and only chunks whose look changes are re-baked (visible first, a few per frame). Measured in headless Chrome at 1440 × 900: about +0.5 ms per frame over flat floors; baking every chunk takes about 80 ms in total, spread over play as chunks come into view.
- New tunables (Floor mosaics): `floorMosaic` (0 = old flat room colours), `mosVacAt`, `mosSpill`.
- Noise pass: new `mosDetail` tunable, default **1 = quiet**. The fill now carries each tile's value; only tiles inside a mosaic hold an ad, printed 15% off the fill with one header bar and two lines (no borders or photo blocks); every other tile is plain vinyl; grout drops to 20%. Hub spill is a solid, tinted patch with a ragged edge instead of dithered speckle. `0` = flat tones only, `2` = the first busy pass.

## 2026-11-24
- Added `prototypes/brand-wings/`: a new level built on the Store Slice (`index.html` is unchanged). You start in the same Entrance; its north door opens into a central HUB, and four linear brand wings hang off it (BLU west, RED east, GLD and GRN north), playable in any order.
- Every wing runs R1 entry -> R2 signature room -> pay gate -> R3 gauntlet (checkpoint + that brand's ammo / health / buff machines) -> R4 arena. Clearing R1 opens R2; R3 clear opens the arena; the arena door locks until the bosses are down.
- Rooms: RED Loading Dock, Conveyor Lanes (belts push you sideways), Foundry Aisle (corridor with flanker bays), Crucible (pit and pillars). GLD Atrium (stepped diamond, relic on the altar), Procession (pews), Tithe Hall (V-pockets), Sanctum (octagon round a dais). GRN Produce Market, Greenhouse Stacks (two cubes swap on a timer), Fulfillment Maze (courier terminal), Canopy (cube rings). BLU Lobby (relic on the mezzanine), Server Hall (racks you can zap or hop through), Beta Floor (panels swap every 9 s with a warning flash), Keynote Stage.
- Wing enemies take the wing's brand (scripted waves always, Director spawns 70%, Temps 75%). One Director budget across the store, spent only in the wing you are in; RED leans Kill, BLU leans Drain (Taunt and Trick are not built yet).
- Placeholder: every arena uses the BOXY BROS pair in the wing's brand, which means four boss fights in one day against the one-Target-per-day design.
- Exit is the Street door on the Entrance's left wall: with all four arenas cleared it's the Settlement; leaving earlier ends the run and forfeits the loot bag.
- New tunables: `beltSpd`, `panelEvery`, `panelWarn`. Minimap is taller (300 x 180) to fit the wider store.

## 2026-11-23
- Added `docs/specs/melee-spec.md`: HAND OF THE MARKET melee hits now shove nearby enemies (neighbor kickback), and there are four brand strikes alongside the default: RED Rapid Deployment (short and quick), BLU Wire Transfer (fast and long), GRN Diversification (wide and arcing), GLD Tithe (heavy overhead slam). Mechanics Inventory rows 6.2–6.3 and the README docs table updated.
- Store Slice: removed the gun view from the bottom HUD; the panel now holds only the card reader, cash and ammo price per pull.
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
