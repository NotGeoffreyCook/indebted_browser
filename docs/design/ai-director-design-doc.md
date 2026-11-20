**AI DIRECTOR**

Game Systems Design Document

*Version 0.2  —  Design Draft  (revised 2026-10-30: cross-references to the Design Reference, debt collectors, open questions)*

# **1. Overview**

BP_AIDirector is a single persistent actor spawned by GameMode on level load. It is the sole source of truth for how the level responds to player behaviour — deciding which enemies and traps appear, how segments are modified, and when the level escalates pressure. Individual enemies and segments are passive reporters and executors; the director is the brain.

The director operates on a hybrid scripted + reactive model: each level defines an opening stance, and the director adapts from that baseline based on a rolling PlayerSnapshot evaluated every 2–3 seconds.

# **2. Core Structure**

## **2.1  Key Variables**

| **Variable** | **Type** | **Description** |
| --- | --- | --- |
| CurrentApproach | Enum | Kill / Drain / Trick / Taunt |
| EscalationTier | Int (1–3) | 1 = scripted, 2 = adapted, 3 = panic mode |
| ScriptedOpeningStance | Data Asset | Per-level starting posture, set in Level Data Asset |
| DirectorState | Enum | Dormant / Observing / Active / Escalating |
| LevelBudget | Float | Replenishes over time; director spends to spawn |
| BudgetReplenishRate | Float | Per-level replenish speed |
| BossUpgradePool | Float | Accumulates from pay-to-pass fees |
| PlayerSnapshot | Struct | Refreshed every 2–3s by decision loop timer |

## **2.2  PlayerSnapshot Struct**

Updated on a repeating timer (never on Tick) to avoid performance overhead. SegmentDelta and ResourceBurnRate are the two highest-signal fields — SegmentDelta flags whether the player is outpacing level design, and ResourceBurnRate tells the director whether Drain mode is working.

| **Field** | **Purpose** |
| --- | --- |
| HealthPercent | Current / max HP |
| CurrentMoney | Liquid resources |
| KillsThisMinute | Rolling count from EnemyManager |
| TimeInLevel | Elapsed seconds |
| ExpectedTimeAtSegment | Design target from level data |
| SegmentDelta | Actual vs expected progress (+ = player ahead) |
| ResourceBurnRate | Rolling average of player spending speed |

# **3. Decision Loop**

Runs on a repeating timer every 2–3 seconds. Timer interval is itself a director variable — it shortens as EscalationTier increases, creating a natural sense of the level waking up and reacting faster.

**Each cycle executes in order:**

- Refresh PlayerSnapshot

- Check escalation triggers — update Tier if thresholds are breached

- If Tier changed, re-evaluate CurrentApproach weights

- Run spawn decision for CurrentApproach against available budget

- Queue any modifications to unvisited segments

Approach selection uses a priority weight array rather than a strict state machine. Each approach holds a 0–100 weight recalculated each cycle from the snapshot. Whichever weight is highest becomes the CurrentApproach. Starting weights are baked into the Level Data Asset, providing the scripted opening posture while allowing smooth reactive shifting without hard cuts.

# **4. Approach Modes & Escalation**

Each approach runs through three escalation tiers. Tier 1 is the scripted opening. Tier 2 triggers after a configurable time window or threshold breach. Tier 3 is the panic state — the player is clearly winning and the director has exhausted its preferred options.

| **Mode** | **Tier 1 (Scripted)** | **Tier 2 (Adapted)** | **Tier 3 (Panic)** |
| --- | --- | --- | --- |
| **Kill** | Standard damage-pressure enemies | Ramp damage weight if HP > 75% after X min | Spawn elites / stack hazards if HP > 85% at midpoint |
| **Drain** | Moderate-cost enemies, trap mix | Escalate if ResourceBurnRate < threshold (player hoarding) | Shift to Trick if budget > 60% replenished at midpoint |
| **Trick** | Redirect traps, false paths, route disruption | Activate segment-mod queue early if SegmentDelta > 0 | Stalling enemies (high HP, low threat) to eat the clock |
| **Taunt** | Cheap bait threats near upgrades / pickups | Trigger if kill rate + burn rate both high — bait bad trades | Leave openings near expensive targets, counter-spawn on commit |

Kill ease-off condition: if HealthPercent < 30%, pause Kill escalation to avoid cheap deaths.

# **5. Budget System**

LevelBudget replenishes passively at BudgetReplenishRate per second. The player races to drain it faster than it replenishes. Each spawnable enemy and trap type has a BudgetCost defined in a shared data table.

| **Event** | **Budget Effect** |
| --- | --- |
| Passive time | LevelBudget replenishes at BudgetReplenishRate/s |
| Director spawns enemy/trap | Director spends BudgetCost (from data table) |
| Player kills enemy | Portion refunded to player side (not back to director) |
| Player triggers trap and survives | Trap activation costs director budget |
| Pay-to-pass fee | Routes to BossUpgradePool — does not restock LevelBudget |
| LevelBudget hits zero mid-escalation | Director borrows — LoanTotal increases; player bonus scales with LoanTotal at level end |
| Debt collector spawns | Free — never spends LevelBudget. Spawn cap scales with the player's outstanding loan balance (see Design Reference → Debt Collectors) |

### **Director Spend Rules**

- Never spend below a ReserveBudget floor (keeps capacity for emergency response)

- Higher EscalationTier lowers the reserve floor — director gets more reckless

- Approach weights influence what budget is spent on: Kill = combat-heavy spawns, Drain = trap-heavy, Trick = obstruction / routing, Taunt = bait placements

### **Loan Mechanic & Performance Bonus**

When LevelBudget hits zero and escalation still demands spawning, the director borrows — incrementing a LoanTotal float. The player's level-end performance bonus scales directly with LoanTotal. This means the director's panic spending when the player is winning rewards the player more: a clean positive feedback loop that makes winning feel earned.

Naming: LoanTotal is the Director's own borrowing. It is separate from the player's 0 HP loan and the Target Loan, though all three feed the performance bonus.

# **6. Pay-to-Pass System**

Each BP_LevelSegment actor has a trigger volume at its entrance. On player overlap, a PayWindow countdown begins (duration configurable per segment). The fee amount scales with segment difficulty and current level budget state — if the director is at Tier 3, fees increase because the level 'knows' the player is winning.

### **If the player pays**

- Fee routes to BossUpgradePool (not back into LevelBudget)

- Segment opens immediately

- Director receives a free upgrade allocation, selecting from a weighted list based on CurrentApproach: Kill buys a damage upgrade, Drain buys a resource-sapping upgrade, Trick buys a routing obstruction upgrade, Taunt buys a counter-aggression upgrade

### **If the player does not pay**

- Segment opens normally after PayWindow expires

- No director upgrade allocation

- Player retains more resources but faces the director's default configuration

BossUpgradePool is a float held by the director. It feeds the Target Budget (Design Reference → Target System): TargetBudget = Baseline + PlayerPosition x WealthScale + BossUpgradePool + TargetLoan. It is the only part of the Target's strength that changes mid-day. Specific boss upgrade selections remain TBD with the boss system.

# **7. Level Modification System**

Each BP_LevelSegment holds an array of ModSlots — tagged actor references with metadata covering slot type (obstacle / enemy spawn / trap), location in segment, and current assignment.

The director holds a SegmentQueue — an ordered list of unvisited segments. Each decision cycle, if budget permits and approach is relevant, the director calls ModifySegment(SegmentRef, ApproachHint), which:

- Reads the player's current ability loadout from PlayerState

- Queries a counter-ability data table (player ability type mapped to effective counters)

- Selects modifications suited to the current approach

- Writes assignments to the segment's ModSlots

- Marks slots as locked to prevent re-evaluation

When the player enters a segment, BP_LevelSegment reads its assigned slots and populates them. Segments are passive executors — all decision-making lives in the director.

Counter-ability logic is most active during Trick and Taunt modes, where the director has the strongest incentive to use player-specific counters rather than generic difficulty pressure.

# **8. Architecture Summary**

Responsibility separation — the director never directly places actors. It issues spawn requests to EnemyManager and modification requests to BP_LevelSegment instances. This keeps debugging clean and makes a director decision log straightforward to add without coupling it to spawn or placement logic.

| **Actor** | **Responsibilities** |
| --- | --- |
| GameMode | Spawns BP_AIDirector on level load |
| BP_AIDirector | Decision loop, PlayerSnapshot, approach weights, budget tracking, BossUpgradePool, SegmentQueue |
| EnemyManager | Receives spawn requests from director; tracks active enemy array; reports kills |
| BP_LevelSegment | Holds ModSlots; fires EntranceTrigger; executes director assignments on player entry |
| Individual Enemies | Report death via event dispatcher; do not self-manage in the director's array |

# **9. Open Questions**

| **Topic** | **Question** |
| --- | --- |
| Kill refund | "Portion refunded to player side" on a kill is unclear. Is this the kill cash payout, or a separate refund? |
| Brand awareness | Approaches react to HP, money, kills and pace, but not to the relationship web, Worth or the player's ammo brand. Should the Director pick enemy primary brands against the player's loadout? |
| Counter-ability inputs | ModifySegment reads a "player ability loadout," but the player's loadout is ammo, tags, gun parts and cards. The counter table needs those as its inputs. |
| Pay-to-pass incentive | Skipping the fee only costs the PayWindow wait. Should waiting have a cost (Director escalates or replenishes while the player waits)? |
| Stolen loot | Should the snapshot track CarriedValue so the Director can respond to theft (see Design Reference → Fence & Hoard)? |
| Missing dependencies | No doc yet for store / segment generation or the enemy and trap roster with its BudgetCost data table. The budget system can't be tuned without them. |

*Design subject to revision as enemy types, traps, and boss systems are developed.*
