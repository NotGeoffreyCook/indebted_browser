# InDebtEd Credit Card Spec

Nov 5, 2026 · @Geoffrey Cook

Equipping a card grants its ability, using more credit makes it stronger, and every charge follows that brand's live stock price.

## Core rules

Every card starts with a $250 limit and gains power in 25% steps as its balance rises.

| Credit used | Balance at $250 limit | Power |
| --- | --- | --- |
| Under 25% | Under $62.50 | Underpowered floor (25% placeholder) |
| 25% | $62.50 | 50% |
| 50% | $125 | 100% |
| 75% | $187.50 | 150% |
| 100% | $250 | 200% |

- **Credit limit:** Upgrading a card to the next level raises its limit. The upgrade system is still to be designed.
- **Pricing:** Card charges use the brand's live price at Worth x $10. This does not change ammo, heal, or loan prices.
- **Interest:** A flat 5% of the card balance is charged every 30 seconds. It bills to the BANK card and does not raise the card balance or power.
- **At the limit:** The ability stays usable, and further charges bill to the BANK card.
- **Repayment:** Repaying through the WALLET lowers the balance, so it also lowers card power.

## Card abilities

Each card has one ability, and each ability has its own charge trigger.

| Card | Ability | Credit scales | Charged | Cooldown |
| --- | --- | --- | --- | --- |
| GLD | Toggle sprint | Sprint strength | Per step | None |
| RED | Dash | Distance and damage to enemies in the path | Per dash | Faster at higher Worth |
| GRN | Boosted jump | Jump power | Per jump | Faster at higher Worth |
| BLU | Q toggles HUD display | Unlocked HUD elements | While on (every 5 seconds, placeholder) | None |

BLU's card stays clear and reliable, unlike the brand's usual illegibility.

## Payment failure

Charges move down this ladder as each source runs out, ending in Default.

1. **Card:** Ability charges go on the card while it is under its limit. Interest always goes to the BANK card.
2. **BANK card:** Takes ability charges past the limit, plus interest.
3. **Overdraft:** When the BANK card is empty, charges come from Savings with a severe fee (+50% placeholder).
4. **Default:** When Savings is empty, the player takes and deals double damage. Killing enough enemies ends Default.
5. **Game over:** Reaching 0 HP in Default ends the run, with no loan offers or respawn.

**BANK card:** The player moves money from Savings to the BANK card when leaving the apartment. Remaining money returns to Savings if the player survives the day and is lost if the run ends.

**Cash:** Cash and the BANK card are separate. Kills and smashing still pay Cash, and Cash cannot cover BANK charges.

## HUD

BLU replaces the starter-card HUD ladder, while the performance bonus and receipt stay separate from cards.

| BLU credit used | Unlock |
| --- | --- |
| 25% | Crosshairs and damage numbers |
| 50% | Enemy healthbars |
| 75% | Enemy outlines |
| 100% | Top-down map |

- **BLU display:** Q turns the display on and off, and it only charges while on.
- **Performance bonus tracker:** Separate UI item at the top right.
- **Purchase receipt:** Sits under the tracker and records each purchase. It holds up to 20 line items, and each new line fades out after 1 second.

## Formulas and test values

The test environment starts with $1,000 on the BANK card, and a maxed $250 card costs $25 per minute in interest.

```latex
\text{Power} = \left\lfloor \frac{\text{CreditUsed}}{0.25} \right\rfloor \times 50\%
```

```latex
\text{Charge} = \text{Worth} \times \$10
```

```latex
\text{Interest} = \text{CardBalance} \times 5\% \text{ every 30 seconds}
```

```latex
\text{Cooldown} = \text{BaseCooldown} \times \frac{\text{BaselineWorth}}{\text{Worth}}
```

The cooldown needs a minimum and maximum so a Floor crash does not create an extreme wait. Base cooldowns and those limits are not set yet.

| Card | Charge at baseline Worth | Charges to 100% power | Charges to $250 limit |
| --- | --- | --- | --- |
| GLD | $6.20 | 21 | 41 |
| RED | $5.80 | 22 | 44 |
| GRN | $4.00 | 32 | 63 |
| BLU | $4.50 | 28 | 56 |

## Open questions and doc changes

The core rules are set, but these details need answers or tuning before the cards are fully playable.

**Open questions**

- [ ] How many kills end Default, and does the counter reset if the player leaves combat?
- [ ] Can abilities still be used in Default when no money source can pay for them?
- [ ] What happens when a player starts a new day with $0 in Savings?
- [ ] Should crosshairs be exclusive to BLU, leaving GLD, RED, and GRN without them?
- [ ] Does each ammo round and sprint step get its own receipt line, or should rapid charges be grouped?
- [ ] Should the BLU display charge every 5 seconds?
- [ ] What are the base cooldowns and cooldown limits for RED and GRN?
- [ ] What should the overdraft fee and underpowered floor be?
- [ ] What step rate should GLD sprint use?
- [ ] What do card upgrades cost, and how much do they raise the limit?

**Design Reference changes**

- **Savings:** Replace "Cannot be withdrawn" with "Can be moved to the BANK card when leaving the apartment."
- **Player economy:** Add the BANK card, separate from Cash.
- **Credit card HUD unlocks:** Replace the starter-card ladder with the BLU ladder, and move the performance bonus tracker to its own top-right UI item.
- **APR:** Replace balance-based APR with the flat 5% placeholder.
- **Zero HP flow:** Add Default as an exception, with no loan offers or respawn at 0 HP.
- **Morning funding:** Note that the BANK card can pay for purchases made after leaving the apartment.
