---
layout: single
title: "Optimizing Early Game Economy – Part 1: Selecting Remotes"

toc: true
toc_sticky: true

comments: true
---

# Optimizing Early Game Economy – Part 1: Selecting Remotes

Hello! Recently, I've been working on optimizing the early game economy for my new bot. I managed to reach RCL 5 in just under 15,900 ticks, which is a significant improvement from my previous record of 16,900 ticks. If you're curious about how I did it, check out this [YouTube video](https://youtu.be/JBAmxd6hq_o?si=D3pd0PAb9RgTWVAQ).

Now, I'd like to share the strategies that helped me achieve this milestone. While some aspects are a bit complex, the overall approach is fairly straightforward. Let’s dive in.

## 1. Key Constraints

The main constraint in optimizing the early game economy is spawn time. Since you only have one spawn, it's crucial to use it wisely. To break it down:

- You can spawn 500 parts over 1,500 ticks, meaning you can have a maximum of 500 parts at any given time. Most of the time, your spawn will produce one of these creeps:

  - **Miners:** Harvest energy from sources throughout their lifespan.
  - **Haulers:** Transport harvested energy to where it's needed, such as storage or spawn extensions.
  - **Upgraders/Builders:** Use the transported energy to upgrade the controller or construct structures.

Other important creeps include:

- **Reservers:** Reserve controllers to increase energy production.
- **Porters:** Withdraw energy from storage and refill the spawn and extensions.

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

## 4. Calculation

Now, let's get into the calculations necessary for selecting which sources to mine. The goal is to balance ambition with practicality—aim too high and you'll stagnate, aim too low and you won't progress much. Here’s what you need to calculate:

### A. Net Income for Sources

Net income = Energy generated per tick - Energy used to spawn miners - Energy used to spawn haulers - Energy used to repair containers - Energy used to spawn reservers. 

You can omit the last two terms if not applicable.

### B. Spawn Usage for Sources

Spawn usage = Spawn time for miners + Spawn time for haulers + Spawn time for reservers. 

Again, omit the last term if not applicable.

### C. Adjusted Spawn Usage Considering Upgraders

This might seem odd, but more income means more upgraders. Increase spawn usage by the amount needed to spawn upgraders. For every 1,500 ticks, spawn approximately net income × (1~2) more parts, depending on your upgrader body composition.

## 5. Selecting Remotes

Finally, let's talk about remote selection, which is similar to the knapsack problem—a well-known issue in computer science. Typically, dynamic programming is used to solve the knapsack problem. If you’re unfamiliar with this, check out this [Wikipedia article](https://en.wikipedia.org/wiki/Knapsack_problem).

### Applying Dynamic Programming to Remote Selection

1. **Objective:** Select remotes without exceeding spawn usage while maximizing net income.
2. **Value vs. Weight:** Net income is the value, and spawn usage is the weight. I usually set a buffer, so I aim for a maximum of 480 parts.

By applying dynamic programming, you can find the best combination of remotes. Here’s a simple pseudo-code example:

```javascript
// Input:
// Remote stats (stored in array s)
// Values (stored as 'value' in each remote stat)
// Weights (stored as 'weight' in each remote stat)
// Number of distinct remotes N (s has length N)
// Knapsack capacity W (maximum spawn usage for remotes)

array m[0, 1, 2, …, W]
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

If you want to see a real example, you can check out my code [here](https://github.com/sy-harabi/harabiBot_early_eco_example/blob/d185cd2716b8e7cee1bd126e94e64f7241f513d3/src/manageSource.js#L1189).

## Conclusion
In many cases, simply selecting remotes from closest to farthest while monitoring spawn time works well. However, in edge cases—like the one shown below—dynamic programming can offer better results. In this scenario, my bot initially selected the red remote (14), but after implementing dynamic programming, it switched to the blue remotes (14 and 15), which was a more efficient choice.

Stay tuned for more insights in the next part of this series!
