---
layout: single
title: "How My Bot Boosts a Creep – A Detailed Explanation for Beginners"

toc: true
toc_sticky: true

comments: true
---

# How My Bot Boosts a Creep – A Detailed Explanation for Beginners

Boosting is one of the most important steps to becoming a veteran **Screeps** player. You’ll eventually need boosted creeps to attack high-level rooms or defend against them — and once you can boost efficiently, you can also use those boosted creeps to accelerate your economy.

In this post, I’ll explain how my bot handles boosting from start to finish — including how it requests boosts, prepares resources, and executes the actual boosting process.

---

## 1. Boost Request

Whenever my bot requests a creep to be spawned and boosted, it also creates a boost request object in the room’s memory:

```js
room.memory.boostRequests = [
  {
    creepName: "Attacker_123",
    requestedAt: Game.time,
    resources: {
      XKH2O: 660, // Resource type and amount
      XLHO2: 540,
      XZHO2: 300,
    }
  },
  ...
];
````

Each `boostRequest` contains:

1. The creep name to be boosted.
2. The `Game.time` when the request was made.
3. The resource types and required amounts for each boost.

While the creep is being spawned, another creep in the same room starts preparing everything needed for the boost.

---

## 2. Boosting Process Overview

My bot processes boost requests from **oldest to newest**.
Each request goes through three phases: **Gather → Prepare → Boost**.

---

### 🧺 Gather Phase

1. **Check resources:**
   Make sure all the required minerals and energy are available to perform the boost.

2. **Import if missing:**
   If some resources are missing, request them via the room’s terminal from other rooms.

3. **Switch to prepare phase:**
   Once all resources are ready, the system transitions to the next step.

---

### ⚗️ Prepare Phase

1. **Stop lab reactions:**
   If the labs are currently running reactions, stop them first.

2. **Courier transfers:**
   Courier creep transport the correct resources to the appropriate labs.

3. **Batch preparation:**
   If there are multiple boost requests, try to prepare all of them — again from the oldest to the newest.

4. **Ready check:**
   If the target creep has finished spawning and everything is prepared, the bot switches to the boost phase.

---

### ⚡ Boost Phase

1. **Move the creep:**
   The target creep moves to the prepared labs.

2. **Perform boosting:**
   Each lab boosts the creep until the entire process completes.

---

## 3. Tips and Safety Checks

Here are a few lessons I learned the hard way:

1. ✅ **Check if the target creep is alive** and waiting before starting the boost phase.
2. ⚗️ **Select labs wisely** — prefer labs that already contain the required minerals, and skip those currently used for other resources.
3. ⚡ **Don’t forget energy** — labs need energy as well as minerals to perform the boost!
4. 🔄 **Handle failures** — if something goes wrong, return to the gather or prepare phase instead of getting stuck.
5. 🤝 **For duos or quads**, make the creeps that spawn first get renewed until all teammates are ready.
6. ⏳ **Timeouts matter** — if a boost request makes no progress after a certain time, give up and move to the next one.

---

## 4. Final Thoughts

So that’s how my bot handles boosting — a full cycle from request to completion, with recovery in case of failure.

Once you’ve built a stable boosting pipeline, you’ll unlock a huge new layer of Screeps gameplay.
Try spawning a boosted singleton with **tough** and **heal** boosts, and send it to your neighbor — it’s a great way to test your system (and maybe start a small war 😄).

```
```
