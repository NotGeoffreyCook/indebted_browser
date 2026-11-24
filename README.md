# InDebtEd — Browser Prototype & Design Archive

Prototypes and design docs for **InDebtEd**, a first-person roguelike (UE5) where every bullet costs money and the economy runs on a live stock market across four brands: GLD, RED, BLU, GRN.

This repo is for **mechanics, values and rules**. Visuals, effects and game feel live in the UE5 project, which is also the reference for the part system and repay mini games.

## Run it

Open any `index.html` in a browser. Each one is a single self-contained file with no build step. Most load three.js r128 and Google Fonts from CDNs, so they need an internet connection.

## Current prototype

| Path | What it is |
|---|---|
| `index.html` | **Store Slice Prototype**: the current top-down browser version |

## Earlier prototypes (`prototypes/`)

Newest first.

| Folder | What it is |
|---|---|
| `brand-wings/` | Hub-and-wings store level built on the Store Slice: same Entrance start, a central hub, and four 4-room brand wings (GLD, RED, BLU, GRN), each with its own pay gate, brand machines and arena |
| `ed/` | InDebtEd ED |
| `gameplay-slice/` | InDebtEd Gameplay Slice |
| `test-environment/` | InDebtEd Test Environment |
| `store-floor/` | Walkable low-poly store with all 48 merchandise models; shoot items to explode them for cash |
| `merchandise/` | Low-poly 3D renders of all 48 shelf products: four brands × four industries × three sizes |
| `enemy-lab/` | Enemy test room with the 0 HP flow, four brand health plans and a BLU Beam barrel |
| `lighting-bench/` | 3D scene for tuning key, fill, rim and atmosphere lighting per brand and market state |
| `brand-heads/` | 3D exoskeleton heads for GLD, RED, BLU and GRN with Base/Bull/Bear states |
| `test-level/` | InDebtEd Test Level |
| `gun-parts/` | InDebtEd Gun Parts |
| `escrow-park-courtyard/` | Top-down level layout: brand-wing segments, pay-to-pass gates, ModSlots, arena sockets, Director setup |
| `walk-room/` | First-person test room: customizable guns, swappable credit cards, target dummies, WALLET repayment games |
| `brand-npcs/` | InDebtEd Brand NPCs |
| `consumables/` | PS1-style 3D renders of all 12 consumables across the four brands |
| `systems-map/` | Interactive map of every system across one day-run and what feeds what |

## Docs

| Path | What it is |
|---|---|
| `docs/design/design-reference-v2.md` | **Design Reference**: glossary, macro loop, day generation, relationship web, market math, economy, loans, Fence & Hoard, brand headlines. Single source for market states. |
| `docs/design/ai-director-design-doc.md` | AI Director v0.2: decision loop, approaches and escalation, budget, pay-to-pass, segment modification |
| `docs/specs/market-spec.md` | Market Spec: tested tick order, nudges, thresholds, shares and simulation results |
| `docs/specs/melee-spec.md` | Melee Spec: HAND OF THE MARKET neighbor kickback and the four brand strikes |
| `docs/specs/credit-card-spec.md` | Credit Card Spec: card power, abilities, payment-failure ladder, BLU HUD unlocks |
| `docs/specs/game-mechanics-inventory.md` | Every mechanic (~110) by system with status, for review |
| `docs/specs/level-design-guide.md` | Level Design Guide: Pop Brutalism districts, color ladder, lo-fi treatment, UE5 notes |

## Assets

`assets/brand/`: brand logos, tone boards for each brand, and the general rules sheet.

## Source

Everything here was exported from Geoffrey's Claude artifacts and the "InDebtEd Production Management" project on 2026-11-20. The Design-canvas mockups (HUD directions, Tag Bench UI, gun sprites, mood boards, consumable ads, market and newspaper) are not included yet.
