**InDebtEd**

Design Reference  //  Systems, Math & Headlines

*Revised 2026-11-23 (GRN enemy reaction bias changed to Scatter & Flank). Previous revision 2026-11-22 (market fixes pass; Enemy Reactions section added). Momentum decay, saturating nudges, zone-boosted recovery, seeded sine phase, Worth^0.7 damage, heal speed by provider Worth, numeric Floor / Recovery thresholds, and one Worth scale with a Price Bases table. Previous revision 2026-10-30 (hygiene pass). This is the single source for market states and headlines; market_states_v3 is retired.*

**GLOSSARY**

Key terms used throughout this document. All values are runtime variables unless noted as design constants.

| **TERM** | **DEFINITION** |
| --- | --- |
| **Worth** | Float 0.01 to 1.00. The current market value of a brand and the only Worth scale in the game. Every price is Worth x a base from the Price Bases table. Drives damage output (Worth^0.7). The primary market variable. |
| **Baseline Worth** | Design constant. The resting value Worth oscillates around per brand. GLD: 0.62, RED: 0.58, BLU: 0.45, GRN: 0.40. |
| **Volatility** | Integer 0 to 5. Amplitude multiplier on the sinusoidal engine. Higher = larger price swings per tick. |
| **Momentum** | Integer -3 to +3. Directional pressure applied to Worth each tick. Positive pushes up, negative pushes down. Baseline is 0 for every brand. Market events set it to +3 / -3; it then decays one step toward 0 every 4 ticks. |
| **Recovery Rate** | Integer 0 to 5, or R (BLU random). Speed at which Worth drifts back toward Baseline. Doubled while Worth is in the Bull or Bear zone. 0 = frozen. |
| **Liquidity** | Integer 0 to 5. Controls shop ammo availability and how much player actions move the price. 0 = ammo absent. |
| **Sine Phase** | Per-brand offset (0 to 2π) rolled from MarketStream at Wake-Up so the four brands do not peak and dip together. |
| **Base Tick** | The 20-second timer cycle. Primary cadence for sinusoidal update, recovery drift, momentum decay, and Sudden Death debt increment. |
| **Event Trigger** | Immediate market update fired by player action (kill, purchase, combo change, threshold crossing). Adds on top of the base tick; never replaces it. |
| **Foreshadow Headlines** | Ticker headlines that run before a threshold is crossed. Signal to attentive players that a market event is coming. |
| **Triggered Headlines** | Ticker headlines that fire when a threshold is crossed. Reflect the brand identity spin on the market state. |
| **Pre-State** | Market metric values during the foreshadow window — the conditions heading into a threshold. |
| **Effect State** | Market metric deltas that apply instantly when a threshold is crossed. |
| **Threshold** | A Worth value that triggers a market event. Four per brand: Bull (above 0.75), Bear (below 0.25), Floor (at or below 0.05), Recovery (leaving Floor: back above 0.10, or automatically after 6 ticks at Floor). |
| **Performance Bonus** | End-of-level payout. Combo multiplier x total earnings during level. Paid to Savings at Settlement (Walk Home). Currently the only path to Savings growth; under revision, see Fence & Hoard. |
| **Sudden Death** | Player state triggered when cash goes negative after a heal bill. Next hit = instant death (which triggers the Zero HP flow). Debt accumulates each tick until cleared. Accepting a loan clears the negative balance. |
| **Combo Multiplier** | Float scaling with tier (Bronze through InDebtEd!!!). Applied to earnings and performance bonus. Degrades over time without kills. |
| **Credit Card** | Purchasable item. Grants a credit line usable in shops. Card abilities scale with % of credit used. Challenges grow the credit line. |
| **Provider** | Player-chosen brand for healing. Heal cost per HP = provider Worth x $1.00 at time of charge. Heal speed also scales with provider Worth — cheap providers heal slowly. Also prices the Medical Bill. |
| **Medical Cost** | Savings deduction when a run ends. Permanent career penalty. Amount determined by run circumstances. |
| **Nudge** | Small immediate Worth adjustment from a player action. Saturating: upward nudges are scaled by (1 − Worth), downward nudges by Worth. Kill nudge: 0.005 x Liquidity. Purchase nudge: 0.0005 x Liquidity per round bought. |
| **Day** | One run. Generated in full at Wake-Up from a single Day Seed. |
| **Day Seed** | Integer that drives all day generation. Each system derives its own sub-stream so tuning one system never shifts another. Shown on the morning paper as the edition number. |
| **Player Position** | Snapshot taken at Wake-Up: Savings, credit line size, card levels. Scales the Target Budget. Cash is not included. |
| **Brand Stance** | Symmetric relationship between two brands: Neutral, Positive, Negative, or Tentative. Rolled at Wake-Up, fixed for the day. |
| **Relationship Web** | The day's six Brand Stances. Drives combat modifiers, projectile support/conflict, market coupling, heal pricing, target hiring, and headline selection. |
| **Primary Brand** | Every enemy has exactly one. Combat stance = Stance(AmmoBrand, Enemy.PrimaryBrand). Same brand = Neutral. |
| **Target** | The day's boss (e.g. the BOXY BROS). Runs a baseline store run, then spends extra budget on a wider variety of assets. |
| **Target Budget** | Baseline + (Player Position x WealthScale) + BossUpgradePool + Target Loan. |
| **Target Article** | The single morning-paper article reporting what the Target bought. Intel for loadout planning. |
| **Loan Offer** | Two offers presented at 0 HP by the two highest-Worth brands. Accept = revive in place with a brand debuff. |
| **Loan Debuff** | Brand-flavored penalty active until the loan is repaid at Settlement. Four-tier ladder per brand. |
| **Medical Bill** | Cash cost to decline a loan and respawn at the nearest checkpoint. ProviderWorth x MaxHP x 2.0. |
| **Debt Collector** | Free mob the store can spawn while the player owes a loan. Carries the lender's brand. Garnishes cash on hit. |
| **Settlement** | Walk Home step. Loan auto-repaid from cash (any shortfall ends the run), then Performance Bonus paid to Savings. Stolen loot is also sold here. |
| **LoanTotal** | The AI Director's own borrowing when LevelBudget hits zero. Scales the player's performance bonus. Not the same as the player's Loan Offer or the Target Loan. |
| **Target Loan** | Emergency credit the Target takes at Wake-Up when over budget. Buys insurance (second phase or revive). Feeds the player's bonus on a win. |
| **Steal / Smash** | Choice on melee against a breakable object. Smash pays cash now. Steal bags the item as loot to sell at the Walk Home. |
| **Hoard** | Unsold stolen loot, carried over between days and stored in the player's apartment. One wall stretches backward as it grows. |
| **Decay** | Every looted item expires 3 days after it was stolen. Unsold items are lost. |
| **Reaction Table** | Per-enemy weighted list of reactions rolled on a hit or near-miss. Base weights from archetype, bias from brand. See Enemy Reactions. |
| **Near-Miss** | A projectile passing within its enemy-only whiz sphere of an enemy without hitting it. Triggers a reaction roll. Misses still cost money, so near-misses buy suppression. |

**IMPLEMENTATION TO-DO  //  PRIORITY ORDER**

Ordered by what is needed to close the core gameplay loop first. Items marked PROTOTYPE have a working first pass. Items marked TODO are not yet started.

| **STATUS** | **SYSTEM** | **NOTES** |
| --- | --- | --- |
| **PROTOTYPE** | **Combo system** | Kill increments counter, timer resets on kill, timer expiry breaks combo. BP_ComboManager on Player Controller. Tier thresholds: Bronze 1-4, Silver 5-9, Gold 10-19, Platinum 20-34, InDebtEd!!! 35+. |
| **PROTOTYPE** | **Ammo economy / stock market** | Sinusoidal engine, base tick timer, Worth variable per brand, per-round price output. Needs the 2026-11-22 market fixes: seeded sine phase, momentum decay, zone-boosted recovery, saturating additive nudges, Worth^0.7 damage, numeric Floor / Recovery. |
| **TODO** | **Performance bonus** | Combo multiplier x earnings tracker. Accumulates during level, paid out to Savings at Settlement. Degrades over time to discourage camping. Visible only with starter credit card at 50% usage. |
| **TODO** | **Market event system** | Threshold monitors per brand. Bull above 0.75, Bear below 0.25, Floor at or below 0.05, Recovery back above 0.10 (or 6 ticks at Floor). Each fires effect state values instantly and triggers headline rotation. |
| **TODO** | **News ticker / in-level displays** | UMG scrolling ticker. Foreshadow headlines during normal play. Triggered headlines on threshold crossing. Constant velocity scroll, pause on market events. |
| **TODO** | **Foreshadow headlines** | Pre-threshold headline rotation. Fires approximately 60s (3 ticks) before projected threshold crossing based on current momentum and trajectory. Not guaranteed — momentum can shift. |
| **TODO** | **In-level marketing assets** | Spawnable billboards, storefront banners, digital screens. Placed by level generator at tagged socket locations. Each actor pulls correct brand copy from DataTable at spawn. Tied to foreshadow state. |
| **TODO** | **Apartment computer display** | BP_ApartmentComputer in-world actor. Subscribes to GameState stock feed. Three UMG widgets: WBP_StockTicker, WBP_StockChart, WBP_NewsPanel. Now the required morning-paper / planning screen (see Day Generation). |
| **TODO** | **Credit card HUD unlocks** | Starter card gates basic HUD. 25% = crosshairs, 50% = performance bonus tracker, 75% = overhead map (circular, static radius, below crosshair), 90% = damage numbers. Card power scales with % of credit used. |
| **TODO** | **Day generation + seed** | Wake-Up sequence on GameState. One FRandomStream per subsystem derived from DaySeed. Order: Player Position → web → target → target shopping → market seed → news. |
| **TODO** | **Brand relationship web** | Six FBrandPairStance entries on GameState, GetStance(X, Y). Triangle-balance roll, BLU hidden pairs. Hooks into combat, projectile support/conflict, market coupling, heal pricing. |
| **TODO** | **Target system** | Target Budget, budget bands, hiring filtered by the web, Target Loan, boss arena composition. First target: BOXY BROS. |
| **TODO** | **Morning paper** | Relationship headlines (dueling statements) + Target Article. Rendered on WBP_NewsPanel. |
| **TODO** | **Loan + death flow** | 0 HP offer UI (two offers), four-tier brand ladders, debuff application, checkpoint respawn, Medical Bill, boss fight restart. |
| **TODO** | **Debt collectors** | Free mob. Spawn cap scaled by outstanding balance. Wage garnishment on hit. Spawns in store and boss arena. |
| **TODO** | **Enemy reaction layer** | Weighted reaction table per enemy (archetype base + brand bias), 2s per-enemy cooldown, enemy-only whiz sphere on projectiles for near-misses, EQS for out-of-sight and flank points, max 3 simultaneous repositioners. RED and GRN biases first. See Enemy Reactions. |
| **TODO** | **Settlement (Walk Home)** | Loan auto-repaid from cash, shortfall = run over, then performance bonus payout. Stolen loot sold here. No combat after leaving the store. |
| **TODO** | **Fence & Hoard** | Steal / smash choice on breakables, loot bag, Walk Home sale screen, apartment hoard with stretching wall, 3-day decay. See Fence & Hoard. |
| **TODO** | **Easter egg shrine rooms** | Two rooms under GLD. The Reliquary (games) and The Sanctuary (people). GLD diagonal geometry throughout. Reliquary: high ceiling, central altar, 9 game pedestals. Sanctuary: chapel scale, no brand glyph on back wall, 13 personal shrines. |

**MACRO LOOP  //  ONE DAY = ONE RUN**

The frame for every other system in this document. The whole day is generated at Wake-Up. No combat happens after the player leaves the store; selling loot at the Walk Home is a transaction, not gameplay.

| **#** | **STEP** | **SYSTEMS** | **NOTES** |
| --- | --- | --- | --- |
| **1** | **Wake up** | Day Seed, Player Position, relationship web, target selection, target shopping, market seed | Entire day resolves here. Nothing about the day's setup changes afterward except the BossUpgradePool. |
| **2** | **Read paper** | BP_ApartmentComputer, relationship headlines, Target Article | The planning step. Player learns the web and what the Target bought. |
| **3** | **Leave apartment** | — | Planning phase ends. |
| **4** | **Buy gear / swap parts** | Shops, gun parts, ammo tags, credit cards, heal provider | Loadout built against the Target's hires: rival ammo, allied tags, a provider friendly to the ammo brand. |
| **5** | **Buy and sell stocks** | Market engine, Worth, purchase nudges | New mechanic — see Open Questions. May be replaced by listing hoard items for sale (see Fence & Hoard). |
| **6** | **Enter store** | AI Director, LevelBudget, opening stance | Director takes over. |
| **7** | **Play through store** | Director loop, combo, healing / Sudden Death, market ticks, WALLET repayment, debt collectors, steal / smash | Core combat loop. 0 HP triggers the Zero HP flow. |
| **8** | **Pay to pass** | Segment PayWindow, BossUpgradePool | The only mid-day change to Target strength. Paying makes the boss stronger than the morning paper said. |
| **9** | **Boss fight** | Target Budget, hired brand units, Target Loan, debt collectors | Morning-paper intel gets tested. Death restarts the fight (same loan / Medical Bill rules). |
| **10** | **Walk home** | Settlement, Fence | Sell stolen loot. Loan auto-repaid from cash (shortfall ends run), then Performance Bonus → Savings. Unsold loot goes to the hoard. No chases, no combat. |
| **11** | **End day** | Savings committed, hoard ages | Savings feeds tomorrow's Player Position. Hoard items move one day closer to decay. |

**DAY GENERATION  //  WAKE-UP SEQUENCE**

Everything resolves when the player wakes up. Tentative stances do not resolve mid-run. Order matters — each step depends on the one before it.

1. Snapshot Player Position — Savings, credit line size, card levels

2. Roll the relationship web — six pairs, triangle-balance pass, hide BLU pairs

3. Select the Target and its primary brand

4. Compute Target Budget

5. Run Target shopping — fill budget bands, filtered by the web

6. Seed the market — each brand starts at Baseline Worth, Momentum 0, and a Sine Phase rolled from MarketStream

7. Generate the news — relationship headlines + Target Article

8. Morning paper displays on BP_ApartmentComputer — player plans and leaves

**DAY SEED  //  one seed, one sub-stream per system**

WebStream      FRandomStream(HashCombine(DaySeed, Hash("Web")))

TargetStream   FRandomStream(HashCombine(DaySeed, Hash("Target")))

ShopStream     FRandomStream(HashCombine(DaySeed, Hash("Shopping")))

NewsStream     FRandomStream(HashCombine(DaySeed, Hash("News")))

MarketStream   FRandomStream(HashCombine(DaySeed, Hash("Market")))

Separate streams so adding a roll to one system never shifts the results of another. Same seed always reproduces the same web, target, and headlines.

Seed source: random per run by default, or derived from the real date for a shared daily mode.

Seed is shown on the morning paper as an edition number (EDITION #48213) so days can be shared and replayed.

Same seed + different Savings = same world, different Target Budget. Every player faces the same day scaled to their career.

**BRAND RELATIONSHIP WEB**

Each pair of brands holds one symmetric stance, rolled at Wake-Up and fixed for the day. Six pairs: GLD–RED, GLD–BLU, GLD–GRN, RED–BLU, RED–GRN, BLU–GRN.

| **STANCE** | **MEANING** |
| --- | --- |
| **Neutral** | No relationship. Baseline — no modifiers. |
| **Positive** | Partnership. Soft on each other, share upside. |
| **Negative** | Rivalry. Bonus damage, profit from each other's collapse. |
| **Tentative** | Uneasy all day. Never resolves. Effects roll per hit / per shot. |

**STANCE EFFECTS**

| **STANCE** | **COMBAT (ammo vs enemy)** | **MIXED PROJECTILE (ammo + tag)** | **MARKET COUPLING** | **HEAL PROVIDER (vs ammo brand)** |
| --- | --- | --- | --- | --- |
| **Neutral** | Baseline | No interaction | None | Baseline |
| **Positive** | -20% damage. Kill refunds 25% of round cost | Support: tag +1 charge, tag price halved | Shared fate (see below) | -15% heal cost |
| **Negative** | +25% damage. Double kill nudge to ammo brand. Rival -0.01 x RivalWorth per kill | Conflict: existing passive / active debuffs | Inverse (see below) | +15% heal cost |
| **Tentative** | Damage rolls 0.8x – 1.3x per hit | Each shot rolls support or conflict | None | Baseline |

**COMBAT RESOLUTION RULES**

Only the ammo brand checks stance against the enemy: GetStance(Projectile.AmmoBrand, Enemy.PrimaryBrand). One combat stance per hit.

Tag brands only check stance against the ammo brand. Resolved at spawn as support / conflict.

Counters take priority over supports. If any edge on a hit is Negative, conflict wins.

Same-brand ammo vs enemy = Neutral.

**MARKET COUPLING  //  sympathy nudges when brand B crosses a threshold**

| **B CROSSES** | **IF A–B POSITIVE** | **IF A–B NEGATIVE** |
| --- | --- | --- |
| **Bull** | A +0.03 | A -0.02 |
| **Bear** | A -0.02 | A +0.03 |
| **Floor** | A -0.05, A Liquidity -1 | A +0.05, A Liquidity +1 |

Sympathy nudges fire with the threshold event. Like every nudge, they add on top of the tick; they are flat values (not saturating) because they come from rare threshold events, not repeated player actions.

**THE ROLL  //  triangle balance**

The four brands form four triangles (e.g. GLD–RED–GRN). A triangle is balanced with an even number of Negative edges (all friendly, or two allies sharing a rival) and unbalanced with an odd number.

1. Roll each pair using both brands' personality weights added together

2. Enforce constraints: at least 2 Negative, at least 1 Positive, at most 1 Neutral

3. Find unbalanced triangles — convert 1–2 of their edges to Tentative

4. Hide two of BLU's three pairs. One BLU pair is revealed on the morning paper; the others are revealed only through headlines

| **BRAND** | **PERSONALITY BIAS** |
| --- | --- |
| **GLD** | Leans Positive toward the brand with the highest starting Worth. Follows the money, calls it providence. |
| **RED** | Leans Negative overall. Needs enemies. |
| **BLU** | Neutral weights. Stances partially hidden — consistent with BLU illegibility everywhere. |
| **GRN** | Leans Tentative. Always announcing partnerships that never close. |

**DATA STRUCTURE**

UENUM EBrandStance { Neutral, Positive, Negative, Tentative }

USTRUCT FBrandPairStance { EBrand BrandA (lower enum index); EBrand BrandB; EBrandStance Stance; bool bRevealed (false for hidden BLU pairs) }

Six entries stored on GameState. GetStance(X, Y) sorts inputs so callers never deal with order. GetStance(X, X) returns Neutral.

**RELATIONSHIP HEADLINES  //  dueling statements**

Each pair gets alternating headlines — both brands comment on the same relationship in their own voice. 6 pairs x 4 stances x 2 voices = 48 slots, ~3 lines each.

**GLD–GRN  NEGATIVE**

**GLD  **PASTOR WARNS FLOCK AGAINST "FALSE GREEN IDOLS" // TITHE ACCORDINGLY

**GRN  **GRN "SADDENED" BY RADIANCE REMARKS // TASK FORCE FORMED

**RED–GRN  TENTATIVE**

**RED  **FIRE CEO "OPEN TO TALKS" WITH GROWTH // CALLS THEM "SOFT BUT USEFUL"

**GRN  **GRN EXPLORING VALUES-ALIGNED DEFENSE DIALOGUE // NOTHING SIGNED

**RED–BLU  NEGATIVE**

**RED  **RED CEO CALLS ZAP "SOFT ON ENEMIES" // DEMANDS HEARING

**BLU  **FOUNDER DECLINES TO ACKNOWLEDGE FIRE // "WE DO NOT SEE THEM"

**BLU  (hidden pair reveal)**

**BLU  **FOUNDER ACKNOWLEDGES "A CONNECTION" WITH RADIANCE // NATURE UNSPECIFIED

**TARGET SYSTEM  //  THE BOXY BROS**

The Target is the day's boss. It always does a baseline store run. More money buys a wider variety of assets, not just more of the same. Everything is bought at Wake-Up and reported in the Target Article.

**TARGET BUDGET**

TargetBudget = Baseline + (PlayerPosition x WealthScale) + BossUpgradePool + TargetLoan

Player Position is snapshotted at Wake-Up from Savings, credit line, and card levels. Savings cannot be spent, so the snapshot cannot be gamed. Career success attracts better-funded opposition — a built-in ascension ladder.

BossUpgradePool (pay-to-pass fees) is the only budget that grows mid-day.

**BUDGET BANDS**

| **BUDGET** | **UNLOCKS** | **ARTICLE TONE** |
| --- | --- | --- |
| **1.0x (baseline)** | Home-brand stock, standard arena | Routine coverage |
| **1.5x** | Hires — one contracted squad from another brand | BOXY BROS CONTRACT FIRE SECURITY // "PURELY PRECAUTIONARY" |
| **2.0x** | Fortifications — arena traps and hazards from a third brand | Facility upgrades, renovation permits |
| **3.0x** | Specialists — elites (e.g. snipers), multi-brand mix | BOXY BROS RETAIN ZAP "CAPABILITIES" // NATURE UNDISCLOSED |
| **Over budget** | Target Loan — insurance (second boss phase or revive) | BOXY BROS SECURE EMERGENCY CREDIT LINE // EXPANSION NON-NEGOTIABLE |

Target Loan feeds the player's performance bonus on a win, mirroring the Director's LoanTotal.

**HIRING BY STANCE  //  Target brand vs hire brand**

| **STANCE** | **HIRING** |
| --- | --- |
| **Positive** | Available at a discount. Picked first. |
| **Neutral** | Available at full price. |
| **Negative** | Refused. That brand will not contract with the Target. |
| **Tentative** | Rolled at Wake-Up: deal closes or collapses. Article reports either way. |

BLU hires are reported vaguely ("unspecified capabilities"). The player knows BLU is coming, not in what form.

Debt collectors can also spawn in the boss arena if the player owes a loan.

**TARGET ARTICLE  //  example**

**BOXY BROS  **ANNOUNCE "FLAGSHIP DEFENSE INITIATIVE" // SHAREHOLDERS SUPPORTIVE

**BOXY BROS  **ZAP SECURITY TALKS COLLAPSE AFTER RADIANCE–ZAP DISPUTE // "EXPLORING ALTERNATIVES"

**BOXY BROS  **SECURE EMERGENCY FINANCING // FIRE CONTRACTORS RETAINED

**STOCK PRICE MATH  //  WORTH CALCULATION**

All formulas operate on Worth as a float 0.01 to 1.00. This is the only Worth scale: UI may display it as 1–100, but no system stores or prices on another scale. Worth is clamped to 0.01 minimum (never true zero) and 1.00 maximum after any operation.

Tick order, every 20s per brand: 1. Sine + momentum  →  2. Recovery drift  →  3. Momentum decay  →  4. Threshold checks. Player nudges apply immediately when they happen and add on top; they never cancel or replace a tick.

**SINUSOIDAL ENGINE  //  fires every 20s base tick**

Amplitude    A = VOL x 0.02                        Range: 0.00 to 0.10

Frequency    F = 0.1 (fixed)                       1 full cycle = 10 ticks / 200s

Phase        φ = MarketStream roll, 0 to 2π         One per brand, rolled at Wake-Up

Mom. Offset  M = MOM x 0.01                        Range: -0.03 to +0.03

Sine Value   S = A x sin(2 x PI x F x t + φ)

Apply        Worth += S + M

Example: VOL 2, MOM +1  ->  A=0.04, M=+0.01, max swing +-$0.04 plus +$0.01 push per tick

Example: VOL 4, MOM -2  ->  A=0.08, M=-0.02, max swing +-$0.08 minus -$0.02 push per tick

Example: VOL 0, MOM  0  ->  A=0.00, M=0.00,  frozen — no movement

Note: S is added every tick, so Worth accumulates the sine. The per-tick change is at most ±A, but the cumulative swing over half a cycle is about 3.1 × A (VOL 2 ≈ 0.12 peak to trough), before recovery drift damps it.

Note: without φ, every brand shares t and F and moves in lockstep. The per-brand phase keeps their cycles apart, so one brand can be peaking while another dips.

**MOMENTUM DECAY  //  fires every 20s base tick after recovery**

Baseline     MOM = 0 for every brand. Brand character comes from Baseline Worth, Volatility and Recovery, not a standing lean.

Market event MOM = +3 or -3 (replaces current Momentum; the decay counter restarts)

Decay        Every 4 ticks (80s), MOM moves one step toward 0

Shock length +3 / -3 lasts 12 ticks (4 min) before Momentum is back to 0

Total push   (3 + 2 + 1) x 4 ticks x 0.01 = 0.24 Worth before recovery drift

Simulated result (no player input): a -3 event takes RED from 0.58 to about 0.39 and GRN from 0.40 into Bear (about 0.23) within 5 ticks, then both drift back to baseline within about 5 minutes. A +3 event takes GLD into Bull (about 0.84) for roughly 3 minutes.

Note: this resolves the old momentum equilibrium problem. Constant momentum used to settle Worth at Baseline + MOM / REC, which parked GLD permanently in Bull (about 0.87) and GRN near Floor (about 0.07). With decay, the long-run equilibrium is Baseline.

**RECOVERY DRIFT  //  fires every 20s base tick after sine**

Difference   D = BaselineWorth - CurrentWorth

Zone boost   REC_eff = REC x 2 if Worth > 0.75 (Bull) or Worth < 0.25 (Bear), else REC

Drift Step   Step = D x REC_eff x 0.01

Apply        Worth += Step

Skip if      REC == 0 (frozen — awaiting trigger event), or the brand is at Floor

BLU special  REC = random roll 0 to 5 each tick (the zone boost doubles the roll)

Example: Worth 0.20, Baseline 0.62, REC 3  ->  D=0.42, in Bear zone REC_eff=6, first Step=+0.025

Example: Worth 0.50, Baseline 0.62, REC 5  ->  D=0.12, normal zone REC_eff=5, Step=+0.006

Example: Worth 0.20, Baseline 0.62, REC 0  ->  frozen, no drift until recovery event fires

Note: the step shrinks as the gap shrinks, so Worth approaches baseline exponentially and never fully arrives. Half-life in ticks in the normal zone = ln 0.5 / ln(1 − REC × 0.01): REC 1 ≈ 69, REC 2 ≈ 34, REC 3 ≈ 23, REC 4 ≈ 17, REC 5 ≈ 14. In the Bull / Bear zones the half-life is roughly halved.

Note: the zone boost is the "overbought / oversold" correction. Players can still hold a brand in Bull by killing steadily, but the market pushes back harder the further it is pushed.

**EVENT NUDGES  //  fire immediately, add on top of the tick**

Saturation   Upward nudges are multiplied by (1 − Worth). Downward nudges are multiplied by Worth. Each push gets weaker the closer the brand is to the ceiling or the floor.

Kill nudge       Worth += 0.005 x Liquidity x (1 − Worth)        Ex: LIQ 4 at 0.62 = +0.0076

Purchase nudge   Worth += 0.0005 x Liquidity x Rounds x (1 − Worth)    Ex: 50 rounds, LIQ 4 at 0.62 = +0.038

Other purchase   Counts as Spend / RoundPrice rounds of that brand, capped at +0.05 per purchase

Combo tier up    Worth += 0.03 x (1 − Worth)  (active brand)

Combo break      Worth -= 0.05 x Worth  (active brand)

Rival kill       RivalWorth -= 0.01 x RivalWorth  (Negative stance only)

Threshold event  Apply full effect state instantly

All nudges stack additively and none of them suppress the base tick. If a threshold event fires in the same frame as other nudges, its effect state applies last.

Simulated steady state (45 ticks, no market events) — the ammo brand settles at:

| **KILLS PER 20s TICK** | **GLD** | **RED** | **BLU** | **GRN** |
| --- | --- | --- | --- | --- |
| **0 (no player)** | 0.57–0.73 | 0.49–0.67 | 0.43–0.50 | 0.34–0.48 |
| **1** | 0.77 | 0.68 | 0.66 | 0.65 |
| **3** | 0.85 | 0.78 | 0.80 | 0.78 |
| **7 (hordes)** | 0.93 | 0.87 | 0.89 | 0.87 |

Previous formulas pinned every brand at 1.00 within 1–12 ticks at as little as one kill per tick. Now heavy play earns a Bull market without locking it at the ceiling.

**FLOOR AND RECOVERY  //  numeric thresholds**

Floor enter     Worth <= 0.05

At Floor        Volatility 0, Momentum 0, Recovery frozen, Liquidity 0 (ammo leaves shops, kill nudges do nothing). Brands at Floor do not lend or buy.

Recovery fires  Worth rises above 0.10 (sympathy nudges, market events), OR the brand has spent 6 ticks (2 min) at Floor

Recovery effect Worth = max(Worth, 0.10) + 0.15. Momentum +2 (then decays as normal). Volatility, Recovery Rate and Liquidity return to Baseline values. Recovery headlines fire.

Simulated result: every brand forced to Floor recovers within 6 ticks and lands at about 0.25–0.36, then drifts back toward baseline.

Note: the +0.15 spike is the engine default until effect-state units are settled (see Open Questions). The brand tables' Recovery Worth values (GLD +25%, RED +30%, BLU +35%, GRN +20%) replace it once converted.

**OUTPUT FORMULAS  //  derived from Worth each tick**

Price per round    Price = AmmoWorth x $1.00 + Sum(TagWorth x $0.01)  (see Price Bases)

Damage output      Damage = MaxDamage x Worth^0.7

Heal cost per HP   HealCost = ProviderWorth x $1.00

Heal speed         HealRate = BaseHealRate x (0.25 + 0.75 x ProviderWorth)  (BaseHealRate starting value 10 HP/s)

Full heal bill     Bill = HealCost x HP_Recovered

Sudden Death debt  Debt += Bill  (each tick while cash < 0)

Performance bonus  Bonus = Sum(Earnings x ComboMultiplier)  paid to Savings at Settlement

Combo multiplier   Bronze: 1.0x  Silver: 1.5x  Gold: 2.0x  Platinum: 2.5x  InDebtEd!!!: 3.0x+

Medical Bill       Bill = ProviderWorth x MaxHP x 2.0  (decline loan, respawn at checkpoint)

Loan amount        Amount = LadderBase x LenderWorth x 1.6  (frozen at acceptance)

Loan repayment     Repay = Amount x (1 + APR)  (auto-deducted from cash at Settlement)

Projectile spec    Damage = BaseDamageEvent x Brand.Worth^0.7 x PriceMultiplier x DamageMultiplier  (Worth read live at impact; PriceMultiplier frozen at spawn)

Note: the 0.7 exponent breaks the flat cost per kill on purpose. Price rises 1:1 with Worth but damage rises more slowly, so a high-Worth brand kills faster but costs more per kill. A Bull brand (0.85) costs about 44% more per kill than a Bear brand (0.25) and kills about 2.4x faster.

**PRICE BASES  //  one Worth scale, one table of bases**

Every price in the game is Worth (0.01–1.00) x a base from this table. No system uses a different Worth scale.

| **ITEM** | **FORMULA** | **AT WORTH 0.62** | **NOTES** |
| --- | --- | --- | --- |
| **Ammo round** | AmmoWorth x $1.00 | $0.62 | Replaces the projectile spec's AmmoType.Worth x $100 |
| **Loaded tag** | TagWorth x $0.01 per tag per round | $0.0062 | Keeps the old 100 : 1 ammo-to-tag ratio |
| **Heal** | ProviderWorth x $1.00 per HP | $62 per 100 HP | Stance modifier ±15% on top |
| **Share** | Worth x $10.00 | $6.20 | Stock trading (loop step 5) |
| **Card ability charge** | Worth x $10.00 per use | $6.20 | RED dash, GRN jump, BLU Zap. GLD sprint is charged per step — see Open Questions |
| **Loan** | LadderBase x LenderWorth x 1.6 | $50–$496 | Unchanged |
| **Punchables** | ItemBase x Worth | TBD | Item bases to be set |
| **Loot sale (proposed)** | ItemBase x Worth at Settlement | TBD | See Fence & Hoard |

Reference wallet: $250 starting card limit, $1000 BANK card in testing. At these bases a baseline engagement (5 enemies) costs about $12 and a full 100 HP heal about $62, so healing is the larger money sink.

**CLAMP  //  applied after every operation**

Floor   Worth = max(Worth, 0.01)   never true zero — prevents free ammo / zero damage

Ceiling Worth = min(Worth, 1.00)   cannot exceed 100%

**METRIC SCALES  //  NUMERICAL KEY**

All market metrics use fixed numerical scales. Labels in the threshold tables map to these values. BLU recovery rate is a special case — treated as a random roll each cycle rather than a fixed value.

**VOLATILITY  //  amplitude multiplier on sinusoidal engine**

| **VALUE** | **LABEL** | **MEANING** |
| --- | --- | --- |
| **0** | Zero | Frozen — no movement |
| **1** | Very low | Barely perceptible drift |
| **2** | Low | Gentle oscillation |
| **3** | Medium | Noticeable swings |
| **4** | High | Aggressive movement |
| **5** | Very high | Chaotic — hard to predict |

**MOMENTUM  //  directional pressure on worth per tick**

Baseline 0. Set to ±3 by market events, then decays one step toward 0 every 4 ticks.

| **VALUE** | **LABEL** | **MEANING** |
| --- | --- | --- |
| **-3** | Negative, steep | Accelerating hard downward |
| **-2** | Negative, building | Building downward pressure |
| **-1** | Negative, slow | Gentle downward drift |
| **0** | Neutral / Flat | No directional pressure |
| **+1** | Positive, slight | Gentle upward drift |
| **+2** | Positive, steady | Building upward pressure |
| **+3** | Positive, aggressive | Accelerating hard upward |

**RECOVERY RATE  //  speed of drift back to baseline after event**

Doubled while Worth is in the Bull (> 0.75) or Bear (< 0.25) zone.

| **VALUE** | **LABEL** | **MEANING** |
| --- | --- | --- |
| **0** | Frozen | No recovery — awaiting trigger event |
| **1** | Very slow | Barely recovering |
| **2** | Slow | Below normal correction speed |
| **3** | Normal | Standard baseline drift |
| **4** | Fast | Above normal correction |
| **5** | Very fast | Snaps back to baseline quickly |
| **R** | Random (BLU only) | Rolled each cycle — value unknown to player |

**LIQUIDITY  //  shop availability and price responsiveness to player**

| **VALUE** | **LABEL** | **MEANING** |
| --- | --- | --- |
| **0** | Zero / Near zero | Ammo absent from shops entirely |
| **1** | Draining | Ammo scarce, player market impact minimal |
| **2** | Reduced | Below normal availability and responsiveness |
| **3** | Normal | Standard shop stock and price response |
| **4** | High | Above normal availability — player moves price more |
| **5** | Very high | Flooded — cheap, abundant, highly reactive |

**MARKET CADENCE  //  UPDATE SCHEDULE**

Hybrid system — base timer tick every 20 seconds with event triggers firing immediately on top. The 20s base tick is the starting point and should be tuned once playable. 15s makes the market feel reactive, 30s makes it feel like a tide.

| **EVENT** | **CADENCE** | **TYPE** | **NOTES** |
| --- | --- | --- | --- |
| **Base market tick** | Every 20s | Timer | Core sinusoidal cycle. Worth shifts, momentum nudges, recovery pulls toward baseline. |
| **Sinusoidal worth update** | Every tick | Timer | Volatility value determines amplitude of each shift. Per-brand phase. |
| **Momentum decay** | Every 4 ticks | Timer | Momentum moves one step toward 0. Counter restarts when a market event sets Momentum. |
| **Liquidity recalculation** | Every tick | Timer | Shop inventory and price responsiveness updates on each tick. |
| **Sudden Death debt increment** | Every tick | Timer | While insolvent, debt grows once per 20s tick. 20 seconds between increments. |
| **Kill / purchase nudge** | Immediate | Event | Fires instantly and adds on top. Never replaces or cancels the next base tick. |
| **Combo tier change** | Immediate | Event | Tier up or break fires market signal immediately. |
| **Threshold crossed** | Immediate | Event | Full effect state values applied instantly. Triggered headlines fire. Relationship sympathy nudges fire on paired brands. |
| **Floor auto-recovery** | 6 ticks at Floor | Timer | Recovery fires if the brand has not climbed above 0.10 on its own. |
| **Foreshadow window opens** | ~60s before threshold | Predictive | 3 ticks before projected crossing. Momentum can still shift — not guaranteed. |
| **Day generation** | Once, at Wake-Up | Seeded | Relationship web, Target, Target shopping, market seed (incl. sine phases), news. Fixed for the day. |

Base tick cadence: 20s default. Tune between 15s (reactive) and 30s (tidal) based on playtest feel. Momentum decay (4 ticks) and Floor auto-recovery (6 ticks) are counted in ticks, so they scale with cadence.

Event nudges add on top of the tick for that brand only — other brands unaffected.

Foreshadow window is predictive, not guaranteed. Momentum shift can cancel it before crossing.

Sudden Death debt increments on the same 20s tick. Player has one full tick window to earn before next increment.

BLU recovery rate (R) is rolled fresh each tick. Min 0, max 5. Player cannot predict BLU recovery timing.

**BASELINE MARKET STATE**

Resting state values the sinusoidal engine oscillates around. Each brand reflects its satirical identity in its baseline position.

| **METRIC** | **GLD** | **RED** | **BLU** | **GRN** |
| --- | --- | --- | --- | --- |
| **Worth** | 62% | 58% | 45% | 40% |
| **Price / round** | $0.62 | $0.58 | $0.45 | $0.40 |
| **Volatility** | Low (2) | Medium (3) | Very low (1) | Low (2) |
| **Momentum** | Neutral (0) | Neutral (0) | Neutral (0) | Neutral (0) |
| **Recovery rate** | Fast (4) | Very fast (5) | Unknown (R) | Medium (3) |
| **Liquidity** | High (4) | High (4) | Controlled (3) | High (4) |
| **Sine phase** | Seeded | Seeded | Seeded | Seeded |

**GLD  **Sits above midpoint. Congregation believes things are always ascending. Room to crash but baseline confidence is high.

**RED  **Propped up by government contracts. Volatile but rarely truly low. Bureaucracy provides a stable floor.

**BLU  **Deliberately opaque at midpoint. The market cannot read it. Every move feels surprising.

**GRN  **Perpetually underperforming its own projections. Looks stable on paper, sits lowest of the four, and has the slowest recovery, so dips linger.

**DAMAGE AND ECONOMY SCALE**

Worth translates directly to per-round cost (62% = $0.62/round). Damage scales with Worth^0.7. Standard engagement: 5v1. Base enemy requires 4 body hits or 2 headshots at GLD baseline (0.62).

| **STATE** | **PRICE / RND** | **DAMAGE** | **BODY HITS** | **HEADSHOTS** | **COST / ENG.** |
| --- | --- | --- | --- | --- | --- |
| Ceiling (100%) | $1.00 | 1.00x | ~3 | 1-2 | $14.31 |
| Bull (85%) | $0.85 | 0.89x | ~3 | 2 | $13.63 |
| GLD base (62%) | $0.62 | 0.72x | 4 | 2 | $12.40 |
| RED base (58%) | $0.58 | 0.68x | 4 | 2 | $12.15 |
| BLU base (45%) | $0.45 | 0.57x | 5 | 2-3 | $11.26 |
| GRN base (40%) | $0.40 | 0.53x | 5-6 | 3 | $10.87 |
| Bear (25%) | $0.25 | 0.38x | 7-8 | 4 | $9.44 |
| Near floor (10%) | $0.10 | 0.20x | ~14 | 7 | $7.17 |

Damage multiplier is relative to a Worth 1.00 round. Relationship stance modifies damage on top (Negative +25%, Positive -20%, Tentative 0.8x–1.3x).

Cost per engagement now rises with Worth (~$7 near floor to ~$14 at ceiling) while time to kill falls. High Worth = fast and expensive. Low Worth = cheap and slow, with longer exposure and more damage taken.

The trade is speed versus cost, and healing prices the exposure: a cheap brand saves on ammo but loses time and HP, and a cheap provider heals slowly.

Dynamic damage means combo multiplier compounds both economy and combat effectiveness simultaneously.

**PLAYER ECONOMY  //  FOUR LAYERS**

Four distinct financial layers, each with different risk profiles, accessibility, and purpose.

**CASH  **Liquid and spendable. Earned in level through kills and smashing objects (melee only), or borrowed via a loan at 0 HP. Used for ammo, parts, cards, healing, Medical Bills, and loan repayment. Lost entirely when the run ends. Treat it aggressively — holding it is wasteful, losing it is pure loss — but a borrower needs enough on hand at Settlement.

**CREDIT  **Extended purchasing power tied to card challenges. Credit line grows as challenges are completed. Card abilities scale with how much credit has been used — spending credit is also leveling up the card. Shop economy only. Spending and progression are the same action.

**SAVINGS  **Career score. Persists across sessions. Cannot be withdrawn. Currently grows only through the performance bonus (combo multiplier x earnings, paid at Settlement); fencing stolen loot is intended to become the key growth path (see Fence & Hoard). Drained by medical costs when a run ends. Also sets tomorrow's Player Position — higher Savings means better-funded Targets.

**HOARD  **Stolen loot not yet sold. Persists between days in the player's apartment, where one wall stretches backward to fit it. Every item decays 3 days after it was stolen. Converts to value only when sold at the Walk Home.

**INCOME SOURCES**

| **SOURCE** | **DESTINATION** | **NOTES** |
| --- | --- | --- |
| Enemy kills | Cash | Primary income source |
| Smashing objects | Cash | Melee only. The alternative is stealing the object. |
| Stolen loot | Sold at Walk Home | Destination of proceeds TBD (see Fence & Hoard) |
| Loan | Cash | Accepted at 0 HP. Repaid from cash at Settlement with APR. |
| Performance bonus | Savings | Combo multiplier x earnings. Paid at Settlement. Current path to Savings growth; see Fence & Hoard. |
| Card challenges | Credit line | Grows card credit and ability power simultaneously |

**HEALING SYSTEM**

Shield recharges automatically after damage (Halo-style). Recharge is not free — cost is deducted from cash when the player reaches 100% HP. Cost per HP equals the current stock % of the player chosen heal provider. Recharge speed also follows the provider's Worth: HealRate = BaseHealRate x (0.25 + 0.75 x ProviderWorth).

**  NORMAL STATE  **

Shield regens after damage. Cash deducted at 100% HP. Cost = HP recovered x provider stock %. Play continues.

**  INSOLVENT STATE  **

Regen completes but cash cannot cover the bill. Cash goes negative. Sudden Death activates immediately.

**  SUDDEN DEATH  **

Any damage taken = 0 HP. No exceptions. Debt grows by the bill every 20s market tick, even without taking hits. State persists until cash returns to zero or above, or a loan is accepted.

**  RECOVERY  **

Player earns enough cash to clear the negative balance. Sudden Death lifts. Normal state resumes.

**  ZERO HP  **

Loan offers appear. Accept = revive in place. Decline = pay Medical Bill and respawn at checkpoint, or the run ends if cash cannot cover it. See LOANS & DEATH.

**  RUN ENDS  **

Cash lost entirely. Savings charged medical costs. Savings persist but reduced.

**HEAL COST AND SPEED BY PROVIDER STOCK STATE**

BaseHealRate starting value: 10 HP/s.

| **STATE** | **STOCK %** | **COST / HP** | **100 HP FULL** | **HEAL RATE** | **100 HP TIME** | **RISK LEVEL** |
| --- | --- | --- | --- | --- | --- | --- |
| Bull (85%) | 85% | $0.85 | $85.00 | 8.9 HP/s | ~11s | Insolvency risk high |
| GLD base (62%) | 62% | $0.62 | $62.00 | 7.2 HP/s | ~14s | Moderate |
| GRN base (40%) | 40% | $0.40 | $40.00 | 5.5 HP/s | ~18s | Lower cost — slower to get back into fights |
| Bear (25%) | 25% | $0.25 | $25.00 | 4.4 HP/s | ~23s | Cheap but slow — long exposure at low HP |
| Near floor (10%) | 10% | $0.10 | $10.00 | 3.3 HP/s | ~31s | Cheapest heals — slowest recharge, fragile for half a minute |

Provider is player-chosen at start of level. Heal cost and heal speed track that brand stock in real time.

Provider stance vs the player's ammo brand modifies heal cost: Positive -15%, Negative +15%.

Provider also prices the Medical Bill — provider choice affects what death costs.

Sudden Death debt grows every 20s market tick — hiding does not stop the clock.

Cheap provider = slower debt accumulation in Sudden Death, but slower recovery between fights. Provider choice is a cost-versus-tempo decision, not a pick-the-lowest-brand default.

The performance bonus is currently the only path to Savings growth; fencing stolen loot is planned to overtake it (see Fence & Hoard). Combo maintenance still scales the bonus.

**LOANS & DEATH  //  ZERO HP FLOW**

Hitting 0 HP is a financial decision, not a fail state.

0 HP  ->  Two loan offers appear

ACCEPT   ->  Revive in place. Debuff applied. Loan cash added (clears any Sudden Death debt).

DECLINE  ->  Cash >= Medical Bill  ->  Pay bill, respawn at nearest checkpoint (boss fight restarts)

DECLINE  ->  Cash < Medical Bill   ->  Run ends

**LOAN RULES**

Offers come from the two highest-Worth brands — brands with capital lend. One offers a lower tier, one a higher tier.

Brands at Floor (Worth <= 0.05) do not lend (no liquidity).

Amount = LadderBase x LenderWorth x 1.6, frozen at acceptance (same rule as PriceMultiplier — locked at the transaction moment).

APR by tier (starting values): Tier 1 10%, Tier 2 15%, Tier 3 25%, Tier 4 40%.

Debuffs last until the loan is repaid at Settlement.

A second loan stacks a second debuff. A repeat loan from the same brand uses the next tier up on its ladder.

**LOAN AMOUNT BY LENDER STATE**

| **LENDER STATE** | **MULTIPLIER** | **TIER 1** | **TIER 2** | **TIER 3** | **TIER 4** |
| --- | --- | --- | --- | --- | --- |
| Ladder base | 1.00x | $50 | $120 | $250 | $500 |
| Bull (85%) | 1.36x | $68 | $163 | $340 | $680 |
| GLD base (62%) | 0.99x | $50 | $119 | $248 | $496 |
| GRN base (40%) | 0.64x | $32 | $77 | $160 | $320 |
| Bear (25%) | 0.40x | $20 | $48 | $100 | $200 |
| Floor | — | No offer | No offer | No offer | No offer |

**GLD  //  RADIANCE  —  faith-based lending**

| **TIER** | **BASE** | **DEBUFF** | **EFFECT** |
| --- | --- | --- | --- |
| **1** | $50 | Tithe | 10% of all cash earned goes to the church |
| **2** | $120 | Walk by Faith | Hit markers and damage numbers removed — trust, don't verify |
| **3** | $250 | Spiritual Audit | All heals billed at GLD Worth +25%, regardless of provider |
| **4** | $500 | Covenant | Non-GLD ammo has a 20% dud chance. You are exclusive now. |

**RED  //  FIRE  —  wartime lending**

| **TIER** | **BASE** | **DEBUFF** | **EFFECT** |
| --- | --- | --- | --- |
| **1** | $50 | Mandatory Overtime | Combo timer 20% shorter |
| **2** | $120 | Rules of Engagement | Shield regen only starts after you land a kill |
| **3** | $250 | Wartime Pricing | All ammo costs +30% |
| **4** | $500 | Conscription | All enemies deal +25% damage to you. RED enemies deal double. |

**BLU  //  ZAP  —  undisclosed terms**

| **TIER** | **BASE** | **DEBUFF** | **EFFECT** |
| --- | --- | --- | --- |
| **1** | $50 | Terms Withheld | Repayment amount hidden until Settlement, rolled 1.0x – 1.6x |
| **2** | $120 | Signal Loss | Worth readouts and ghost line show ?? |
| **3** | $250 | Latent Clause | A random debuff from another brand's ladder activates at an unknown time |
| **4** | $500 | The Void | Ammo count and minimap removed. Repayment hidden. |

BLU escalates by removing information, not by hitting harder — consistent with BLU illegibility everywhere.

**GRN  //  GROWTH  —  sustainable lending**

| **TIER** | **BASE** | **DEBUFF** | **EFFECT** |
| --- | --- | --- | --- |
| **1** | $50 | Carbon Offset Fee | Every round fired costs an extra $0.05 |
| **2** | $120 | Two-Day Delivery | Shop purchases arrive one market tick (20s) after buying |
| **3** | $250 | Resource Rationing | Magazine size -30% |
| **4** | $500 | Task Force Oversight | Combo multiplier capped at Silver (1.5x) |

**DEBT COLLECTORS**

A free mob — costs the AI Director no LevelBudget. The player's debt pays for them.

Spawn only while a loan is active. Carry the lender's brand, so the relationship web applies (ammo that is a rival of the lender does bonus damage).

Spawn pressure scales with outstanding balance via a spawn cap. Tier 1 debt = an occasional collector. Tier 4 debt = present in most encounters.

Wage garnishment: a collector hit takes cash and applies it to the loan. Getting hit pays debt down but drains cash needed for heals.

Best fit for Director Drain and Taunt approaches — placed near shops and pickups.

Multiple lenders = multiple crews. If the lenders are rivals in the web, their collectors can fight each other.

Can spawn in the store and the boss arena. Never after the player leaves the store.

**ENEMY REACTIONS  //  HIT AND NEAR-MISS**

Status: direction set 2026-11-22. RED biases confirmed; GRN bias confirmed 2026-11-23 (Scatter & Flank); GLD and BLU biases proposed. All numbers are starting values.

Enemies react at random to being shot or shot at, so fights vary run to run without feeling scripted. Each enemy rolls on a weighted reaction table when it is hit or near-missed. Base weights come from its archetype; its primary brand adds a bias on top. Brand biases let players learn patterns; the roll keeps any single fight unpredictable.

**RULES**

Trigger: a hit, or a near-miss. Near-miss = an enemy inside a projectile's enemy-only whiz sphere as it passes during the travel phase (works because every round is a discrete projectile).

Cooldown: one reaction roll per enemy every 2s, so it does not react to every bullet.

Tells: every reaction has a readable animation / bark so the player can learn brand patterns.

Repositioning cap: at most 3 enemies breaking sightline or flanking at once, or the whole room scatters.

Flankers ignore Hesitate traffic (they already push through crowds).

The existing roll / dive away flavor stays pure flavor with no economic cost to the player.

Economy: misses still cost money, but near-misses now buy suppression. Spraying a crowd becomes a deliberate spend, not pure waste.

**REACTIONS**

| **REACTION** | **TRIGGER** | **EFFECT** | **NOTES** |
| --- | --- | --- | --- |
| **Hesitate** | Hit or near-miss | Slow to ~40% speed for 1–1.5s | Mostly fodder. Fodder leads the push, so larger enemies behind get stuck in traffic. |
| **Break Sightline** | Hit or near-miss | Move to the nearest point out of the player's view (EQS, scoring offset to the side of the player's aim), then return from that angle | Mid-tier. Builds on the roll / dive flavor and gives it a goal. Counts toward the 3-enemy cap. |
| **Push Forward** | Hit or near-miss | Advances instead of slowing | Anchor reaction. GLD pushes forward with fodder; RED pushes alone. |
| **Adrenaline** | Hit only | Stacking move-speed and attack-rate buff | RED signature. Punishes chip damage; rewards focused takedowns. |
| **Retaliate** | Near-miss | Turns and sprints at the shooter | RED only. See RED detail. |
| **Scatter & Flank** | Hit or near-miss | Breaks left or right off the line of fire, then paths a flanking route and re-engages from the player's side | GRN signature. Counts toward the 3-enemy cap. See GRN detail. |
| **Roll / dive** | Hit or near-miss | Evasive roll away | Existing flavor reaction. No economic cost. |

**BRAND BIASES  //  added to archetype base weights**

| **BRAND** | **BIAS** | **STATUS** |
| --- | --- | --- |
| **GLD** | Pushes forward when shot ("doubt is the enemy") and drags fodder along. The opposite of Hesitate. | Proposed |
| **RED** | Shooting it makes it angrier. Adrenaline and Retaliate heavy; rarely hesitates or hides. | Confirmed — see RED detail |
| **BLU** | No bias. Rolls the full table flat, so its reactions are deliberately hard to predict (enemy-side illegibility; tag customization stays legible). | Proposed |
| **GRN** | Scatter & Flank: when shot at, GRN enemies scatter to their left and right and seek flanking routes, re-engaging from the player's sides. Splits groups apart instead of clustering them. Replaces the earlier Task Force regroup proposal. | Confirmed — see GRN detail |

**RED  //  FIRE  —  reaction biases**

Design intent: shooting RED makes it angrier. These weight adjustments add on top of each archetype's base table.

| **REACTION** | **RED BIAS** | **WHY** |
| --- | --- | --- |
| **Adrenaline** | +40 | Signature reaction. Every hit becomes fuel. |
| **Retaliate** | +25 | On a near-miss, turns and closes on the shooter instead of flinching. |
| **Push Forward** | +10 | Shares GLD's reaction, but pushes alone rather than carrying fodder. |
| **Break Sightline** | −20 | RED does not hide. When it does, it comes back faster. |
| **Hesitate** | −30 | Almost never slows. RED fodder makes a poor screen for the enemies behind it, which is also why it is dangerous. |

**ADRENALINE  //  starting values**

Per stack   +15% move speed, +10% attack rate

Max         3 stacks (+45% move speed, +30% attack rate)

Duration    4s per stack; each new hit refreshes the timer

At max      Ignores Hesitate entirely, including from near-misses

Tell        Coolant-cyan (#7FF3FF) glow that intensifies per stack; steam-vent burst at max stacks

**RETALIATE  //  starting values**

Trigger     Near-miss

Effect      Locks onto the shooter and sprints at +25% speed for 2s

Window      Cannot fire while sprinting — a short punish window for the player

Cooldown    5s, so it never gets stuck chasing

**STANCE INTERACTIONS**

| **STANCE vs RED** | **EFFECT ON REACTIONS** |
| --- | --- |
| **Negative** | +25% damage shortens RED's time to stack. Rivalry days turn Adrenaline from a threat into a burst-damage reward. |
| **Positive** | −20% damage means more hits per kill, so RED hits max stacks more often. A soft penalty for using friendly ammo on RED. |
| **Tentative** | The per-hit damage roll (0.8x–1.3x) makes stack speed unpredictable. |

Player read: RED cannot be suppressed and gets worse when chipped. Kill it fast or leave it alone, and save cheap spray for GLD crowds.

**GRN  //  GROWTH  —  reaction biases**

Design intent: shooting at GRN splits it. A shot GRN group breaks apart to its left and right and comes back at the player from the sides. Corporate framing: diversifying exposure, then circling back.

| **REACTION** | **GRN BIAS** | **WHY** |
| --- | --- | --- |
| **Scatter & Flank** | +40 | Signature reaction. Fire into GRN and it spreads out around you. |
| **Break Sightline** | +10 | Shares the goal of getting out of the line of fire; used when no flank route is found. |
| **Hesitate** | −15 | GRN fodder scatters rather than stalling, so it screens poorly for enemies behind it. |
| **Push Forward** | −25 | GRN does not take a head-on push. It goes around. |

**SCATTER & FLANK  //  starting values**

Trigger     Hit or near-miss

Scatter     Short lateral break (~3–5 m) perpendicular to the player's aim line. Side is chosen away from the aim point; when several GRN enemies react together, sides alternate so the group splits left and right (a pincer).

Flank       EQS query for a reachable point roughly 60–120° off the player's facing, preferring cover or out-of-view positions. Paths there, then re-engages from that side.

Fallback    No valid flank point within ~1.5s of searching: falls back to Break Sightline.

Cap         Counts toward the 3-enemy repositioner cap. GRN enemies over the cap scatter only (lateral break, no flank route).

Cooldown    6s per enemy before it can flank again.

Tell        Lateral hop with a Bloom-magenta (#FF3DCB) flash in shadow zones; corporate bark on scatter ("Let's take this offline") and on re-engage ("Circling back").

Distinct from the Flanker archetype: GRN flankers re-engage normally from the side. They do not get the Flanker's look-at-it-or-it-charges threat or push through crowds.

Player read: shooting into a GRN group splits it to your sides. Area damage pays off on the opening shot, before the group breaks; after that, track the sides or put your back to a wall.

**AI DIRECTOR HOOK  //  proposed**

Director approach tilts reaction weights: Trick favors Break Sightline and GRN Scatter & Flank, Kill favors Adrenaline, Drain favors Hesitate (making the player spend more on suppression).

**SETTLEMENT  //  WALK HOME**

No combat after leaving the store. Current steps, in order (where the loot sale sits in this order is open — see Fence & Hoard):

1. Loan repayment — Amount x (1 + APR) deducted from cash automatically. Covered = debuffs clear. Any shortfall = run ends.

2. Performance bonus — paid to Savings.

Paying off exactly to $0 counts as covered.

**FENCE & HOARD  //  STOLEN LOOT**

Status: direction set, details open. Stolen loot is intended to be highly profitable and the key path to growth.

**DECIDED**

Steal or smash: melee on a breakable object either smashes it for cash now or steals it as loot.

Stolen items are sold at the Walk Home.

Unsold loot carries over between days and is stored in the player's apartment. One wall stretches backward to make room for the growing hoard.

Every item decays: 3 days to sell before it is gone.

**PROPOSED  //  not yet confirmed**

| **AREA** | **PROPOSAL** |
| --- | --- |
| **Sale price** | Item base value x that brand's Worth at Settlement. Pumping a brand during the run raises your take-home. |
| **Selling the hoard** | List hoard items on BP_ApartmentComputer in the morning; they sell at Settlement at live Worth. Could replace loop step 5 (stock trading). |
| **Decay curve** | Stepped rather than a cliff: 100% / ~70% / ~40% / gone, so each morning is a hold-or-sell decision. |
| **Buyers** | Set by the relationship web: a brand pays a premium for a rival's goods, a discount for a partner's. Brands at Floor don't buy. BLU's offer hidden until accepted. |
| **Settlement order** | Fence → loan repayment from cash + proceeds → remaining proceeds x combo multiplier + performance bonus → Savings. |
| **Player Position** | Hoard counts at a fraction of its post-decay value so wealth can't be hidden from Target scaling. |
| **Risk while carrying** | Limited bag slots. Collectors garnish items when cash is out. Lenders take the freshest item as collateral. Medical Bill respawn drops the bag. |
| **Run-end penalty** | Asset seizure: unpaid medical costs are covered by repossessing the most valuable hoard items. |
| **Director** | Add CarriedValue to PlayerSnapshot. Taunt places high-value items as bait; Drain targets carriers. |
| **The wall** | One shelving bay per ~10 items, grouped by brand. Decayed items leave dust outlines; wall length tracks lifetime haul. |
| **Brand flavor** | GLD items lose their blessing, RED gear is decommissioned, GRN goods are biodegradable, BLU expiry is hidden. |

**OPEN QUESTIONS**

| **TOPIC** | **QUESTION** |
| --- | --- |
| **Stock trading (loop step 5)** | Do shares sell for cash or pay into Savings? May be superseded by listing hoard items (see Fence & Hoard). |
| **Stock trading timing** | Trade only before the store (planning decision) or mid-run too (market manipulation via kills)? |
| **Stock trading + Player Position** | Do held shares count toward tomorrow's Target Budget? |
| **Market clock** | Does the market tick during steps 3–5, or start when the player enters the store? |
| **WALLET repayment** | Usable outside the store (before the boss), or only inside? |
| **Target identity** | Does each Target have a primary brand, or is it a non-brand entity that hires all its muscle? |
| **Tuning** | Medical Bill multiplier (2.0x), APR bands, stance damage values, WealthScale, collector spawn cap. All starting values. |
| **Market fix tuning** | New starting values from the 2026-11-22 pass: damage exponent 0.7, kill nudge 0.005, purchase nudge 0.0005 per round, other-purchase cap +0.05, momentum decay every 4 ticks, Bull / Bear recovery x2, Floor 0.05, Recovery 0.10 / 6 ticks / +0.15 spike, BaseHealRate 10 HP/s and 25% minimum heal speed. Validated in a numerical sim only — retune in play. |
| **Enemy reaction tuning** | Archetype base weights not yet set. Starting values to retune in play: 2s reaction cooldown, whiz sphere radius, Hesitate 40% / 1–1.5s, repositioner cap 3, RED Adrenaline and Retaliate values, GRN scatter distance / flank angle / 6s flank cooldown. Confirm GLD / BLU biases. |
| **Fence proceeds** | Do sale proceeds land in cash, Savings, or both? Where does the sale sit relative to loan repayment? |
| **Morning funding** | Cash is lost at day end and Savings can't be spent. What pays for step 4 purchases besides credit? |
| **Purpose of Savings** | Higher Savings only raises Target budgets. What does it unlock or lead to — a long-term goal or end state? |
| **Credit card overnight** | Settlement covers loans only. What happens to an unpaid card balance at day end (APR, carry-over, Player Position)? |
| **Effect-state units** | Is "+20% Worth" absolute (+0.20) or relative (x1.2)? "+15% Volatility" doesn't fit the integer 0–5 scale. Effect states need concrete deltas. Until then, Recovery uses the engine default spike (+0.15). |
| **Threshold events vs market events** | Bull / Bear effect states (e.g. GLD Bull +20% Worth) would push a brand straight to the ceiling. Decide whether threshold crossings still apply Worth spikes, or only fire headlines and coupling now that the four daily market events drive Momentum. |
| **GLD sprint cost** | Card abilities are Worth x $10 per use, but GLD sprint is charged per step. At $6.20 a step it drains a card in ~40 steps. Needs a per-second or per-step base of its own. |
| **Ammo brand for stance** | Heal pricing uses "the ammo brand," but a loadout can carry several. Equipped brand at time of charge, or most-fired? |
| **Undefined values** | Run-end medical cost, performance bonus decay rate, and whether gun parts / tags persist between days. |
| **Loan naming** | Player loan, Director LoanTotal and Target Loan all feed the bonus. Consider renaming the latter two (e.g. Director overdraft, Target credit line). |

**GLD  //  RADIANCE**

**  BULL  //  ABOVE 75%  **

Congregation ascending — PR is euphoric, almost too loud

**FORESHADOW**

**GLD  **RADIANCE QUARTERLY STRONG // PASTOR CREDITS "FOCUSED PRAYER AND DIVERSIFIED PORTFOLIO"

**GLD  **CONGREGATION GROWTH UP 18% // DOUBTERS ENCOURAGED TO RECONSIDER THEIR POSITION

**GLD  **HEALING DEMAND OUTPACES SUPPLY // WAITLIST NOW SPIRITUALLY BINDING

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Rising — 60% to 75% | +20% immediate spike |
| **Volatility** | Low — suppressed by confidence | +15% — euphoria creates instability |
| **Momentum** | Positive, accelerating | Positive, strong |
| **Recovery rate** | Fast — congregation buying dips | Very fast |
| **Liquidity** | High — demand increasing | Very high — flood of activity |

— THRESHOLD CROSSED —

**TRIGGERED**

**GLD  **PASTOR CONFIRMS: THE MARKET CHOSE US // TITHE RESPONSIVELY

**GLD  **RADIANCE ASCENDING // THE FAITHFUL WERE RIGHT ALL ALONG

**GLD  **HEALING ARENA SOLD OUT THROUGH Q4 // GLORY IS SCALABLE

**GLD  **GOD'S PORTFOLIO OUTPERFORMS SECTOR // ANALYSTS HUMBLED

**  BEAR  //  BELOW 25%  **

Stock sliding — church is confident, visibly managing

**FORESHADOW**

**GLD  **RADIANCE REMAINS STRONG // MINOR FLUCTUATIONS "NOT A CONCERN"

**GLD  **PASTOR URGES CALM // "SHORT-TERM THINKING IS A SIN"

**GLD  **GLD HOLDS STEADY // CONGREGATION REMINDED THAT FAITH IS NOT A HEDGE FUND

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Declining — 40% to 25% | -30% immediate |
| **Volatility** | Low — market suppressing reaction | +40% |
| **Momentum** | Negative, slow | Negative, accelerating |
| **Recovery rate** | Normal | Slow — congregation holds but bleeds |
| **Liquidity** | Slightly reduced | Normal |

— THRESHOLD CROSSED —

**TRIGGERED**

**GLD  **MARKET CORRECTION IS A SPIRITUAL CLEANSING // PASTOR CONFIRMS

**GLD  **FAITHFUL INVESTORS HOLD // "DOUBT IS THE ENEMY"

**GLD  **RADIANCE DOES NOT FALL // IT DESCENDS TO LIFT YOU HIGHER

**GLD  **TITHE NOW // BLESSINGS ARE COUNTERCYCLICAL

**  FLOOR  //  ABSOLUTE BOTTOM  **

Collapse imminent — language getting vague, fast

**FORESHADOW**

**GLD  **INVENTORY "UNDERGOING SPIRITUAL AUDIT" // TIMELINE UNCONFIRMED

**GLD  **PASTOR ENTERS PRIVATE RETREAT // COMMUNICATIONS REROUTED TO MINISTRY

**GLD  **GLD SUPPLY "IN TRANSITION" // ALL IS WELL // FURTHER QUESTIONS DISCOURAGED

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Critical — approaching floor | Floor — trading halts |
| **Volatility** | Spiking — panic setting in | Near zero — frozen market |
| **Momentum** | Negative, steep | Flat — no movement |
| **Recovery rate** | Failing — buy pressure collapsing | Frozen — awaiting recovery event |
| **Liquidity** | Draining rapidly | Near zero — ammo leaves shops |

— THRESHOLD CROSSED —

**TRIGGERED**

**GLD  **SUPPLY CHAIN INTERRUPTION // "THE LORD IS TESTING LOGISTICS"

**GLD  **HEALING INVENTORY RECALLED // PRODUCT "SPIRITUALLY INCOMPATIBLE"

**GLD  **CONGREGATION ASKED TO FAST // "FROM COMMERCE, BRIEFLY"

**GLD  **RADIANCE PAUSED // NEW SCRIPTURE INCOMING // HOLD POSITIONS

**  RECOVERY  //  CLIMBING BACK  **

Signs of life — church sensing an opportunity to rewrite history

**FORESHADOW**

**GLD  **EARLY INDICATORS POSITIVE // PASTOR CALLS IT "A SIGN"

**GLD  **GLD SHOWING MOVEMENT // CONGREGATION REMINDED TO STAY VIGILANT AND INVESTED

**GLD  **RADIANCE STIRS // "WE HAVE BEEN PREPARING FOR THIS MOMENT"

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | At floor — first upward tick | +25% volatile spike |
| **Volatility** | Low but rising — cautious activity | High — unstable recovery |
| **Momentum** | Turning — negative to neutral | Positive, erratic |
| **Recovery rate** | Thawing — buy pressure returning | Fast but unstable — congregation overreacts |
| **Liquidity** | Low but increasing | Surging — restocks shops |

— THRESHOLD CROSSED —

**TRIGGERED**

**GLD  **PRAYER WORKED // STOCK CONFIRMS DIVINE INTERVENTION

**GLD  **RADIANCE RESTORED // DOUBTERS NOTED FOR THE RECORD

**GLD  **PASTOR: "I NEVER LOST FAITH" // RECEIPTS UNAVAILABLE

**GLD  **RESURRECTION EVENT CONFIRMED // LIMITED MERCH AVAILABLE

**RED  //  FIRE**

**  BULL  //  ABOVE 75%  **

Contracts incoming — aggression building before the spike

**FORESHADOW**

**RED  **SENATE COMMITTEE REVIEWS BUDGET // RED POSITIONED "FAVORABLY"

**RED  **CEO TOURS MILITARY BASE // "PURELY INFORMATIONAL" // DEAL NOT YET SIGNED

**RED  **FIRE OUTPUT TARGETS RAISED // "DEMAND IS COMING // WE ARE READY"

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Rising — 55% to 75% | +25% hard spike |
| **Volatility** | Low — confidence suppressing swings | +20% — overconfidence creates exposure |
| **Momentum** | Positive, steady | Positive, aggressive |
| **Recovery rate** | Fast — government contracts stabilize floor | Very fast — contract backstop |
| **Liquidity** | High — procurement demand rising | Very high |

— THRESHOLD CROSSED —

**TRIGGERED**

**RED  **RED CHOSE FIRE // CONTRACT EXTENDED INDEFINITELY

**RED  **DEFENSE SPENDING UP // "THIS IS WHAT WINNING LOOKS LIKE"

**RED  **CONGRESS AWARDS SOLE-SOURCE BID // COMPETITION "UNNECESSARY"

**RED  **CEO APPEARS ON PODCAST // CALLS CRITICS "SOFT"

**  BEAR  //  BELOW 25%  **

Slide beginning — deflection warming up, blame being assigned

**FORESHADOW**

**RED  **RED FACES HEADWINDS // CEO BLAMES "HOSTILE REGULATORY CLIMATE"

**RED  **FIRE DEMAND SOFTER THAN PROJECTED // "TEMPORARY // ENEMIES STILL EXIST"

**RED  **ANALYSTS CAUTIOUS ON RED // CEO CALLS ANALYSTS "PART OF THE PROBLEM"

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Declining — 40% to 25% | -25% hard drop |
| **Volatility** | Rising — denial creating instability | +35% — anger-driven swings |
| **Momentum** | Negative, building | Negative, aggressive |
| **Recovery rate** | Slowing — blame cycle delays response | Slow — externalized blame delays correction |
| **Liquidity** | Reducing | Reduced — enemies blamed for thin market |

— THRESHOLD CROSSED —

**TRIGGERED**

**RED  **DECLINE CAUSED BY WEAK MARKET // NOT OUR PRODUCT

**RED  **GOVERNMENT INTERFERENCE BLAMED // PATRIOTS TAKE NOTE

**RED  **ENEMIES OF INDUSTRY CELEBRATING // DON'T LET THEM WIN

**RED  **REAL ONES HOLD // EVERYONE ELSE WAS NEVER IN

**  FLOOR  //  ABSOLUTE BOTTOM  **

Collapse incoming — supply chain language getting tactical

**FORESHADOW**

**RED  **PRODUCTION SLOWING // "SOURCING CHALLENGES" CITED // UNRELATED TO PERFORMANCE

**RED  **FIRE STOCKPILES "UNDER REVIEW" // DETAILS CLASSIFIED

**RED  **RED OPERATIONS "RESTRUCTURING" // ALL PERSONNEL REMAIN COMMITTED

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Critical — accelerating toward floor | Floor — operations suspended |
| **Volatility** | Very high — panic and blame | Near zero — frozen |
| **Momentum** | Negative, steep and fast | Flat |
| **Recovery rate** | Near zero — enemies absorbing all blame capital | Frozen |
| **Liquidity** | Collapsing | Near zero — ammo leaves shops entirely |

— THRESHOLD CROSSED —

**TRIGGERED**

**RED  **PRODUCTION HALTED // "REGULATORY SABOTAGE" SUSPECTED

**RED  **ORDNANCE STOCKPILE SEIZED // BUREAUCRATS RESPONSIBLE

**RED  **SUPPLY CHAIN COMPROMISED // ENEMIES WITHIN

**RED  **MARKET SUSPENDED // "TEMPORARY // WE WILL BE BACK // STRONGER"

**  RECOVERY  //  CLIMBING BACK  **

Stirring — already rehearsing the victory speech

**FORESHADOW**

**RED  **FIRE OUTPUT TICKING UPWARD // CEO MONITORS PERSONALLY

**RED  **RED OPERATIONS RESUMING // "WE WERE NEVER TRULY DOWN"

**RED  **SUPPLY CHAIN STABILIZING // ENEMIES HAVE BEEN PUT ON NOTICE

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | At floor — first hard upward move | +30% aggressive spike |
| **Volatility** | Rising from zero — aggression returning | High — overcorrection |
| **Momentum** | Snapping positive — RED recovers hard | Positive, very aggressive |
| **Recovery rate** | Accelerating fast | Very fast — RED does not stay down |
| **Liquidity** | Returning quickly | High — floods back into shops |

— THRESHOLD CROSSED —

**TRIGGERED**

**RED  **RED REBOUNDS // "WE TOLD YOU WE WOULDN'T STAY DOWN"

**RED  **FIRE RETURNS // ENEMIES HAVE BEEN NOTIFIED

**RED  **STOCK RECOVERS // CEO CREDITS "RED GRIT"

**RED  **FULL OPERATIONAL CAPACITY RESTORED // WEAKNESS WAS TEMPORARY

**BLU  //  ZAP**

**  BULL  //  ABOVE 75%  **

Signal building — vague optimism escalating toward revelation

**FORESHADOW**

**BLU  **INTERNAL METRICS "ENCOURAGING" // SPECIFICS NOT FOR EXTERNAL CONSUMPTION

**BLU  **FOUNDER RETURNS FROM RETREAT // "CLARITY ACHIEVED" // NO FURTHER COMMENT

**BLU  **ZAP DEMAND ACCELERATING // THE PATTERN IS BECOMING VISIBLE

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Rising — 58% to 75% | +22% sudden emergence |
| **Volatility** | Very low — BLU suppresses visible movement | +25% — pattern collapse creates instability |
| **Momentum** | Positive, hidden — not visible until threshold | Positive, sharp |
| **Recovery rate** | Strong — true believers hold regardless | Fast but opaque |
| **Liquidity** | Moderate — deliberate scarcity maintained | Spike then controlled reduction |

— THRESHOLD CROSSED —

**TRIGGERED**

**BLU  **ALIGNMENT CONFIRMED // THE NUMBERS ALWAYS KNEW

**BLU  **MARKET RESPONDS TO VISION // DETAILS REMAIN INTERNAL

**BLU  **FOUNDER MEDITATION YIELDS RECORD QUARTER // CAUSATION UNVERIFIED

**BLU  **BULLISH SIGNAL DETECTED // SOURCE: THE PATTERN

**  BEAR  //  BELOW 25%  **

Sliding — reframing beginning, language becoming abstract

**FORESHADOW**

**BLU  **ZAP PRICING SOFTENS // "THE MARKET IS PROCESSING SOMETHING IMPORTANT"

**BLU  **BLU ENTERS "REFLECTIVE PERIOD" // OPERATIONS UNAFFECTED // PROBABLY

**BLU  **ANALYSTS DOWNGRADE BLU // FOUNDER SAYS ANALYSTS "LACK THE FRAMEWORK"

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Declining — 38% to 25% | -28% reframed as intentional |
| **Volatility** | Low — BLU frames decline as stillness | Low — BLU refuses to spike |
| **Momentum** | Negative, slow and deliberate-seeming | Negative, steady |
| **Recovery rate** | Appears normal — reframing masks damage | Slow — process must be trusted |
| **Liquidity** | Thinning quietly | Reduced — scarcity now involuntary |

— THRESHOLD CROSSED —

**TRIGGERED**

**BLU  **THE DESCENT IS INTENTIONAL // TRUST THE PROCESS

**BLU  **LOW PRICES REFLECT INNER WORK // GROWTH REQUIRES STILLNESS

**BLU  **THIS WAS IN THE ROADMAP // PUBLIC ROADMAP REMAINS CLASSIFIED

**BLU  **WE HAVE ALWAYS BEEN HERE // THE NUMBERS DO NOT DEFINE US

**  FLOOR  //  ABSOLUTE BOTTOM  **

Collapse incoming — going fully esoteric before the silence

**FORESHADOW**

**BLU  **CEO COMMUNICATION REDUCED // "WORDS ARE INCREASINGLY INSUFFICIENT"

**BLU  **ZAP SUPPLY "ENTERING NEW PHASE" // PHASE NOT YET NAMED

**BLU  **BLU OPERATIONS "RECALIBRATING" // TIMELINE IS NONLINEAR

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Critical — the void approaching | Floor — latent phase active |
| **Volatility** | Near zero — BLU goes silent before collapse | Zero — the void is still |
| **Momentum** | Negative, invisible | Flat — undefined |
| **Recovery rate** | Nonlinear — timeline unspecified | Frozen — emergence timeline unknown |
| **Liquidity** | Near zero — supply entering latent phase | Zero — ammo entirely absent from shops |

— THRESHOLD CROSSED —

**TRIGGERED**

**BLU  **CEO ENTERS SILENCE PERIOD // QUESTIONS DISCOURAGED

**BLU  **SUPPLY ENTERS LATENT PHASE // EMERGENCE TIMELINE UNSPECIFIED

**BLU  **MARKET ABSENCE IS INTENTIONAL // THE VOID IS PRODUCTIVE

**BLU  **ZAP INVENTORY WITHHELD // "SCARCITY IS A FEATURE"

**  RECOVERY  //  CLIMBING BACK  **

Signal returning — universe apparently wrapping up its work

**FORESHADOW**

**BLU  **CEO BREAKS SILENCE // ONE WORD STATEMENT: "SOON"

**BLU  **ZAP INDICATORS SHIFTING // "THE LATENT PHASE HAS PHASES"

**BLU  **BLU INTERNAL MEMO LEAKED // CONTENTS: "IT IS TIME"

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | At floor — single electric signal | +35% sudden emergence spike |
| **Volatility** | Rising from zero — emergence beginning | Very high — BLU recovery is chaotic |
| **Momentum** | Turning — nonlinear snap | Positive, unexplained |
| **Recovery rate** | Sudden — latent phase resolves instantly | Instant then stabilizes — the process is complete |
| **Liquidity** | Reappearing without warning | Full return — shops restock without announcement |

— THRESHOLD CROSSED —

**TRIGGERED**

**BLU  **THE SIGNAL RETURNS // IT NEVER LEFT

**BLU  **EMERGENCE CONFIRMED // THE LATENT PHASE IS COMPLETE

**BLU  **RECOVERY WAS ALWAYS THE PLAN // PLAN REMAINS UNDISCLOSED

**BLU  **UNIVERSE SELF-CORRECTS // ANALYSTS CALL IT "UNPRECEDENTED"

**GRN  //  GROWTH**

**  BULL  //  ABOVE 75%  **

Metrics climbing — impact reports being quietly pre-written

**FORESHADOW**

**GRN  **GROWTH DEMAND TRACKING AHEAD OF FORECAST // JUNGLE EXPANSION LIKELY

**GRN  **ESG COMMITTEE CONVENES // "OUTCOMES EXPECTED TO BE POSITIVE"

**GRN  **GRN FULFILLMENT CAPACITY UP 30% // NEW WAREHOUSE BUILT ON FORMER WETLAND

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Rising — 60% to 75% | +18% steady climb |
| **Volatility** | Low — GRN suppresses volatility with PR | +10% — success creates mild instability |
| **Momentum** | Positive, managed | Positive, cheerful |
| **Recovery rate** | Strong — stakeholders reassured proactively | Fast — task force standing by |
| **Liquidity** | High — fulfillment capacity expanding | Very high — jungle restocked |

— THRESHOLD CROSSED —

**TRIGGERED**

**GRN  **SUSTAINABILITY PAYING OFF // REPORT AVAILABLE IN RECYCLABLE PDF

**GRN  **GROWTH METRICS EXCEED TARGETS // JUNGLE EXPANSION CONFIRMED

**GRN  **ESG SCORE UPGRADED // THIRD PARTY HIRED BY US

**GRN  **PLANET AND PROFIT ALIGNED // AS ALWAYS // PER OUR VALUES

**  BEAR  //  BELOW 25%  **

Sliding — task force being assembled, statement being drafted

**FORESHADOW**

**GRN  **GROWTH METRICS "MODERATING" // COMPANY REMAINS COMMITTED TO ITS VALUES

**GRN  **GRN MONITORS SITUATION CLOSELY // TASK FORCE ON STANDBY

**GRN  **QUARTERLY TARGETS MISSED // JUNGLE "PERFORMING AS DESIGNED"

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Declining — 38% to 25% | -22% acknowledged decline |
| **Volatility** | Low — GRN manages optics aggressively | +20% — task force activity creates noise |
| **Momentum** | Negative, slow | Negative, managed |
| **Recovery rate** | Normal — task force on standby | Slow — investigation pending |
| **Liquidity** | Reducing steadily | Reduced — fulfillment experiencing friction |

— THRESHOLD CROSSED —

**TRIGGERED**

**GRN  **WE ARE AWARE // A TASK FORCE HAS BEEN FORMED

**GRN  **UNDERPERFORMANCE ACKNOWLEDGED // STAKEHOLDERS REASSURED

**GRN  **THE DIP IS TEMPORARY // OUR COMMITMENT TO THE PLANET IS NOT*

**GRN  **INVESTIGATION UNDERWAY // RESULTS DUE Q4 OF AN UNSPECIFIED YEAR

**  FLOOR  //  ABSOLUTE BOTTOM  **

Collapse incoming — supply chain euphemisms hitting critical mass

**FORESHADOW**

**GRN  **GROWTH SUPPLY "EXPERIENCING FRICTION" // FRICTION "BEING ADDRESSED SUSTAINABLY"

**GRN  **FULFILLMENT DELAYS NOTED // CARBON OFFSET PURCHASED TO ACKNOWLEDGE DELAYS

**GRN  **GRN INVENTORY "IN REVIEW" // REVIEW COMMITTEE MET ONCE // FINDINGS PENDING

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | Critical — friction becoming undeniable | Floor — operations between cycles |
| **Volatility** | Rising — PR losing control of narrative | Low — GRN issues calming statement |
| **Momentum** | Negative, accelerating | Flat — suspended |
| **Recovery rate** | Failing — committee findings still pending | Frozen — carbon offset purchased |
| **Liquidity** | Collapsing — fulfillment pausing | Near zero — jungle on backorder |

— THRESHOLD CROSSED —

**TRIGGERED**

**GRN  **FULFILLMENT PAUSED // ECOSYSTEM "BETWEEN CYCLES"

**GRN  **GROWTH INVENTORY DELAYED // SUPPLY CHAIN IMPACT ASSESSED

**GRN  **JUNGLE TEMPORARILY UNAVAILABLE // BIODIVERSITY ON BACKORDER

**GRN  **OPERATIONS SUSPENDED // CARBON OFFSET PURCHASED TO COMPENSATE

**  RECOVERY  //  CLIMBING BACK  **

Turning corner — task force preparing to claim credit

**FORESHADOW**

**GRN  **TASK FORCE SUBMITS PRELIMINARY FINDINGS // FINDINGS "ENCOURAGING"

**GRN  **GRN SUPPLY CHAIN "SHOWING RESILIENCE" // RESILIENCE RETROACTIVELY PLANNED

**GRN  **GROWTH INVENTORY MOVING AGAIN // JUNGLE RESTOCKED // PACKAGING UNCHANGED

| **METRIC** | **PRE-STATE** | **EFFECT STATE** |
| --- | --- | --- |
| **Worth** | At floor — task force findings arriving | +20% managed recovery |
| **Volatility** | Low — GRN manages recovery optics | +15% — cheerful instability |
| **Momentum** | Turning positive, slow | Positive, bright |
| **Recovery rate** | Beginning — findings being implemented | Fast — task force vindicated |
| **Liquidity** | Returning — jungle restocking | Returning fully — two-day delivery resumes |

— THRESHOLD CROSSED —

**TRIGGERED**

**GRN  **TASK FORCE FINDINGS IMPLEMENTED // RESULTS SPEAK FOR THEMSELVES

**GRN  **GROWTH RESTORED // JUNGLE BACK IN STOCK // TWO-DAY DELIVERY

**GRN  **RECOVERY VALIDATES OUR APPROACH // APPROACH UNCHANGED THROUGHOUT

**GRN  **RESILIENCE CONFIRMED // NEW SUSTAINABILITY REPORT INCOMING
