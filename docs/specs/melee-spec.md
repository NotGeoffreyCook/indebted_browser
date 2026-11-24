# InDebtEd Melee Spec: HAND OF THE MARKET

Nov 23, 2026 · @Geoffrey Cook

HAND OF THE MARKET has five strike types: the current melee as the default and one brand strike each for RED, BLU, GRN and GLD. Every strike shoves nearby enemies when it lands, and each brand changes the shape of that shove.

All values are starting values for tuning in play.

## Neighbor kickback (all strikes)

When a melee hit lands, enemies near the one that was hit get a small shove.

| Rule | Value |
| --- | --- |
| Who | Enemies within 2.5 m of the struck enemy, nearest first, max 4. The struck enemy is excluded. |
| Direction | Away from the struck enemy (not away from Ed), flattened, with a 10–15° upward tilt. The crowd ripples outward from the impact. |
| Strength | 30% of the primary hit's knockback, falling off linearly to 0 at the 2.5 m edge. Scaled by enemy mass; tanks barely move. |
| Damage | None. Push only. |
| Reactions | Each pushed enemy rolls its Reaction Table as a near-miss: fodder Hesitates and blocks bigger enemies behind it, RED gets Adrenaline, GRN Scatters & Flanks. |
| Exception | A Flanker mid-charge ignores kickback, since it already pushes through crowds. |

Each brand strike overrides the kickback shape (see below).

## Strikes

### Default: Ed's stock glove

The current melee, unchanged. Baseline for every multiplier below. Uses the standard radial kickback.

### RED: "Rapid Deployment" (short and quick)

- Three-jab chain, ~0.25 s per jab, 0.8x reach. Jab 3 is the heavy hit.
- **Kickback:** minimal on jabs 1–2, full on jab 3, so the crowd stays put while you work one target.
- **Brand hook:** each connected jab in a chain speeds up the next. RED gets angrier the longer it fights, like RED enemies. Pairs with the RED dash: dash in, jab out.
- **Look:** Coolant cyan (#7FF3FF) vent puffs from the knuckles on each jab.

### BLU: "Wire Transfer" (fast and long)

- Lunging straight punch: fast windup, ~2x reach, Ed slides forward ~1.5 m. Long recovery on a whiff, so a miss leaves you exposed.
- **Kickback:** linear instead of radial. The next 1–2 enemies behind the target are pushed along the strike line, clearing a lane.
- **Brand hook:** a short, committed cousin of the Zap blink. For closing on ranged enemies or clearing a corridor. All numbers stay visible (no BLU number-hiding).
- **Look:** Arc orange (#FF8A1A) spark trail along the reach line.

### GRN: "Diversification" (wide and arcing)

- Sweeping backhand, ~160° arc. Slower windup, hits everything in the arc at ~0.7x damage each.
- **Kickback:** lateral, perpendicular to the swing, alternating left and right. It splits groups the way GRN enemies scatter.
- **Brand hook:** the crowd and flank tool: fodder hordes, enemies closing on your sides. GRN spreads itself thin and hopes it works out.
- **Look:** Bloom magenta (#FF3DCB) trail in the shadow side of the arc.

### GLD: "Tithe" (heavy overhead slam)

Proposed as the fourth shape, since GLD needed a strike alongside the three listed shapes.

- Slow overhead smite with the longest windup, plus a ground shockwave at impact.
- **Kickback:** radial, bigger and stronger than the rest (~4 m radius, ~60% strength). GLD is the strongest pusher.
- **Brand hook:** smashing breakables with Tithe pays a cash bonus (GLD follows the money). The best way to farm merchandise; heavy shelves still only shift slightly.
- **Look:** Radiance (#FFD24A) flash with a Purple (#9B4DFF) shockwave ring at floor level.

## At a glance

| Strike | Speed | Reach | Shape | Kickback |
| --- | --- | --- | --- | --- |
| Default | 1.0x | 1.0x | Single hit | Radial, 30% |
| RED Rapid Deployment | Fastest | 0.8x | 3-jab chain | Minimal, then full on jab 3 |
| BLU Wire Transfer | Fast in, slow out | 2.0x + lunge | Straight line | Linear, along the strike line |
| GRN Diversification | Slow | 1.0x, 160° arc | Sweep | Lateral, alternating |
| GLD Tithe | Slowest | 1.0x + 4 m wave | Overhead slam | Radial, 60% |

## System rules

- **Brand stance:** the strike's brand checks stance against the enemy's primary brand with the same `GetStance` call as ammo. A Negative-stance strike gets +25% damage, Positive -20%, Tentative rolls 0.8x–1.3x.
- **One data-driven ability:** a single melee ability reads a DataTable row per strike (`FMeleeStrikeData`): Brand, WindUp, Active, Recovery, Reach, ArcDegrees, HitCount, DamageMultiplier, PrimaryKnockback, SplashRadius, SplashScale, SplashDirMode (`Radial`, `Lateral`, `Linear`). Adding or tuning a strike is a data change, not new Blueprint logic.

## Open

- **Selection:** proposed as a swappable brand plate on the HAND OF THE MARKET gloves, chosen at the gear / swap-parts step before the store.
- **Economic role:** whether brand strikes cost anything to use is still undecided (see Mechanics Inventory 6.1).
