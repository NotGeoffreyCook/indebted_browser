# InDebtEd — Game Mechanics Inventory

Oct 28, 2026 · @Geoffrey Cook

InDebtEd has about 110 distinct mechanics across 11 systems; every one is listed below with its current status and an empty Call column for the review.

## How to use this doc

Work section by section and fill the **Call** column with one of: **Keep**, **Flesh out**, **Trim**, **Cut**. Add a comment on any row that needs discussion.

| Status | Meaning |
| --- | --- |
| Locked | Decided and recorded as a design invariant |
| Built | Implemented in UE5 and working (may need tuning) |
| In progress | Partially built or being refactored |
| Designed | Specced on paper, not yet built |
| Exploring | Brainstormed; options exist but nothing is committed |
| Open | Known gap or unanswered question |

Statuses reflect project notes and past design sessions; some may be stale, so correct any that are wrong as you go.

## 1. Core loop and run structure

The player buys every round they fire, uses it to generate profit, and resells to restart the loop, inside short, timed roguelike levels.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 1.1 | Buy-per-round loop | Ammo is purchased per round; weapons generate profit; resell to restart the loop | Locked |  |
| 1.2 | Roguelike run | First-person roguelike; runs are a sequence of generated levels | Locked |  |
| 1.3 | Timed levels | Each level is only open for roughly 5–10 minutes | Designed |  |
| 1.4 | Level segments | Levels split into segments gated by physical checkpoints / trigger volumes | Designed |  |
| 1.5 | Arena clear gating | Arena/segment ends when spawned enemies are cleared, then the next area opens | Designed |  |
| 1.6 | Bankrupt the store | The player's goal is to make the store exhaust its level budget and take out loans | Designed |  |
| 1.7 | Performance bonus | Level-end payout; scales with the loans the store was forced to take (and the combo meter) | Designed |  |
| 1.8 | Pay-to-pass | Brief window at each segment entrance to pay a fee and walk through; fee funds final-boss upgrades | Designed |  |
| 1.9 | Final boss | End-of-level boss strengthened by pay-to-pass money; upgrade types WIP | Open |  |
| 1.10 | Bank account | The player's cash balance, reached through the default bank card | Designed |  |

## 2. Brands and the live stock market

Four brands with live Worth values (0.01–100.00) are the spine of the game: pricing, damage, durability, enemy relationships and art all flow through them.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 2.1 | Four brands | GLD (Radiance), RED (Fire), BLU (Zap), GRN (Growth); each has a palette, satirical corporate personality and mechanical identity | Locked |  |
| 2.2 | Brand Worth | Each brand carries a Worth of 0.01–100.00 that drives price, damage and durability | Locked |  |
| 2.3 | Live market reactivity | Worth moves during play; damage reads Worth live at impact, so the market can change a shot mid-flight | Locked |  |
| 2.4 | Player moves the market | Hitting / impacting a brand pushes its stock up or down (shown as stock delta popups) | Built |  |
| 2.5 | Bull / bear thresholds | Worth thresholds that mark a brand as bullish or bearish | Designed |  |
| 2.6 | Market Engine | BP\_MarketEngine owns market simulation; link to the Shop is implied but no actor is named | In progress |  |
| 2.7 | Ticker and headlines | WBP\_Ticker / BP\_TickerManager show prices; headlines come from an FHeadlineRow DataTable | In progress |  |
| 2.8 | Brand conflict / support pairs | Competing brands loaded together cause passive debuffs (spread, fire rate) and active debuffs (double pay, dud round); chance scales with number of conflict pairs | Designed |  |
| 2.9 | Counter beats support | If a target meets both a counter and a support condition, counter wins; one hit never triggers both | Locked |  |
| 2.10 | Market-assigned enemy tags | Before a level generates, enemies get brand tags from market state: conflicting brands deal bonus damage to each other, collaborating brands deal reduced or no damage | Designed |  |
| 2.11 | Randomized tag brands | Each tag gets a random brand assignment per load, so synergies are discovered rather than memorized | Designed |  |
| 2.12 | BLU illegibility | BLU is deliberately opaque, illegible and unpredictable across every system (UI, VFX, audio, behavior) | Locked |  |
| 2.13 | Brand Loyalty | Ammo, card and weapon brand makeup earns progress toward brand benefits, capped at 4 levels per brand; switching brands bleeds away the other programs' benefits. Tier effects only exist as brainstorm drafts | Exploring |  |

## 3. Gun system

The gun is a modular assembly line: five part slots decide what a round is, what it costs at the moment it is made, and how hard it hits when it lands.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 3.1 | Five part slots | Ammo Type, Frame, Chamber, Barrel, Ammo tag slots | Locked |  |
| 3.2 | Ammo types by brand | Ammo is branded; its brand's Worth sets base price and damage | Locked |  |
| 3.3 | Price per round | AmmoType.Worth × $100 + each loaded tag's Brand.Worth × $1 | Designed |  |
| 3.4 | Price frozen at spawn | PriceMultiplier is locked at the transaction moment (when the round is made) | Locked |  |
| 3.5 | Damage formula | BaseDamageEvent × Brand.Worth × PriceMultiplier × DamageMultiplier; Worth read live at impact | Designed |  |
| 3.6 | Durability tiers | Projectiles have 1 / 5 / 10 charges by Worth range; charges = max tag capacity per projectile | Designed |  |
| 3.7 | Projectile attributes | Integrity, ProjectileSpeed, ProjectileSize, PriceMultiplier, DamageMultiplier (UProjectileAttributeSet) | Built |  |
| 3.8 | Discrete projectiles only | Every ammo type resolves as a projectile with spawn, travel and hit phases; no exceptions | Locked |  |
| 3.9 | Beam barrel | Beam treated as a very fast projectile; needs a visited-target list for safe chain resolution | Designed |  |
| 3.10 | Barrel accuracy variation | Barrels get accuracy spread values (prerequisite for the BLU card) | Designed |  |
| 3.11 | No ammo category gating | Synthetik 2-style category locks rejected to keep parts freely combinable | Locked |  |
| 3.12 | Procedural weapon feel | Recoil and sway via Control Rig; per-weapon camera shake asset, intensity scalable (e.g. by ADS or market volatility) | Built |  |
| 3.13 | Parts as static meshes on a shared skeleton | Part variants socket onto named bones rather than being skinned | Exploring |  |

## 4. Projectile tags and status effects

Tags are the run's deckbuilder layer: bought in break rooms, loaded onto rounds as charges, and resolved in three phases of a projectile's life.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 4.1 | Break rooms | Rooms found mid-level where the player buys and equips one tag at a time | Designed |  |
| 4.2 | Stacking tags | Tags stack as independent instances (e.g. +5% speed, +1 bounce, +1 penetration) | Designed |  |
| 4.3 | Three resolve phases | Each tag modifier resolves at spawn, in flight, or on impact | Designed |  |
| 4.4 | Charge-tracked modifiers | Tags are FAmmoModifier structs with charge counts, because charges carry mechanical meaning | Locked |  |
| 4.5 | Tag price | Each loaded tag adds its brand's Worth × $1 to the round's price | Designed |  |
| 4.6 | 8 modifier tags | Eight core modifier tags, all stackable | Designed |  |
| 4.7 | 4 ammo status effects | One status effect per ammo brand, all stackable | Designed |  |
| 4.8 | Status stacks | Stacks applied = tags loaded × durability remaining at impact | Designed |  |
| 4.9 | Status trigger | Effect fires at 10 stacks, once per enemy per life | Designed |  |
| 4.10 | Scattershot | Splits a round into pellets; pellet charge inheritance with Compressor needs a cap | Open |  |
| 4.11 | Compressor | Named modifier; combines with Scattershot into degenerate stacking without a cap | Open |  |
| 4.12 | Flechette | Recursive sub-projectiles; needs a recursion depth cap | Open |  |
| 4.13 | BLU.Redacted | BLU tag that hides information from the player; needs UI suppression architecture | Open |  |
| 4.14 | Brand gun tag list (drafts) | 16 drafted tags, 4 per brand (GLD precision, RED volume, GRN sustain, BLU control) plus an optional 5th signature tag for brand-pure builds; brainstorm only | Exploring |  |

The exact names of the eight modifier tags and four status effects are in the production notes, not here; add them as rows if you want to judge each one separately.

## 5. Credit cards, debt and the WALLET

Each brand card grants an ability that scales with credit used; debt is repaid through brand-specific micro games on the WALLET instead of reloading.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 5.1 | Card ability scales with credit used | The more of a card's limit is used, the stronger its ability | Built |  |
| 5.2 | GLD card | Sprint / run speed; refresh planned as a sprint toggle with per-step cost scaling | Built |  |
| 5.3 | RED card | Dash | Built (needs test) |  |
| 5.4 | GRN card | Jump height | Built (needs vetting) |  |
| 5.5 | BLU card | Changed from laser sight to accuracy; still not satisfying, alternatives brainstormed (damage reduction, parry, lifesteal, slow field, i-frames) | Open |  |
| 5.6 | Bank card | Default card tied to the bank account; asks for confirmation on every purchase | Designed |  |
| 5.7 | Overcharge | Spending past 100% adds the excess as a depreciating addition; placeholder is card dim + white % over limit | In progress |  |
| 5.8 | Active repay | Active-reload-style repayment; placeholder fixed 70–80% window, meant to shift with credit usage later | In progress |  |
| 5.9 | WALLET repayment | Pull the card from the gun and jam it into the WALLET (chunky Game Boy-style console) to play a repay game in place of reloading; less frequent, higher risk/reward | Designed |  |
| 5.10 | WALLET payout rules | Each success pays a small amount; each mistake adds a small fee; playing longer repays more; auto-closes when paid off | Designed |  |
| 5.11 | APR drives difficulty | Larger balance = higher APR = harder repay game | Designed |  |
| 5.12 | RED repay game | Frogger riff; each lane advanced pays | Designed |  |
| 5.13 | GLD repay game | Auto-runner, press A to jump; each successful jump pays | Designed |  |
| 5.14 | GRN repay game | Tetris riff; each row cleared pays | Designed |  |
| 5.15 | BLU repay game | Galaga riff; each ship destroyed pays | Designed |  |
| 5.16 | In-level card leveling | Cards level up during a level via a Vampire Survivors-style level-up choice | Designed |  |
| 5.17 | Card tiers | Credit card tier system | Open |  |
| 5.18 | Interest tuning | Store Owner BaseInterestRate and LinearStep, to tune once cards are fully specced | Open |  |

Rows 5.8 and 5.9 likely overlap: WALLET games replace reloading, which may make active repay redundant.

## 6. Melee

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 6.1 | HAND OF THE MARKET | Melee weapon; prototyped and confirmed working. Its role in the economy (free damage? brand interaction?) is not written down | Built |  |

## 7. Enemies, AI Director and spawning

The AI Director is the store's mind: it picks an approach, spends a replenishing budget on enemies and traps, and reshapes unvisited parts of the level.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 7.1 | Four approaches | Kill (drain health with damage-heavy enemies), Drain (make the player overspend), Trick (stall; levels last 5–10 min), Taunt (bait over-commitment) | Designed |  |
| 7.2 | Hybrid approach selection | Each level starts on a scripted stance, then adapts to the player | Designed |  |
| 7.3 | Escalation | Starts in one mode and ramps into another as the level goes on | Designed |  |
| 7.4 | Replenishing level budget | Budget refills over time; the player races to drain it faster than it refills | Designed |  |
| 7.5 | Spawn costs | Enemies and traps each cost budget; approach shapes what the budget buys | Designed |  |
| 7.6 | Store loans | When the budget is empty and the Director still needs to spawn, it borrows; loans raise the player's performance bonus | Designed |  |
| 7.7 | Pace tracking | Compares player progress to expected progress (is the player too fast?) | Designed |  |
| 7.8 | Segment modification | Director edits unvisited segments with obstacles that counter the player's abilities or sidetrack them | Designed |  |
| 7.9 | Pay-to-pass upgrades | Fees paid at segment entrances fund final-boss upgrades | Designed |  |
| 7.10 | Enemy Manager | Tracks active enemies, enforces a concurrent cap and spawn cooldowns | In progress |  |
| 7.11 | Spawn points and rooms | Room volumes own spawn points; validity checks on distance, line of sight and nav mesh | In progress |  |
| 7.12 | Enemy pooling | Enemies reused rather than spawned and destroyed | Designed |  |
| 7.13 | Brand-aligned enemy roster | Enemies belong to brand factions with budget costs; types are WIP | Exploring |  |
| 7.14 | Market-tagged enemies | See 2.10: enemy damage relationships set from the market before the level generates | Designed |  |
| 7.15 | NPC behavior | StateTree-based behavior (death, hit react, combat, investigate, idle) planned in UE 5.6 | Exploring |  |
| 7.16 | Enemy drops | Enemy deaths feed the brand economy through drops | Exploring |  |
| 7.17 | Boss concepts | Final boss plus a "Boxer Brothers" boss idea; mechanics undefined | Open |  |

## 8. Store Owner, shop and level intelligence

The store is both the shop the player buys from and the opponent whose budget the player is trying to break.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 8.1 | Store Owner | BP\_StoreOwner (Store Manager) gives the level its intelligence; planned after improved enemies and traps | Designed |  |
| 8.2 | Shop purchases | Where ammo and parts are bought; purchases logged as FSOPurchaseEntry | In progress |  |
| 8.3 | Resell | Player resells to restart the buy-fire-profit loop | Designed |  |
| 8.4 | Store interest rates | Store Owner sets BaseInterestRate and LinearStep for credit | Designed |  |
| 8.5 | Market Engine to Shop link | Shop prices should follow the market, but no named actor connects them | Open |  |
| 8.6 | Brand-locked room slots | Modifiers that lock a room's slot assignments to a brand | Open |  |

## 9. Traps, hazards, puzzles and combos

These systems are named in the architecture but have the least written design of anything in the game.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 9.1 | Traps and hazards | Director-funded traps; trap activation drains the store's budget. Types are WIP | Open |  |
| 9.2 | Counter-obstacles | Obstacles placed to counter the player's current abilities | Designed |  |
| 9.3 | Combo meter | BP\_ComboManager tracks a combo that feeds the level-end payout | Designed |  |
| 9.4 | Logic puzzles | BP\_LogicPuzzleManager exists; no puzzle design recorded | Open |  |

## 10. Feedback systems

Feedback is where the satire is most visible: every shot prints a receipt, and every brand looks, sounds and moves differently.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 10.1 | Stock delta popups | Floating +/- stock numbers per brand when the player moves a brand; 8 pooled widgets | Built |  |
| 10.2 | Heartbeat pulse HUD | Pulse animation as a core HUD motion | Built |  |
| 10.3 | Card threshold animations | Card idle 0–25%, escalating pulses at 25, 50, 75 and 100%+ | Designed |  |
| 10.4 | Overcharge HUD state | Card pulses / dims with % over limit | In progress |  |
| 10.5 | Blister-pack receipts | Each round ejects a two-half blister pack with a receipt printed inside; fixed split axis relative to the gun | Designed |  |
| 10.6 | One pack per gun part | Barrel, chamber and handle each eject their own pack with its own line item | Designed |  |
| 10.7 | Brand material identity | RED metal, BLU clicky plastic, GLD premium and floaty, GRN cardboard; drives motion and sound | Designed |  |
| 10.8 | Ghost line | Guide line (exact function to confirm); BLU's is unstable. Open question whether a card tier gates it | Open |  |
| 10.9 | BLU noise | Jittery trails, unresolved negative tone, obscured UI | Designed |  |
| 10.10 | Volumetric smoke | CS2-style interactive smoke that bullets and explosions push around | Exploring |  |

## 11. World and level structure

Levels are Brutalist with a Pop Art twist: four monochrome brand districts whose shapes double as navigation.

| # | Mechanic | What it does | Status | Call |
| --- | --- | --- | --- | --- |
| 11.1 | Brand districts | RED industrial, flat, claustrophobic; BLU lofty and open; GLD 45° angles; GRN overlapping cubes | Designed |  |
| 11.2 | Monochrome surfaces | Every surface in a district is its brand color | Designed |  |
| 11.3 | Shade-based wayfinding | Different shades of the brand color guide pathfinding | Designed |  |
| 11.4 | District borders | Transitions are always a black line or space; outside any district is white | Designed |  |
| 11.5 | Showroom level | Demo level to validate ammo tags, guns, cards, traps and spawns with no level logic | Designed |  |

## 12. First-pass flags for the review

The biggest risk is not missing mechanics but overlapping ones: the game currently has four money systems and two repayment systems competing for the player's attention.

**Likely trims or merges**

- **Active repay (5.8) vs WALLET (5.9):** both replace or reshape reloading. Pick one, or make active repay the WALLET's quick-tap fallback.
- **Brand Loyalty (2.13) vs card leveling (5.16) vs break-room tags (4.1):** three progression ladders per run. Consider folding loyalty into card leveling.
- **Four money pools:** player bank account, card balances, store level budget, boss upgrade pool. Check the player can track all four at once in a 5–10 minute level.
- **Pay-to-pass (1.8) vs bankrupt-the-store (1.6):** paying to skip feeds the boss while the core goal is draining the store; confirm the incentives point the same way.

**Needs fleshing out before the demo**

- BLU card ability (5.5)
- Trap and hazard types (9.1) and logic puzzles (9.4)
- HAND OF THE MARKET's economic role (6.1)
- Final boss and Boxer Brothers (7.17)
- Enemy roster by brand (7.13)
- Market Engine to Shop link (8.5)
- Card tiers and interest tuning (5.17, 5.18)

**Safety caps still open**

- [ ] Compressor + Scattershot pellet charge inheritance cap
- [ ] Flechette recursion depth cap
- [ ] Beam barrel visited-target list
- [ ] BLU.Redacted UI suppression architecture
