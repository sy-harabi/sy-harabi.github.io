---
layout: single
title: "Logistics – How My Bot Distributes Resources Across Rooms"

toc: true
toc_sticky: true

comments: true
---

Today, I want to talk about logistics in Screeps.  
A lot of players manage logistics on a **per-room basis**, where each room tries to keep resources balanced. While this approach works, I think it introduces some inefficiencies — like redundant `terminal.send` calls or even resource ping-pong between rooms.  

Instead, I handle logistics at the **bot level**. This keeps things simpler and avoids unnecessary shuffling. My guiding principles are:  

1. Always think at the **bot level**, not per-room.  
2. Don’t send resources unless it’s absolutely necessary.  

Here’s how that looks in practice:  

---

## Market Strategy  

I define a `threshold_per_room` for each mineral. For the empire, this becomes:  

```
threshold = threshold_per_room * myRooms.length
```

- **Surplus:**  
  If the empire has much more than this threshold, the bot sells some.  
  By the pigeonhole principle, some room must be holding more than its fair share (`threshold_per_room`), so that room handles the selling.  

- **Shortage:**  
  If the empire has much less than the threshold, the bot buys some.  
  Again, some room must be below its fair share, so those rooms handle the buying.  

This keeps the empire balanced with minimal transactions.  

---

## Labs and Boost Production  

When labs are free, the bot checks which boost has the **lowest ratio of (current amount / desired amount)** across the empire.  
- That boost becomes the objective for the labs.  
- The “current amount” also includes boosts currently being produced in other rooms, so production doesn’t overlap.  

This ensures boosts are produced efficiently and only where they’re most needed.  

---

## Handling Overflow  

Because I don’t rebalance resources constantly, some rooms may overflow with energy or minerals.  

Each room has an **energy threshold**. If a room exceeds it — or both storage and terminal are nearly full — the bot pushes resources elsewhere:  
- If it’s energy, send energy.  
- Otherwise, send whichever resource is most abundant in that room.  
- Always send to the **closest room with enough free space**.  

This way, the empire stays healthy without wasting CPU on endless balancing.  

---

## Boost Requests  

When a room needs boosts (for spawning combat creeps), the bot:  
1. Finds the nearest room that has the needed boosts.  
2. Requests the transfer.

Since spawning a full-sized creep takes **150 ticks**, there’s plenty of time to prepare.  

---

## Final Thoughts  

Managing logistics at the **bot level** avoids wasted CPU, network congestion, and terminal spam. Instead of juggling every room, I let surpluses and shortages resolve naturally with a few well-placed transfers.  

It’s clean, efficient, and makes empire-wide logistics far more manageable.
