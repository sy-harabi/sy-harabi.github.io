---
layout: single
title: "An Emergent Mission Architecture for Screeps"

toc: true
toc_sticky: true

comments: true
---

# An Emergent Mission Architecture for Screeps

_How I structure my bot to handle strategy, warfare, and large-scale decisions without drowning in micro-management._

---

## Why I Rewrote My Bot

Like many Screeps players, my first bot grew organically: creeps made local decisions, special cases piled up, and every new feature felt harder than the last. Combat logic was especially painful—tactical decisions leaked everywhere, and it was difficult to answer simple questions like:

> _Is this war actually working?_

When I rewrote my bot, I committed to two guiding ideas:

1.  **Top-down control where it matters** Creeps shouldn’t decide _why_ they act. Higher-level systems should.
2.  **Natural task decomposition** Large goals should break down into smaller ones without relying on a rigid planner.

The result is a **mission-based architecture**, where Managers act as the brains of the empire and Missions act as the specialized tools they deploy.

---

## High-Level Structure: Managers as the Highest Level

In this architecture, **Managers are the highest-level actors**. They are persistent, "always-on" entities that bridge the gap between global state and specific action.

Managers serve two primary functions:

1.  **Maintenance:** They handle repetitive, baseline responsibilities (e.g., the Room Manager ensures the controller doesn't downgrade).
2.  **Deployment:** They act as the "commanders" who decide when the situation requires a specialized Mission.

| Manager               | Repetitive Responsibility                 | Mission Deployment Example |
| :-------------------- | :---------------------------------------- | :------------------------- |
| **Room Manager**      | Local infrastructure, mining & upgrading | **Remote Defense Mission** |
| **Combat Manager**    | Global threat assessment                  | **Total War Mission**      |
| **Expansion Manager** | Room scouting & planning                  | **Claim Mission**          |

---

## Missions: Goal-Oriented Execution

While Managers are the **"who,"** Missions are the **"what."** A mission represents a concrete objective with a clear success or failure condition.

A useful mental model is chess:

- The **Manager** is the chess player.
- The **Mission** is the specific opening or strategy being executed.
- **Creeps** are the pieces.

### Recursive Decomposition (The Mission Chain)

Missions are most powerful when they spawn sub-missions. This allows for complex behavior to emerge from simple, nested logic. A high-level mission defines the _intent_, while its children handle the _tactics_.

**Example Mission Chain:**
`Total War` -> `Siege` -> `Quad`

1.  **Total War (deployed by Combat Manager):** Tracks the overall economic drain on the enemy and decides which rooms to pressure.
2.  **Siege:** Focuses on a specific room, managing staging points and wall-breaking progress.
3.  **Quad:** A tactical unit mission that spawns and handles four creeps in formation to execute the siege's goals.

---

## How Missions Communicate

All coordination happens through **mission memory**. The guiding rule is simple:

> **A mission may read any mission’s memory, but may only write to its own.**

### Example Decision Flow

1.  A **Quad Mission** loses a creep and records the loss in its memory.
2.  A **Siege Mission** observes the creep loss flag in its child Quad's memory and adapts future squad composition.
3.  Continued poor results cause the siege to stagnate.
4.  The **Total War Mission** observes the lack of progress across multiple Sieges and decides to pivot the empire's resources elsewhere.

No explicit reporting or signaling is required—coordination emerges from shared observation of state.

---

## Mission Memory Structure

All missions are stored in a double-layered structure: `Memory.missions[type][id]`. This allows for fast lookups and easy iteration.

### Example

```js
Memory.missions["siege"]["W1N1"] = {
  type: "siege",
  id: "W1N1",
  targetRoomName: "W1N1",
  childMissions: [
    { type: "quad", id: "W1N1_17253400" }
  ],
  status: {
    isEffective: false,
    netDamage: -1500
  }
}
```

---

## The Execution Loop

The daily life of the bot follows a predictable cycle, moving from high-level observation down to individual action:

1.  **Analyze:** **Managers** maintain the baseline bot functions and scan the world state to identify strategic needs. 
2.  **Deploy:** Based on the analysis, Managers spawn or update **Missions**. For example, the Combat Manager may deploy a `Total War` mission if a player is being mean. The `Total War` mission then recursively deploys several `Siege` missions toward that player's rooms. 
3.  **Resolve:** High-level missions read the memory of their child missions to evaluate progress. For example, a `Siege` mission checks its `Quad` mission's results to see if the room's defenses are cracking.
4.  **Act:** Missions update their own state and issue direct orders to their assigned **Creeps**. In this phase, a `Quad` mission control creeps and reports how much damage it is dealing, feeding the loop for the next decision making.

This sequence ensures that **execution remains local**, while **evaluation remains strategic**.
