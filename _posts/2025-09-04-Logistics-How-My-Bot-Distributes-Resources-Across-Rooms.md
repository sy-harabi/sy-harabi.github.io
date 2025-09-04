---
layout: single
title: "Logistics – How My Bot Distributes Resources Across Rooms"

toc: true
toc_sticky: true

comments: true
---

# Logistics – How My Bot Distributes Resources Across Rooms

Today, I want to cover **logistics**. Many players manage logistics using a **per-room method**: they try to distribute resources equally to all rooms.  
While simple, I think this approach creates inefficiencies — unnecessary `terminal.send` operations, resource ping-pong between rooms, and wasted CPU.  

Instead, I chose to treat logistics as an **empire-wide (bot-level) system**. My guiding principles are:

1. **Always think at bot-level if possible.**  
2. **Do not send resources unless necessary.**

Here’s how I currently handle logistics.

---

## Managing Surplus and Shortage of Raw Minerals

My bot sets a `threshold_per_room` for each mineral. For the whole empire, this becomes:

```
threshold = threshold_per_room * myRooms.length
```

- **Surplus case:**  
  If a mineral amount is *far above* this threshold, the bot sells it.  
  By the pigeonhole principle, some rooms must be holding more than `threshold_per_room`. Those rooms are chosen to sell the resource.

- **Shortage case:**  
  If a mineral amount is *far below* the threshold, the bot buys it.  
  By the same logic, some rooms must be holding less than `threshold_per_room`. Those rooms buy the resource.

This ensures that buy/sell actions naturally balance the empire without constant redistribution.

---

## Choosing Lab Process Targets

When labs are idle and looking for a new objective, my bot selects the boost with the **lowest ratio**:

```
(current amount) / (desired amount)
```

across the entire empire, but only among boosts where the bot has enough ingredients available.

- **Empire-wide view:**  
  The `current amount` includes both what we already have **and** what is currently being produced in other rooms.  
  This prevents multiple rooms from redundantly producing the same boost.

---

## Handling Resource Overflow

Because my bot doesn’t equalize resources between rooms, some rooms occasionally overflow with resources.  
When that happens:

1. If **energy stockpile is above a certain threshold**, push out energy.  
2. Otherwise, push out whichever resource the room has the most of(Other than energy), to the **nearest room with enough space**.

This keeps rooms from stalling due to full storage.

---

## Handling Resource Requests

There are several situations where a room may require resources it doesn’t currently have.  
In such cases, my bot allows targeted `terminal.send()` operations. The guiding rule is: **only send when a room explicitly needs something.**

### Typical Scenarios

1. **Lab ingredient requests:**  
   - When a room wants to make a boost but lacks ingredients.  
   - Other rooms must have the ingredients (since the target was chosen with that assumption).

2. **Creep boost preparation:**  
   - When a room wants to boost a creep but lacks the boosts.  
   - It requests them from rooms that have the boosts.  
   - Since a full creep spawn takes ~150 ticks, usually there’s plenty of time to prepare.

3. **Funneling for RCL growth:**  
   - Extra energy is sent to the room where I want to **raise the RCL as quickly as possible**.  

4. **Low energy recovery:**  
   - When a room’s energy drops too low to sustain operations, other rooms send emergency energy.  

5. **Power processing:**  
   - When the bot wants to process power, usually some rooms hold most of the power (often close to highways where power banks occur).  
   - In this case, the bot distributes power to other rooms so that multiple rooms can process it simultaneously, increasing the speed.

6. **Nuker preparation:**  
   - When a room wants to **fill a nuker** and needs ghodium, it requests it from other rooms.  

---

In all of these scenarios, the system selects the **closest suitable room** to minimize transfer cost and avoid resource ping-pong.

---

## Closing Thoughts

By shifting logistics to the **bot-level**, I avoid constant rebalancing and keep resource movement purposeful.  
This approach reduces CPU overhead, prevents resource ping-pong, and ensures that logistics always serve empire-wide goals rather than per-room symmetry.
