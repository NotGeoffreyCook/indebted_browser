**InDebtEd**

Design Reference  //  Systems, Math & Headlines

*Revised 2026-10-30 (hygiene pass). Now the single source for market states and headlines; market_states_v3 is retired. New: Fence & Hoard section and expanded Open Questions.*

**GLOSSARY**

Key terms used throughout this document. All values are runtime variables unless noted as design constants.

| **TERM** | **DEFINITION** |
| --- | --- |
| **Worth** | Float 0.0 to 1.0. The current market value of a brand. Directly maps to per-round price ($0.00 to $1.00) and damage output. The primary market variable. |
| **Baseline Worth** | Design constant. The resting value Worth oscillates around per brand. GLD: 0.62, RED: 0.58, BLU: 0.45, GRN: 0.40. |
| **Volatility** | Integer 0 to 5. Amplitude multiplier on the sinusoidal engine. Higher = larger price swings per tick. |
| **Momentum** | Integer -3 to +3. Directional pressure applied to Worth each tick. Positive pushes up, negative pushes down. |
| **Recovery Rate** | Integer 0 to 5, or R (BLU random). Speed at which Worth drifts back toward Baseline after an event. 0 = frozen. |
| **Liquidity** | Integer 0 to 5. Controls shop ammo availability and how much player actions move the price. 0 = ammo absent. |
| **Base Tick** | The 20-second timer cycle. Primary cadence for sinusoidal update, recovery drift, and Sudden Death debt increment. |
| **Event Trigger** | Immediate market update fired by player action (kill, purchase, combo change, threshold crossing). Fires on top of base tick. |
| **Foreshadow Headlines** | Ticker headlines that run before a threshold is crossed. Signal to attentive players that a market event is coming. |
| **Triggered Headlines** | Ticker headlines that fire when a threshold is crossed. Reflect the brand identity spin on the market state. |
| **Pre-State** | Market metric values during the foreshadow window — the conditions heading into a threshold. |
| **Effect State** | Market metric deltas that apply instantly when a threshold is crossed. |
| **Threshold** | A Worth value that triggers a market event. Four per brand: Bull (above 75%), Bear (below 25%), Floor (absolute bottom), Recovery (climbing from below 25%). |
| **Performance Bonus** | End-of-level payout. Combo multiplier x total earnings during level. Paid to Savings at Settlement (Walk Home). Currently the only path to Savings growth; under revision, see Fence & Hoard. |
| **Sudden Death** | Player state triggered when cash goes negative after a heal bill. Next hit = instant death (which triggers the Zero HP flow). Debt accumulates each tick until cleared. Accepting a loan clears the negative balance. |
| **Combo Multiplier** | Float scaling with tier (Bronze through InDebtEd!!!). Applied to earnings and performance bonus. Degrades over time without kills. |
| **Credit Card** | Purchasable item. Grants a credit line usable in shops. Card abilities scale with % of credit used. Challenges grow the credit line. |
| **Provider** | Player-chosen brand for healing. Heal cost per HP = provider brand Worth at time of charge. Also prices the Medical Bill. |
| **Medical Cost** | Savings deduction when a run ends. Permanent career penalty. Amount determined by run circumstances. |
| **Nudge** | Small immediate Worth adjustment from a player action. Kill nudge: +0.01 x Liquidity. Purchase nudge: spend / 100. |
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

**IMPLEMENTATION TO-DO  //  PRIORITY ORDER**

Ordered by what is needed to close the core gameplay loop first. Items marked PROTOTYPE have a working first pass. Items marked TODO are not yet started.

| **STATUS** | **SYSTEM** | **NOTES** |
| --- | --- | --- |
| **PROTOTYPE** | **Combo system** | Kill increments counter, timer resets on kill, timer expiry breaks combo. BP_ComboManager on Player Controller. Tier thresholds: Bronze 1-4, Silver 5-9, Gold 10-19, Platinum 20-34, InDebtEd!!! 35+. |
| **PROTOTYPE** | **Ammo economy / stock market** | Sinusoidal engine, base tick timer, Worth variable per brand, per-round price output. Prices must move in response to player behavior before any downstream system works. |
| **TODO** | **Performance bonus** | Combo multiplier x earnings tracker. Accumulates during level, paid out to Savings at Settlement. Degrades over time to discourage camping. Visible only with starter credit card at 50% usage. |
| **TODO** | **Market event system** | Threshold monitors per brand. Bull above 75%, Bear below 25%, Floor at bottom, Recovery climbing back. Each fires effect state values instantly and triggers headline rotation. |
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

6. Seed the market — each brand starts at Baseline Worth

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
| **Negative** | +25% damage. Double kill nudge to ammo brand. Rival -0.01 Worth per kill | Conflict: existing passive / active debuffs | Inverse (see below) | +15% heal cost |
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

Sympathy nudges fire with the threshold event and follow the existing nudge priority order.

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

All formulas operate on Worth as a float 0.0 to 1.0. Worth is always clamped to 0.01 minimum (never true zero) and 1.0 maximum after any operation.

**SINUSOIDAL ENGINE  //  fires every 20s base tick**

Amplitude    A = VOL x 0.02                        Range: 0.00 to 0.10

Frequency    F = 0.1 (fixed)                       1 full cycle = 10 ticks / 200s

Mom. Offset  M = MOM x 0.01                        Range: -0.03 to +0.03

Sine Value   S = A x sin(2 x PI x F x t)

Apply        Worth += S + M

Example: VOL 2, MOM +1  ->  A=0.04, M=+0.01, max swing +-$0.04 plus +$0.01 push per tick

Example: VOL 4, MOM -2  ->  A=0.08, M=-0.02, max swing +-$0.08 minus -$0.02 push per tick

Example: VOL 0, MOM  0  ->  A=0.00, M=0.00,  frozen — no movement

Note: S is added every tick, so Worth accumulates the sine. The per-tick change is at most ±A, but the cumulative swing over half a cycle is about 3.1 × A (VOL 2 ≈ 0.12 peak to trough), before recovery drift damps it.

**RECOVERY DRIFT  //  fires every 20s base tick after sine**

Difference   D = BaselineWorth - CurrentWorth

Drift Step   Step = D x REC x 0.01

Apply        Worth += Step

Skip if      REC == 0 (frozen — awaiting trigger event)

BLU special  REC = random roll 0 to 5 each tick

Example: Worth 0.20, Baseline 0.62, REC 3  ->  D=0.42, first Step=+0.013  half the gap closed in ~23 ticks / 7.6 min

Example: Worth 0.20, Baseline 0.62, REC 5  ->  D=0.42, first Step=+0.021  half the gap closed in ~14 ticks / 4.5 min

Example: Worth 0.20, Baseline 0.62, REC 0  ->  frozen, no drift until recovery event fires

Note: the step shrinks as the gap shrinks, so Worth approaches baseline exponentially and never fully arrives. Half-life in ticks = ln 0.5 / ln(1 − REC × 0.01): REC 1 ≈ 69, REC 2 ≈ 34, REC 3 ≈ 23, REC 4 ≈ 17, REC 5 ≈ 14.

Note: constant momentum and recovery settle at an equilibrium of Worth = Baseline + MOM / REC. See Open Questions (Momentum equilibrium).

**EVENT NUDGES  //  fire immediately, override next tick adjustment**

Kill nudge       Worth += 0.01 x Liquidity          Max at LIQ 5: +0.05

Purchase nudge   Worth += SpendAmount / 100         Ex: $6.20 spend = +0.062

Combo tier up    Worth += 0.03  (active brand)

Combo break      Worth -= 0.05  (active brand)

Threshold event  Apply full effect state instantly   Overrides all other nudges

Priority order if multiple fire same frame:

1. Threshold event  (highest — overrides all)

2. Combo change

3. Kill nudge

4. Purchase nudge   (all stack additively)

**OUTPUT FORMULAS  //  derived from Worth each tick**

Price per round    Price = Worth x $1.00

Damage output      Damage = Worth x MaxDamage

Heal cost per HP   HealCost = ProviderWorth x $1.00

Full heal bill     Bill = HealCost x HP_Recovered

Sudden Death debt  Debt += Bill  (each tick while cash < 0)

Performance bonus  Bonus = Sum(Earnings x ComboMultiplier)  paid to Savings at Settlement

Combo multiplier   Bronze: 1.0x  Silver: 1.5x  Gold: 2.0x  Platinum: 2.5x  InDebtEd!!!: 3.0x+

Medical Bill       Bill = ProviderWorth x MaxHP x 2.0  (decline loan, respawn at checkpoint)

Loan amount        Amount = LadderBase x LenderWorth x 1.6  (frozen at acceptance)

Loan repayment     Repay = Amount x (1 + APR)  (auto-deducted from cash at Settlement)

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
| **Sinusoidal worth update** | Every tick | Timer | Volatility value determines amplitude of each shift. |
| **Liquidity recalculation** | Every tick | Timer | Shop inventory and price responsiveness updates on each tick. |
| **Sudden Death debt increment** | Every tick | Timer | While insolvent, debt grows once per 20s tick. 20 seconds between increments. |
| **Kill / purchase nudge** | Immediate | Event | Fires instantly. Overrides next base tick worth adjustment for that brand. |
| **Combo tier change** | Immediate | Event | Tier up or break fires market signal immediately. |
| **Threshold crossed** | Immediate | Event | Full effect state values applied instantly. Triggered headlines fire. Relationship sympathy nudges fire on paired brands. |
| **Foreshadow window opens** | ~60s before threshold | Predictive | 3 ticks before projected crossing. Momentum can still shift — not guaranteed. |
| **Day generation** | Once, at Wake-Up | Seeded | Relationship web, Target, Target shopping, market seed, news. Fixed for the day. |

Base tick cadence: 20s default. Tune between 15s (reactive) and 30s (tidal) based on playtest feel.

Event triggers override the next base tick worth adjustment for that brand only — other brands unaffected.

Foreshadow window is predictive, not guaranteed. Momentum shift can cancel it before crossing.

Sudden Death debt increments on the same 20s tick. Player has one full tick window to earn before next increment.

BLU recovery rate (R) is rolled fresh each tick. Min 0, max 5. Player cannot predict BLU recovery timing.

**BASELINE MARKET STATE**

Resting state values the sinusoidal engine oscillates around. Each brand reflects its satirical identity in its baseline position.

| **METRIC** | **GLD** | **RED** | **BLU** | **GRN** |
| --- | --- | --- | --- | --- |
| **Worth** | 62% | 58% | 45% | 40% |
| **Price / round** | $0.62 | $0.58 | $0.45 | $0.40 |
| **Volatility** | Low | Medium | Very low | Low |
| **Momentum** | Slight + | Neutral | Neutral | Slight - |
| **Recovery rate** | Fast | Very fast | Unknown | Medium |
| **Liquidity** | High | High | Controlled | High |

**GLD  **Sits above midpoint. Congregation believes things are always ascending. Room to crash but baseline confidence is high.

**RED  **Propped up by government contracts. Volatile but rarely truly low. Bureaucracy provides a stable floor.

**BLU  **Deliberately opaque at midpoint. The market cannot read it. Every move feels surprising.

**GRN  **Perpetually underperforming its own projections. Looks stable on paper, quietly sliding. Slight negative momentum by default.

**DAMAGE AND ECONOMY SCALE**

Worth translates directly to per-round cost (62% = $0.62/round) and damage output. Standard engagement: 5v1. Base enemy requires 4 body hits or 2 headshots at baseline price.

| **STATE** | **PRICE / RND** | **BODY HITS** | **HEADSHOTS** | **COST / ENG.** |
| --- | --- | --- | --- | --- |
| Bull (85%) | $0.85 | ~2 | 1 | $8.50 |
| GLD base (62%) | $0.62 | 4 | 2 | $12.40 |
| RED base (58%) | $0.58 | 4 | 2 | $11.60 |
| BLU base (45%) | $0.45 | 5-6 | 3 | $12.38 |
| GRN base (40%) | $0.40 | 6 | 3 | $12.00 |
| Bear (25%) | $0.25 | 8-9 | 4-5 | $11.25 |
| Near floor (10%) | $0.10 | ~16 | 8 | $8.00 |

Damage scales directly with worth — 100% = max damage, 1% = minimum damage. Relationship stance modifies damage on top (Negative +25%, Positive -20%, Tentative 0.8x–1.3x).

Total engagement cost stays broadly flat (~$8–12). Low price = more rounds fired = longer exposure.

The economy is self-balancing: price changes danger not total cost. Cheap ammo is a survival tax.

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

Shield recharges automatically after damage (Halo-style). Recharge is not free — cost is deducted from cash when the player reaches 100% HP. Cost per HP equals the current stock % of the player chosen heal provider.

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

**HEAL COST BY PROVIDER STOCK STATE**

| **STATE** | **STOCK %** | **COST / HP** | **100 HP FULL** | **RISK LEVEL** |
| --- | --- | --- | --- | --- |
| Bull (85%) | 85% | $0.85 | $85.00 | Insolvency risk high |
| GLD base (62%) | 62% | $0.62 | $62.00 | Moderate |
| GRN base (40%) | 40% | $0.40 | $40.00 | Lower risk — debt accumulates slower |
| Bear (25%) | 25% | $0.25 | $25.00 | Low cost but weak ammo — longer fights |
| Near floor (10%) | 10% | $0.10 | $10.00 | Cheapest heals — but ammo is nearly useless |

Provider is player-chosen at start of level. Heal cost tracks that brand stock in real time.

Provider stance vs the player's ammo brand modifies heal cost: Positive -15%, Negative +15%.

Provider also prices the Medical Bill — provider choice affects what death costs.

Sudden Death debt grows every 20s market tick — hiding does not stop the clock.

Cheap provider = slower debt accumulation in Sudden Death. Provider choice is also a risk management decision.

The performance bonus is currently the only path to Savings growth; fencing stolen loot is planned to overtake it (see Fence & Hoard). Combo maintenance still scales the bonus.

**LOANS & DEATH  //  ZERO HP FLOW**

Hitting 0 HP is a financial decision, not a fail state.

0 HP  ->  Two loan offers appear

ACCEPT   ->  Revive in place. Debuff applied. Loan cash added (clears any Sudden Death debt).

DECLINE  ->  Cash >= Medical Bill  ->  Pay bill, respawn at nearest checkpoint (boss fight restarts)

DECLINE  ->  Cash < Medical Bill   ->  Run ends

**LOAN RULES**

Offers come from the two highest-Worth brands — brands with capital lend. One offers a lower tier, one a higher tier.

Brands at Floor do not lend (no liquidity).

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
| **Fence proceeds** | Do sale proceeds land in cash, Savings, or both? Where does the sale sit relative to loan repayment? |
| **Morning funding** | Cash is lost at day end and Savings can't be spent. What pays for step 4 purchases besides credit? |
| **Purpose of Savings** | Higher Savings only raises Target budgets. What does it unlock or lead to — a long-term goal or end state? |
| **Credit card overnight** | Settlement covers loans only. What happens to an unpaid card balance at day end (APR, carry-over, Player Position)? |
| **Effect-state units** | Is "+20% Worth" absolute (+0.20) or relative (x1.2)? "+15% Volatility" doesn't fit the integer 0–5 scale. Effect states need concrete deltas. |
| **Threshold values** | Floor and Recovery have no numeric Worth value. Recovery needs a crossing point (e.g. back above 0.25 from Floor). |
| **Momentum equilibrium** | Constant momentum settles Worth at Baseline + MOM / REC. GLD (MOM +1, REC 4) rests at ~0.87, permanently Bull. GRN (MOM −1, REC 3) rests at ~0.07, near Floor. Options: momentum decays toward 0 after events, or baseline momentum is 0 and the lean comes from elsewhere. |
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
