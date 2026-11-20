# InDebtEd Market Spec

Nov 6, 2026 · @Geoffrey Cook

## Summary

The market now rests near baseline when left alone and gives focused players a moderate, unreliable edge. The original nudge sizes failed: a player using GLD ammo finished the store with GLD at 0.97 on average, and a pumped long position won every time.

The tested values below are proposals, not locked decisions. Five of them change or add to what we agreed:

- **Player pressure cap:** Each brand can move at most 0.01 up and 0.01 down per tick from player actions. This replaces the 0.05 purchase cap.
- **Kill nudge:** 0.002 x Liquidity, down from 0.01 x Liquidity.
- **Recovery:** Each Recovery point pulls 3% of the distance to baseline per tick, up from 1%.
- **Daily open:** Worth opens 75% of the way from yesterday's close to baseline, up from the 50% I first proposed.
- **Shares:** A new 5% trading fee, 50% short collateral, and a margin call at 1.5x entry, replacing 100% collateral at 2x entry.

## Market state and tick order

Each brand holds Worth, Momentum, Volatility, Recovery, Liquidity, a threshold state, and a seeded sine phase. Worth is clamped to 0.01–1.00 after every change.

| Brand | Baseline Worth | Volatility | Recovery | Liquidity |
| --- | --- | --- | --- | --- |
| GLD | 0.62 | 2 | 4 | 4 |
| RED | 0.58 | 3 | 5 | 4 |
| BLU | 0.45 | 1 | Random 0–5 per tick | 2 |
| GRN | 0.40 | 2 | 3 | 4 |

The numbers convert the Reference's labels and were used for testing. BLU's Liquidity of 2 is my reading of "Controlled."

Every 20-second tick runs in this order:

1. Reset each brand's player pressure caps.
2. Advance any Floor timer. A brand at Floor skips steps 3–5 and fires Recovery when its timer ends.
3. Add the sine movement: Volatility x 0.02 x sin(2π x 0.1 x t + phase).
4. Add Momentum x 0.01, then pull Worth toward baseline by Recovery x 3% of the distance.
5. Move Momentum 1 point toward 0.
6. Check thresholds and apply their effects and relationship coupling.

Player nudges apply immediately between ticks and never skip a tick.

## Nudges, thresholds, and Floor

The shared cap sets how far players can move a brand, so aiming matters more than fighting faster. In testing, a kill plus its ammo purchase usually reached the cap on the first kill of a tick.

| Input | Effect on Worth | Limit |
| --- | --- | --- |
| Kill, ammo brand | +0.002 x Liquidity | Shares the +0.01 cap per tick |
| Kill, victim brand | -0.002 x Liquidity | Shares the -0.01 cap per tick; none if same brand as ammo |
| Ammo purchase | +(Spend / 100) x (Liquidity / 3) | Shares the +0.01 cap per tick |
| Share trades | None | Not applicable |
| Thrown or pocketed items | None | Not applicable |

Thresholds apply absolute Worth changes and set Momentum to +2 or -2. Bull and Bear can't fire again until Worth has moved back past the rearm line.

| Event | Trigger | GLD | RED | BLU | GRN | Rearms at |
| --- | --- | --- | --- | --- | --- | --- |
| Bull | Worth at or above 0.75 | +0.20 | +0.25 | +0.22 | +0.18 | At or below 0.65 |
| Bear | Worth at or below 0.25 | -0.10 | -0.08 | -0.09 | -0.08 | At or above 0.35 |
| Recovery | Floor timer ends | +0.25 | +0.30 | +0.35 | +0.20 | Each Floor |

A brand enters Floor at 0.05 or below. Worth freezes there, Liquidity is 0, and shops stop selling that brand for a seeded 3–6 ticks. BLU's timer stays hidden.

## Day cycle and shares

Players commit to positions before the store, can influence them during the run, and settle on the walk home. A share costs Worth x $10, while ammo stays at Worth x $1.

1. **Wake-up:** Worth opens 75% of the way from yesterday's close to baseline. The seed sets the web, phases, and news.
2. **Before the store:** Withdraw BANK funds from Savings, then buy gear and trade shares using BANK money only.
3. **Store:** Ticks run from store entry through the boss fight. Open shorts can receive margin calls.
4. **Settlement:** Trade at the closing price. Long and short proceeds go only to Savings, so they can't repay a loan.
5. **Day end:** Open positions carry over, and unspent BANK money returns to Savings.

| Rule | Longs | Shorts |
| --- | --- | --- |
| Opening cost | Full position value + 5% fee | 50% collateral + 5% fee |
| Closing | 5% fee, proceeds to Savings | 5% fee, collateral plus profit to Savings |
| Risk during the store | Value can fall, no forced sale | Auto-close at live Worth when it reaches 1.5x entry |
| Run end | Stay open | Close at live Worth, with losses taken from Savings |
| Player Position | Previous close value | Collateral only |

A margin call bills the loss above collateral to cash. A threshold jump can create that extra loss, and any shortfall triggers Sudden Death.

## Game integration

Worth still drives combat and spending, while shares add a reason to choose which brand to fight with.

| System | How it connects |
| --- | --- |
| Ammo and damage | Price is Worth x $1, frozen at spawn. Damage reads Worth live at impact. |
| Healing | Provider Worth sets heal cost and the Medical Bill. |
| Cards | Charges and cooldowns follow Worth. |
| Loans | The two highest-Worth brands make offers, and lender Worth scales the amount. |
| Shops | Liquidity sets availability. A brand at Floor isn't sold. |
| Target | Savings and share positions raise Player Position, so wealth funds tougher Targets. |

**Glove and loot**

- F is the glove's melee attack.
- Punching an item puts it in the left hand and turns the gun off.
- Left click throws it as a stun, and right click pockets it. Either action turns the gun back on.
- Thrown items pay nothing and are used up. Their removal is my assumption.
- Pocketed items go to the hoard and sell at the Walk Home before they decay.
- Loot price comes from fixed item value and decay, not Worth.

**Design Reference updates**

- Remove the rule that nudges override the next tick.
- Remove smashing as a cash source and update the glove and Fence & Hoard rules.
- Change Savings from "cannot be withdrawn" to funding the BANK card.
- Replace the kill, purchase, and Recovery formulas, and the half-life note. At 3%, REC 3 has a half-life of about 7 ticks, REC 4 about 5, and REC 5 about 4.
- Replace the Bear Worth effects and close the stock trading questions.

## Simulation results

The final values remove the guaranteed trade, but GRN is still the best brand to manipulate. Results cover 1,500–2,000 simulated 15-minute store runs, or 45 ticks, with about 1.8 kills per tick when fighting.

| One-day position, baseline entry, 5% fee | First pass | Final values |
| --- | --- | --- |
| Long GLD, using GLD ammo | +56%, wins 100% | +3%, wins 59% |
| Long GRN, using GRN ammo | +139%, wins 100% | +9%, wins 73% |
| Long RED, using RED ammo | Not run | 0%, wins 50% |
| Long GLD, no player action | Not comparable | -5%, wins 30% |
| Short RED, all kills on RED | +54% to +71%, wins 100% | +1%, wins 53% |
| Short GRN, all kills on GRN | Not run | +12%, wins 76% |

The first-pass numbers exclude the fee, so the comparison is approximate. The -5% idle result is mostly the fee.

**Other findings**

- **Market stability:** With no player action, every brand closed at baseline on average and no thresholds fired.
- **Bull frequency:** GLD went Bull in about 25–30% of GLD-ammo runs, down from 100%. RED rarely did, and GRN didn't.
- **Overnight trade:** Betting on the return to baseline earned +2.6% at a 50% open and -0.4% at a 75% open.
- **Ten-day compounding:** All funds in a pumped GLD position had a median return of x1.12. GRN reached x1.69, and no pumping returned x0.59.
- **Margin calls:** At baseline entry, a short was called in 19% of runs for GLD and 3% for RED when the player pumped that brand. BLU and GRN were never called. Threshold jumps added up to 20% of position value in loss above collateral.
- **Low-entry shorts:** Shorting GRN at 0.30 was called in 89% of runs, and at 0.25 in 100%, because recovery carries it past the trigger.
- **Cash economy:** Ammo costs about $2.50 per kill. Adding three full GLD heals raises that to $4.80 per kill over 81 kills, or $7.15 over 40.

**What this didn't test:** Relationship coupling, combo nudges, threshold changes to Volatility, Recovery, or Liquidity, Target scaling, and actual kill payouts. The Reference doesn't list a kill payout, so the cash result is only a break-even target.

## Open items

These decisions are still open. The first group affects the next session on news stories and market events.

**For news and market events**

- [ ] What can a headline change: Momentum, a direct Worth move, Liquidity, or only information?
- [ ] How accurate is the morning paper, and how does BLU's unreliability appear?
- [ ] What triggers the three daily shocks used in testing? They set Momentum to +3 or -3 for a random brand at a random tick.
- [ ] How do Foreshadow headlines work when a Momentum change can cancel the crossing?
- [ ] Do threshold events and coupling keep their Reference values once tested together?

**For the economy**

- [ ] Do you approve the five proposed value changes in the summary?
- [ ] Should the player pressure cap scale with Liquidity? It's flat now, so Liquidity barely changes how far combat moves a brand.
- [ ] Should GRN's stronger trading returns stay as part of its identity or be reduced?
- [ ] What is the kill payout now that smashing no longer pays?
- [ ] Does loot sale money go to cash or Savings?
- [ ] Do margin bills come from earned cash, BANK, or both?
- [ ] Should there be a position size limit, or will Target scaling restrain compounding enough?
- [ ] Does the market stay frozen before store entry? I assumed so, but we haven't locked it.
