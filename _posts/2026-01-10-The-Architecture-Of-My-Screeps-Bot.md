---
layout: single
title: "The Architecture of My Screeps Bot: An Emergent Mission Hierarchy"

toc: true
toc_sticky: true

comments: true
---

# The Architecture of My Screeps Bot: An Emergent Mission Hierarchy

Hi everyone! Today I want to share the structure of my bot, specifically how it handles complex operations through a mission-based system.

When I decided to rewrite my bot, I had two core philosophies in mind:

1.  **Top-Down Control:** Instead of creeps making individual decisions (bottom-up), I think of each mission as a **chess player**. The mission is the strategist, and the creeps are the pieces it moves.
2.  **Emergent Hierarchical Task Networks (HTN):** My bot uses an HTN-inspired approach where missions naturally form a hierarchy. High-level goals are broken down by spawning sub-missions, allowing for deep abstraction and easy commanding.

---

## 1. Managers: The High-Level Actors

Managers are the "architects" of my bot. They oversee the general state of the empire and initiate missions when needed.

- **Room Manager:** Manages internal room tasks like mining, building, and upgrading. It also handles basic external needs by deploying **Remote Defense** or **Power Bank Harvest** missions.
- **Intel Manager:** Scans the world, tracks enemy activity, and maintains vision of key rooms.
- **Spawn Manager:** Processes creep requests and manages spawn queues.
- **Mission Manager:** The lifecycle controller. It runs existing missions and cleans up those that have expired.
- **Combat Manager:** The grand strategist. It evaluates enemies and decides whether to start a war. If it targets a player, it deploys a **Total War Mission**.
- **Expansion Manager:** Decides where to claim or unclaim rooms. When it finds a suitable target, it deploys a **Claim Mission**.

---

## 2. Missions: Goal-Oriented Tasks

Missions are tasks designed to achieve objectives, usually outside of my own rooms. Every mission operates with its own assigned creeps or sub-missions.

### Emergent Hierarchy in Action

While missions don't have hard-coded "tiers," they naturally build a network by spawning one another. For example:

1.  **Total War Mission** (Goal: Remove a player) $\rightarrow$ Deploys multiple **Siege Missions**.
2.  **Siege Mission** (Goal: Attack a specific room) $\rightarrow$ Deploys several **Quad Missions**.
3.  **Quad Mission** (Goal: Tactical combat) $\rightarrow$ Spawns and controls the actual creeps.

### How They Work

- **Chess Player Logic:** Each mission requests creeps from a room and "plays" them. If priorities shift, creeps can even be exchanged between missions.
- **Persistent Memory:** Every mission is stored in a structured way within `Memory.missions`. To keep things organized, I use a double-layered object: `Memory.missions[type][id]`.
- **Intelligence Sharing:** Missions communicate via their memory objects. While any mission can read another's memory, I follow a strict rule: **Missions read from others, but never write to them.** This keeps the logic flow clean and prevents messy state conflicts.

**Decision Making Example:**
If a **Quad Mission** loses a creep, it records the event in its memory. The parent **Siege Mission** reads this and may adjust the next Quad's design to be more tanky. If the Siege Mission isn't achieving net damage, it reports this failure. The **Total War Mission** then reads that report and decides whether to **stop the war** entirely or **change the target** to a different user.

---

## Technical Deep Dive: The Mission Memory Structure

For those interested in the implementation, I use a standardized interface for mission memory. This allows parent missions to easily track and resolve their children.

### Double-Layered Memory

By nesting by type and then ID, I can quickly access all missions of a certain category without iterating through the entire mission database.

```javascript
// Structure: Memory.missions[type][id]
Memory.missions["siege"]["W1N1"] = {
  type: "siege",
  id: "W1N1",
  targetRoom: "W1N1",
  // Child missions are stored as descriptors to be looked up later
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

### The Communication Loop

Because every mission memory contains its own `type` and `id` properties, the code can resolve references from anywhere in the codebase. In the `run()` phase of a high-level mission (like Total War), the logic follows these steps:

1.  **Analyze**: Loop through the `childMissions` array stored in its own memory.
2.  **Access**: Use the `type` and `id` from the child descriptor to look up the actual memory object at `Memory.missions[child.type][child.id]`.
3.  **Evaluate**: Check the child’s `status` (e.g., `isEffective` or `netDamage`).
4.  **Act**: If the children are failing, the parent mission logic determines the next strategic move—such as switching targets or ending the campaign entirely.

By separating the **Execution** (the child mission moving creeps) from the **Evaluation** (the parent reading the child’s memory), the bot becomes much more resilient, abstracted, and easier to debug!
