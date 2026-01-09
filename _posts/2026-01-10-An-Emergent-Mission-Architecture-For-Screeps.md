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

1. **Top-down control**: Creeps should not decide _why_ they act. They should only execute _how_.
2. **Emergent hierarchy**: High-level goals should naturally decompose into smaller ones without a rigid planner.

The result is a **mission-based architecture** inspired by Hierarchical Task Networks (HTN), but adapted specifically for Screeps.

---

## The Core Idea: Missions as Strategists

In this architecture, **missions are the thinking units**.

A useful mental model is chess:

- A **mission** is the chess player.
- **Creeps** are the pieces.

Creeps don't ask _"What should I do next?"_ Instead, missions decide goals and issue commands. Creeps simply execute those commands as efficiently as possible.

This inversion—strategy above execution—simplifies reasoning about the bot.

---

## High-Level Structure: Managers and Missions

At the top level, the bot is divided into **Managers** and **Missions**.

### Managers: Strategic Observers

Managers does repetitive jobs, **observe global state** and decide _which missions should exist_.

Typical managers include:

- **Room Manager** – Handles internal room needs (mining, building, upgrading) and deploys small external missions like remote defense.
- **Intel Manager** – Scans the world, tracks enemies, and maintains strategic visibility.
- **Spawn Manager** – Centralized creep request handling and spawn queue management.
- **Mission Manager** – Owns mission lifecycles: creation, execution, and cleanup.
- **Combat Manager** – Evaluates enemy players and decides whether war is justified.
- **Expansion Manager** – Chooses when and where to claim or abandon rooms.

Managers think in terms of _conditions_ and _intent_, not movement or combat.

---

## Missions: Goal-Oriented Execution

A **mission** represents a concrete objective, usually scoped outside of your core rooms.

Examples:

- Attack a room
- Defend against an invader
- Harvest a power bank
- Claim a new room

Each mission:

- Owns its assigned creeps
- Orchestrates sub-missions if needed
- Reports results upward through structured memory

### Emergent Hierarchy in Practice

Missions do not have hard-coded tiers, but hierarchies naturally form:

1. **Total War Mission** (Goal: Remove a player) $\rightarrow$ Deploys multiple **Siege Missions**.
2. **Siege Mission** (Goal: Attack a specific room) $\rightarrow$ Deploys several **Quad Missions**.
3. **Quad Mission** (Goal: Tactical combat) $\rightarrow$ Spawns and controls the actual creeps.

Each layer focuses on a different level of abstraction. High-level missions never issue move commands, and low-level missions never decide _why_ the fight exists.

---

## How Missions Communicate (Without Chaos)

All communication happens through **mission memory**.

A strict rule keeps the system sane:

> **Missions may read other missions' memory, but never write to it.**

This enforces one-way information flow:

- Children _report facts_
- Parents _interpret meaning_

### Example Decision Flow

1. A **Quad Mission** loses a creep and records the event.
2. The parent **Siege Mission** observes reduced effectiveness and adapts future quad composition.
3. If overall damage trends remain negative, the Siege Mission reports failure.
4. The **Total War Mission** decides whether to escalate, retarget, or end the war entirely.

No mission ever directly commands another mission. Strategy emerges from observation.

---

## Mission Memory Structure

All missions live in a double-layered structure:

```
Memory.missions[type][id]
```

### Example

```js
Memory.missions["siege"]["W1N1"] = {
  type: "siege",
  id: "W1N1",
  targetRoom: "W1N1",
  childMissions: [
    { type: "quad", id: "W1N1_17253400" },
    { type: "quad", id: "W1N1_17253550" },
  ],
  status: {
    isEffective: false,
    netDamage: -1500,
  },
}
```

Missions describe _what happened_, not _what should be done next_.

## The Communication Loop

Because every mission memory contains its own `type` and `id`, any mission can resolve references to its children (or parents) from anywhere in the codebase.

During the `run()` phase of a high-level mission—such as a **Total War Mission**—the logic consistently follows the same four-step loop:

1. **Analyze**
   Iterate over the `childMissions` array stored in the mission’s own memory.

2. **Access**
   Use each child descriptor’s `type` and `id` to resolve the actual memory object at:

   ```
   Memory.missions[child.type][child.id]
   ```

3. **Evaluate**
   Read the child mission’s reported `status` fields (for example, `isEffective` or `netDamage`).

4. **Act**
   If multiple children are failing, the parent mission decides the next strategic move—such as switching targets, or ending the campaign entirely.

Crucially, **execution and evaluation are separated**:

* **Execution** happens inside child missions (moving creeps, fighting, pathing).
* **Evaluation** happens in parent missions (judging whether the approach is working).

Because parent missions only *read* child memory—and never mutate it—this loop remains stable, predictable, and easy to debug even as the hierarchy grows deeper.

