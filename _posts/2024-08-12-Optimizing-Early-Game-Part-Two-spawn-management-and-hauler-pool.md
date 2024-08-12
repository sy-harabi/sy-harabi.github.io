---
layout: single
title: "Optimizing Early Game Economy – Part 2: Spawn Management and Hauler Pool"

toc: true
toc_sticky: true

comments: true
---

Hello everyone! In my previous article, I discussed how to effectively select remotes to optimize the early game economy. Now that we've covered that, it’s time to dive into spawn management and hauler pool optimization. With the right strategy, we can streamline the spawning process and ensure our haulers are operating at peak efficiency.

<img src="https://github.com/user-attachments/assets/38943367-abe4-4c88-9a9a-4b6ced12e785" style="width:100%">

## 1. Spawning Creeps

### A. Spawn Management System Overview

To manage the spawning of creeps efficiently, I've developed a spawn management system that organizes and prioritizes spawn requests. While it may not be the ultimate solution, this system has greatly improved my clarity and coding efficiency.

1. **Queue Initialization:** At the start of the main loop, I iterate through all the rooms I control and check if there is available spawn and prepare a spawn queue if there is. I use a **min-max heap** for this queue, which simplifies the sorting process. You can read more about min-max heaps [here](https://en.wikipedia.org/wiki/Min-max_heap).
  
2. **Requesting Creeps:** During the logic phase for each room, whenever there is available spawn and a new creep is needed, I insert a spawn request into the queue. Each request includes crucial details like the creep’s name, body composition, memory, role, and other necessary information.

3. **Processing the Queue:** After all the logic has been executed, I iterate through all the rooms I control and review the spawn queue and spawn the creeps with the highest priority.


So, my main loop looks like this.

```
for room of myRooms
    spawnManager.resetSpawnQueue(room)

for room of myRooms
    roomManager.run(room) // during run, request creep if there is available spawn and a new creep is needed.
    
for room of myRooms
    spawnManager.run(room)
```


### B. Spawning Order for Earning Creeps

When spawning earning creeps like miners, haulers, and reservers (or source keeper killers), it’s important to prioritize sources strategically:

1. **Close Sources First:** Focus on the closest sources first as they are the most profitable.
2. **Energy Production:** Within each source, prioritize increasing the energy harvested before spawning the necessary haulers.
3. **Sequential Completion:** Complete one source before moving on to the next.

**Detailed Spawning Logic:**

1. **Iterate Through Sources:** Start with the closest or most profitable source.
2. **Check for Special Units:** If a reserver or source keeper killer is needed, spawn them and stop iterating.
3. **Spawn Miners:** If a miner is needed, spawn it and stop iterating.
4. **Spawn Haulers:** If the current haulers are insufficient, spawn additional haulers and stop iterating.
5. **Update Hauler Count:** Reduce the remaining hauler count by the number required for the current source.
6. **Proceed to the Next Source:** Continue to the next source until all are adequately serviced.

<img src="https://github.com/user-attachments/assets/a6159625-8a0e-4bc2-ad96-1ec259b3820b">

(Iterate from source number 1 to source number 9)

## 2. Balancing Earning vs. Using Creeps

Balancing the spawning of earning creeps (miners and haulers) with using creeps (upgraders and builders) is critical, but the strategy differs depending on whether you have storage.

### A. With Storage

When storage is available, the strategy is straightforward:

- **Energy-Based Decision:** The priority between earning and using creeps is based on the energy amount in storage. If the energy in storage exceeds a certain threshold, prioritize spawning using creeps. If it falls below that threshold, focus on earning creeps.

### B. Without Storage

Without storage, the balance requires more delicate management:

- **Adaptive Metric:** I use a metric that adjusts the priority dynamically. This metric ranges from -1 to 1 and naturally decays to zero over time.
  - **Priority Shift:** If haulers are unable to find a place to deposit energy (indicating an overflow), the metric increases, shifting the priority to using creeps.
  - **Energy Shortage:** Conversely, if using creeps are running out of energy, the metric decreases, and the focus shifts back to earning creeps.

This adaptive strategy helps maintain a balance between energy production and consumption when storage is not available.

<img src="https://github.com/user-attachments/assets/9a509e86-3b4e-47da-a55f-9b844ca8c0d9" style="width:100%">

(I visualized this metric using in-game methods.)

## 3. Hauler Pool Management

Once haulers are spawned, the challenge is to manage them efficiently. The goal is to ensure that every hauler's capacity is utilized without wasting travel time or energy.

### A. Strategy Overview

- **Maximize Carry Capacity:** Avoid sending haulers to sources where there won’t be enough energy to fill their capacity.
- **Prioritize Proximity:** Favor closer sources since they yield the same amount of energy with less travel time.

### B. Calculating Expected Energy

To decide where a hauler should go, we need to estimate the energy available at each source, considering other haulers that might be heading there.

1. **Energy Piled Up:** Calculate the sum of energy in containers and any dropped energy near the source.
2. **Expected Energy Gain:** Determine the expected energy gain based on travel distance:
   - **If the distance is less than or equal to the ticks left for regeneration:**

      ```Expected Energy Gain = min(energy piled, harvest power * distance)```
   - **If the distance is greater than the ticks left for regeneration:**

      ```Expected Energy Gain = min(energy piled, harvest power * distance) + (harvest power) * (distance – ticks to regeneration)```


3. **Adjust for Hauler Traffic:** Reduce the expected energy by the amount likely to be taken by other haulers heading to the source.
4. **Select the Optimal Source:** Choose the closest source with enough expected energy.

---

**Conclusion:**

In this article, we've covered the essentials of spawn management and hauler pool optimization. By strategically managing the spawning process and ensuring your haulers are efficiently assigned, you can significantly boost your early game economy. Stay tuned for the next part of this series, where we'll dive into defense techniques!
