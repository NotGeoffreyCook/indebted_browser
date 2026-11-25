# Changelog

## 2026-11-25
- Store Slice (`index.html`): first art pass toward **Graphic Realism** (art_direction_v1 in the Production Management project). Rendering only; no mechanics, values or seeded rolls changed.
  - Floors are neutral concrete with pour joints; walls are dark concrete with a lit top edge instead of flat black.
  - One hard cast shadow, down and to the right, under walls, shelves, machines, merchandise, enemies and Ed. Enemies keep their accent offset on top of it (accent in the shadow zone).
  - Shelves are painted metal with white price tags along the aisle edge.
  - FIRE brand space: pink floor replaced by concrete with a painted RED zone line, and the label is now a physical sign with a black offset backing plate.
  - Brand signage and brand machines brighten and dim with live Worth (`signLit`: 0.3 + 0.7 x Worth) and go dark at Floor.
  - Each pull drops a thermal receipt on the floor to the left of the chamber (capped at 160; uses `Math.random`, not the day seed).
  - Minimap uses the concrete palette. All values live in the new `ART` constant.
- Open conflict: `docs/specs/level-design-guide.md` says every district surface takes its brand hue with no grey; art_direction_v1 proposes a mostly neutral world with brand color reserved for things that matter. Needs a decision before the brand-wings level gets the same pass.

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
