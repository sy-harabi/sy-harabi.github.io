---
layout: single
title: "An ROI-Driven Target Selection System in Screeps: Balancing Threat and Elimination Cost"

toc: true
toc_sticky: true

comments: true
---

In Screeps, choosing which bot to attack is a critical high-level strategic decision. Common approaches to target selection vary widely: remaining entirely passive, declaring unconditional hostility toward everyone, picking targets at random, or focusing purely on the player with the highest raw hostility.

This post introduces an alternative system based on the principle of Return on Investment (ROI). The objective is strictly economic: minimizing the total energy cost inflicted on my colony by external threats, resolved at the lowest possible resource expenditure.

## 1. Infamy: Quantifying Incoming Damage into Energy

To make rational comparisons between different hostile bots, all forms of harassment must be converted into a single, unified metric: **Energy**. The system tracks this under a property called `infamy`.

### Direct Calculations

* **Remote Harassment:** When hostile creeps invade our remote mining rooms, the system estimates the energy cost required to build the invading creeps. For non-combat scouts, this is a flat baseline of `100` energy. For combatants, it uses the exact energy cost of their body parts (e.g., ATTACK, RANGED_ATTACK, HEAL).
* **Creep Casualties:** When a friendly creep is destroyed by an enemy, the infamy value is scaled by the creep's size and remaining lifetime:
  ```
  Infamy Added = (Body Length * Part Energy Max * Ticks to Live) / Creep Lifetime
  ```
  This ensures that losing a freshly spawned, expensive creep adds significantly more infamy than losing a creep that was about to die of old age.

### Heuristics for Complex Interruptions

Certain hostile actions cannot be easily converted into a precise energy value. For these, consistent heuristics are applied to maintain internal logical consistency:
* Power bank mission disruptions
* Direct attacks on owned rooms (e.g., nuke launches)

### Moving Average, Decay, and Locking Mechanics

Because infamy is measured dynamically, the bot accumulates the total damage over a specific tracking period (`CREEP_LIFE_TIME` ticks) and merges it into the existing historical score using an Exponential Moving Average (EMA). This mathematical structure governs how threat scores persist or decline over time:

* **The Decay Mechanism:** Past infamy naturally decays as new cycles with zero hostile activity are processed.
* **The Baseline Floor:** For continuous threats, such as a nearby hostile room generating an infamy penalty every single tick, the EMA ensures this constant influx acts as a persistent baseline floor.
* **The Enemy Lock:** Once a bot accumulates enough infamy to cross a specific threshold, it is marked as an `"enemy"`. For these players, the infamy score is clamped at the enemy threshold baseline and prevented from decaying further—**provided they have successfully breached one of our established rooms (RCL 4 or higher)**. This guarantees that players who have demonstrated the capability to penetrate our core defenses remain permanently flagged as threats, while minor players or those who only breached a low-level expansion outpost are allowed to decay back to neutral.

---

## 2. Durability: Estimation of Elimination Cost

While `infamy` measures the threat, `durability` estimates the cost of eliminating that threat. It answers the question: *How many resources must my bot expend to completely wipe all known rooms of this player?*

To establish a tangible baseline, Durability is defined as the estimated wall hits (where `1.0` durability = 1,000,000 hits) required to clear the target's rooms.

### Hardware Factors

Instead of summing up every single wall and rampart in the room, the bot calculates the **minimum path thickness** to reach critical infrastructure (spawns, towers, terminal). Using Dijkstra pathfinding on exit positions to target structures, the durability score is derived from:
* **Minimum Path Barrier Hits:** The total hit points of the thinnest layer of walls/ramparts blocking access to the core room structures.
* **Energy-to-Repair Conversion:** If the target player has been seen actively repairing structures, the system assumes they will spend their stored energy on defenses. The durability is scaled up by:
  ```
  Barrier Hits + (Stored Energy * Repair Power Constant)
  ```
* **Stored Boosts:** The estimated defensive capabilities are increased based on the opponent's stored Tier 1, Tier 2, and Tier 3 combat boosts.

### Software Multipliers (Intel System)

Raw defensive structures do not tell the whole story; the opponent's defensive code matters. The bot maintains an intel object for each player stored in `Memory`. If the opponent has been seen utilizing advanced defenses, a multiplier scales up their durability score:
* Spawning dedicated rampart defenders (melee or ranged)
* Deploying defensive combat units (especially if utilizing boosts)

---

## 3. The Target Selection Algorithm and Score Modifiers

With both threat (`infamy`) and elimination cost (`durability`) quantified, the strategic planner selects targets by evaluating the overall return on investment.

The bot calculates an evaluation score for each potential target, taking into account a distance penalty:

```
Score = (Durability * Distance Penalty) / Infamy
```

Because the bot uses a **Min-Heap** to evaluate targets, the candidate with the **lowest** score is prioritized. This mathematically favors players who are highly hostile (`infamy` is high), close by (`distance` is low), and cheap to wipe out (`durability` is low).

### The Distance Penalty

To prevent the bot from launching costly campaigns halfway across the world map, a logarithmic penalty is applied based on the room distance between our nearest spawn and the target's territory. This penalty heavily prioritizes local hostiles, while scaling down the strategic priority of distant opponents to reflect the high resource costs and travel-time risks of long-range combat operations.

### Priority Multipliers

To handle critical tactical situations and manage target stickiness, modifiers are applied directly to the score:

* **Invasion Multiplier:** If a player has launched an attack against us recently (`lastInvasion`), their score is halved (doubling their priority in the Min-Heap) to prioritize immediate revenge.
* **Target Stickiness (Hysteresis):** To prevent the bot from frequently switching war targets due to minor updates in structural data or infamy decay, the active target has its score halved. This introduces stickiness, ensuring the bot commits to a war target until a significant shift in the ROI balance occurs.

---

## Conclusion

This system does not offer perfect mathematical precision. The unpredictable variables of combat code and hidden player assets make strict calculations **exceptionally difficult**. However, by framing target selection as a resource allocation problem—weighing energy lost against path-based deployment costs—the bot can manage automated long-term conflicts with reasonable efficiency and zero human intervention.
