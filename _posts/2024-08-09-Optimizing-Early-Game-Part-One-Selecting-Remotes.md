---
layout: single
title: "Optimizing Early Game Economy – Part 1: Selecting Remotes"

toc: true
toc_sticky: true

comments: true
---

Hello! Recently, I've been working on optimizing the early game economy for my new bot. I managed to reach RCL 5 in just under 15,900 ticks, a significant improvement from my previous record of 16,900 ticks. If you're curious about how I did it, check out this [YouTube video](https://youtu.be/JBAmxd6hq_o?si=D3pd0PAb9RgTWVAQ).

<img src="https://github.com/user-attachments/assets/9d2cc685-b27e-4bde-b3cc-b499ed92d2dc" style="width:100%">

Now, I'd like to share the strategies that helped me achieve this milestone. While some aspects are a bit complex, the overall approach is fairly straightforward. Let’s dive in.

## 1. Key Constraints

The primary constraint in optimizing the early game economy is spawn time. Since you only have one spawn, it's crucial to use it wisely. Here’s a breakdown:

- **Spawn Rate:** You can spawn 500 parts over 1,500 ticks, meaning you can have a maximum of 500 parts at any given time.
- **Creep Types:** Most of the time, your spawn will produce one of these creeps:

  - **Miners:** Harvest energy from sources throughout their lifespan.
  - **Haulers:** Transport harvested energy to where it's needed, such as storage, spawn, or extensions.
  - **Upgraders/Builders:** Use the transported energy to upgrade the controller or construct structures.

<img src="https://github.com/user-attachments/assets/a1fe3a13-e8d3-42ff-b27f-2f6d1facfcb2" style="width:100%">

Other important creeps include:

- **Porters:** Withdraw energy from storage and refill the spawn and extensions.
- **Reservers:** Reserve controllers to increase energy production.

<img src="https://github.com/user-attachments/assets/dfba322b-4cd8-4507-b2dd-607fbb866b06" style="width:100%">

## 2. Basic Strategy

Given the spawn time limitation, the challenge is to balance the number of miners, haulers, upgraders, builders, reservers, and porters. Here’s how to approach this balancing act:

### A. Miners vs. Haulers

If you have too many miners relative to haulers, energy will accumulate at the sources and decay. Conversely, if you have too many haulers and not enough miners, there won't be enough energy to transport.

### B. Earning vs. Using

If your setup leans too heavily towards miners and haulers, you'll generate plenty of energy but won't upgrade quickly. On the other hand, if you prioritize upgraders and builders, you may run out of energy and stall your upgrades.

## 3. Additional Considerations

### A. Construction Timing

Reaching RCL 3 and building 7 extensions as quickly as possible is crucial. This allows you to spawn reservers, doubling the energy regeneration from 1,500 to 3,000 in remote sources. Therefore, I avoid building roads or containers until I can spawn a reserver.

### B. Spawn Prioritization

Closer sources are more efficient because they require fewer haulers. Prioritize earning over using because you need to keep spawning creeps to maximize your spawn time. Typically, the spawn order is:

1. Miner for the closest source.
2. Hauler.
3. Miner for the next closest source.
4. Hauler.
5. Upgraders and builders once all miners and haulers are spawned.

## 4. Why Selecting Remotes?

As mentioned earlier, balancing energy production (earning) and energy consumption (using) is crucial. At the same time, it's important to prioritize spawning miners and haulers before transitioning to spawning upgraders. But how do we determine when it's time to stop spawning miners and haulers and start focusing on upgraders? To make this decision, we need to perform some calculations to determine the optimal number of sources to harvest.

## 5. Calculation

Now, let's get into the calculations necessary for selecting which sources to mine. The goal is to balance ambition with practicality—aim too high and you'll stagnate; aim too low and you won't progress much. Here’s what you need to calculate:

### A. Net Income for Sources

`Net income = Energy generated per tick - Energy used to spawn miners - Energy used to spawn haulers - Energy used to repair containers - Energy used to spawn reservers`

Here are more precise explanations. We will not consider the center rooms and source keeper rooms since we're talking about the early game.

1. `Energy generated per tick = (owned || reserved) ? 10 : 5`
2. `Energy used to spawn miners = (cost of miner body composition you use considering energy per tick) / (CREEP_LIFE_TIME - (traveling distance to the source))` where `CREEP_LIFE_TIME = 1,500`.
3. `Number of CARRY parts you need = (Energy generated per tick) * 2 * (traveling distance to the source) / CARRY_CAPACITY`. Where `CARRY_CAPACITY = 50`.
4. `Energy used to spawn haulers = (Number of CARRY parts you need) * (using roads? 75 : 100) / CREEP_LIFE_TIME`
5. `Energy used to repair containers = owned ? 1 : 0.5`
6. `Energy used to spawn reservers = 650 / (CREEP_CLAIM_LIFE_TIME - (traveling distance to the controller))` where `CREEP_CLAIM_LIFE_TIME = 600`. We are assuming to use 1 CLAIM part and 1 MOVE part for all the time.

You can omit the last two terms if not applicable.

### B. Spawn Usage for Sources

`Spawn usage = Spawn time for miners + Spawn time for haulers + Spawn time for reservers`

Again, here are more precise explanations:

1. `Spawn time for miners = (number of body parts you used for miner considering energy per tick) * CREEP_LIFE_TIME / (CREEP_LIFE_TIME - (traveling distance to the source))`
2. `Spawn time for haulers = (Number of Carry parts you need) * (using roads? 1.5 : 2)`
3. `Spawn time for reservers = 2 * CREEP_CLAIM_LIFE_TIME / (CREEP_CLAIM_LIFE_TIME - (traveling distance to the controller))`. Notice that we should consider that reservers have a shorter lifetime.

Again, omit the last term if not applicable.

### C. Adjusted Spawn Usage Considering Upgraders

This might seem odd, but more income means more upgraders. Increase spawn usage by the amount needed to spawn upgraders. Add spawn usage approximately `net income × (1~2)` more parts, depending on your upgrader body composition.

## 6. Selecting Remotes

Finally, let's talk about remote selection, which is similar to the knapsack problem—a well-known issue in computer science. Typically, dynamic programming is used to solve the knapsack problem. If you’re unfamiliar with this, check out this [Wikipedia article](https://en.wikipedia.org/wiki/Knapsack_problem).

### Applying Dynamic Programming to Remote Selection

1. **Objective:** Select remotes without exceeding spawn usage while maximizing net income.
2. **Value and Weight:** Net income is the value, and spawn usage is the weight.
3. **Maximum Weight(Spawn usage)** You can have a maximum of 500 parts. You have to lower this number if you want to spawn other creeps like porters. And it's usually good to have some buffer since it's hard to make spawning perfect.

By applying dynamic programming, you can find the best combination of remotes. Here’s a simple pseudo-code example:

```javascript
// Input:
// Remote stats (stored in array s)
// Values (stored as 'value' in each remote stat)
// Weights (stored as 'weight' in each remote stat)
// Number of distinct remotes N (s has length N)
// Knapsack capacity W (maximum spawn usage for remotes)

array m[0, 1, 2, … , W]
result array r[0, 1, 2, … , W]

for i from 0 to W do:
    m[i] = 0

for i from 0 to W do:
    r[i] = []

for i from 0 to N-1 do:
    for j from W to 1 do:
        if m[j] + s[i].value > m[j + s[i].weight] then:
            m[j + s[i].weight] = m[j] + s[i].value
            r[j + s[i].weight] = [...r[j], s[i]]
        if s[i].value > m[s[i].weight] then:
            m[s[i].weight] = s[i].value
            r[i] = [...r[0], s[i]]

// Find the index i with the highest m[i]; r[i] will be the optimal remote selection.
```

This is a basic example. You can modify the code to consider individual sources within each remote or to avoid using a remote if there’s no accessible path between it and your main room.

If you want to see a real example, you can check out my code [here](https://github.com/sy-harabi/harabiBot_early_eco_example/blob/6fb4ffa5c4546b47e5a91a8a1e0e95e2d610a8a1/src/manageSource.js#L1200).

## Conclusion
In many cases, simply selecting remotes from closest to farthest while monitoring spawn time works well. However, in edge cases—like the one shown below—dynamic programming can offer better results. In this scenario, my bot initially selected the red remote (14), but after implementing dynamic programming, it switched to the blue remotes (14 and 15), which was a more efficient choice.

<img src = "https://github.com/user-attachments/assets/8820cc20-573b-47b7-ad87-c163758e992f" style = "width:100%">


If you want to test my bot on your own, the codebase you can use is [here](https://github.com/sy-harabi/harabiBot_early_eco_example).
Note that it's not that fast compared to the version I used for speedrun, since I patched several features from that version for late game. But I think it can still reach rcl5 under 20k for sure, in the room W7N3 of default map.

Stay tuned for more insights in the next part of this series!
