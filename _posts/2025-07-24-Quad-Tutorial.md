---
layout: single
title: "Quad Tutorial: Attack the Level 1 Stronghold!"

toc: true
toc_sticky: true

comments: true
---

# Quad Tutorial: Attack the Level 1 Stronghold!

In the [last post](https://sy-harabi.github.io/Quad-Movement-Basics/), we talked about basic quad movement. You’re probably eager to attack an enemy room with your quad—but before that, there are some important things to learn.

Today, we’ll use a Level 1 Stronghold as an example to show how quads can be used in real combat. Strongholds are the bases of NPC Invaders that spawn in the central rooms of each sector. They’re great for testing and improving your attack code: they appear regularly, exist in every sector, and reward you when destroyed.

Once a stronghold is destroyed, Invaders and lesser Invader Cores stop spawning in that sector until the next stronghold spawns (around 75,000 ticks later). So even a low-level stronghold is worth attacking—especially as a beginner practice target.

Let’s dive in!

* This is a tutorial, so I’ll keep things as simple and easy to follow as possible. Once you understand the basics, you can add more features and refine your logic. I’ll post more tips about using quads soon—stay tuned!

---

## 🚧 What to Spawn

A Level 1 Stronghold has only **one tower** and **no creeps**, but that tower can deal up to **600 damage per tick** if you're close enough. Since each `HEAL` part can heal **12 hits per tick**, you need at least **50 HEAL parts** in total just to survive.

To be safe, we’ll use **60 HEAL parts** across the quad. That means each creep should have:

- `10 RANGED_ATTACK`
- `25 MOVE`
- `15 HEAL`

That gives your quad enough healing, mobility, and firepower to survive and take down the stronghold.

* Please note: This is not an optimal strategy for destroying a stronghold—I'm simply using it for practice.

---

## Overview of the Logic

Here’s the general structure of our quad attack logic:

1. Each creep attacks nearby structures or enemies.
2. Creeps heal each other effectively.
3. Creeps move as a quad:
   - A. If in danger, retreat.
   - B. If safe, find a target and move toward it.

Sounds simple, right? Let’s go through each part in detail.

---

## 🗡️ Attack Logic

To keep it simple for now, I'll just make each creep attack any enemy or structure within range 3. If you have your own better logic, use that!

```js
for (const creep of creeps) {
    const targets = creep.pos.findInRange(FIND_HOSTILE_CREEPS, 3)
        .concat(creep.pos.findInRange(FIND_HOSTILE_STRUCTURES, 3));

    if (targets.length > 0) {
        creep.rangedAttack(targets[0]);
    }
}
```

---

## 🛡️ Getting Damage

Before handling healing, we need to know how much damage each creep is expected to take.

1. Scan all enemy towers.
2. For each tile, calculate and store the total incoming damage from towers.

```js
function getDamageAt(room, pos) {

    const towers = room.find(FIND_HOSTILE_STRUCTURES, {
        filter: s => s.structureType === STRUCTURE_TOWER
    });

    let damage = 0;

    for (const tower of towers) {
        const dist = tower.pos.getRangeTo(x, y);
        damage += getTowerDamage(dist)
    }

    return damage;
}

function getTowerDamage(range) {
    if (range <= TOWER_OPTIMAL_RANGE) return TOWER_POWER_ATTACK

    if (range > TOWER_FALLOFF_RANGE) {
        range = TOWER_FALLOFF_RANGE
    }

    return TOWER_POWER_ATTACK * TOWER_FALLOFF * (range - TOWER_OPTIMAL_RANGE) / (TOWER_FALLOF_RANGE - TOWER_OPTIMAL_RANGE)
}
```

---

## ❤️ Heal Logic

There are many healing strategies, but I recommend a simple one that **maximizes the lowest expected health** among your creeps:

1. For each creep, calculate `expectedHits = hits - incomingDamage`
2. Each creep should heal the friendly creep with the lowest expected hits.
3. After healing, update that creep’s expected hits by adding the heal amount.

```js
function healCreeps(room, creeps) {
    const expectedHits = {};

    for (const creep of creeps) {
        const dmg = getDamageAt(room, creep.pos) || 0;
        expectedHits[creep.name] = creep.hits - dmg;
    }

    for (const healer of creeps) {
        let target = null;
        let minExpected = Infinity;

        for (const other of creeps) {
            if (expectedHits[other.name] < minExpected) {
                minExpected = expectedHits[other.name];
                target = other;
            }
        }

        if (target) {
            healer.heal(target);
            expectedHits[target.name] += creep.getActiveBodyparts(HEAL) * HEAL_POWER;
        }
    }
}
```

---

## 🧭 Movement Logic

### 1. Danger Detection

Before moving toward your target (usually the `invaderCore`), check whether the quad is in danger:

- Use the damage map.
- Compare each creep’s expected damage to the total healing your quad can provide.
- If any creep is expected to take more damage than can be healed, **retreat**.

A simple retreat logic:  
* Remember which direction you entered the room from. If in danger, move back to that exit.

```js
function shouldRetreat(room, creeps, totalHealPower) {
    for (const creep of creeps) {
        const damage = getDamageAt(room, creep.pos) || 0;
        if (damage > totalHealPower) {
            return true;
        }
    }
    return false;
}
```

---

### 2. Obstacle Map

To plan movement, we need an **obstacle map**, which shows how hard it is to destroy the obstacles on each tile.

Here’s how to build it:

1. First, scan **all structures in the room** to find the maximum hits among them. We’ll call this `maxHits`.
2. Choose a maximum cost value (e.g. `100`) and compute `unitHits = Math.ceil(maxHits / 100)`
3. For each tile in the room:
   - Use `room.lookForAt(LOOK_STRUCTURES, x, y)` to get all structures on that tile.
   - Sum up their hit points.
   - Compute `cost = Math.min(100, Math.ceil(hitsSum / unitHits))`
   - Store that value in a `CostMatrix`.

Here's the example code:

```js
function getObstacleMap(room) {
    const costMatrix = new PathFinder.CostMatrix();
    let maxHits = 1;

    // Step 1: Find maxHits
    const allStructures = room.find(FIND_STRUCTURES);
    for (const s of allStructures) {
        if (s.hits && s.hits > maxHits) {
            maxHits = s.hits;
        }
    }

    // Step 2: Compute cost per tile
    const unitHits = Math.ceil(maxHits / 100);

    for (let x = 0; x < 50; x++) {
        for (let y = 0; y < 50; y++) {
            const structs = room.lookForAt(LOOK_STRUCTURES, x, y);
            let hitsSum = 0;

            for (const s of structs) {
                if (s.hits ) {
                    hitsSum += s.hits;
                }
            }

            if (hitsSum > 0) {
                const cost = Math.min(100, Math.ceil(hitsSum / unitHits));
                costMatrix.set(x, y, cost);
            }
        }
    }

    return costMatrix;
}
```
---

### 3. Destruction Cost Matrix

Since your creeps move as a 2x2 quad, convert your `CostMatrix`:

- For each tile `(x, y)`, take the max cost of the following 2x2 block:
  - `(x, y)`
  - `(x+1, y)`
  - `(x, y+1)`
  - `(x+1, y+1)`
- Store that value in the new matrix.

---

### 4. Move to Target

Now, use the destruction cost matrix to find a path to the invader core.

- Identify the first structure the quad will encounter on this path—that’s your current target.
- Move toward it using quad movement logic from [my previous post](https://sy-harabi.github.io/Quad-Movement-Basics/).

---

## ✅ That’s It!

With this setup, you should be able to take down a Level 1 Stronghold using a basic quad. But this is just the beginning. Here are some ideas to take your system further:

1. Boost your creeps for higher damage output and durability.
2. Detect danger more precisely and proactively, considering enemy creeps.
3. Improve retreat logic.
4. Change targets if something goes wrong.
5. Use `RMA` (Ranged Mass Attack) to destroy enemy structures more effectively.
6. Create smarter healing strategies.
7. Use multiple quads together without them getting stuck.
8. Improve CPU efficiency by caching and reusing previously computed results.

I’ll cover these advanced topics in future posts!

Stay tuned, and happy coding!
