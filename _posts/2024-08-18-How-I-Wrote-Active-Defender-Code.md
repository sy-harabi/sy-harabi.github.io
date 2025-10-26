---
layout: single
title: "How I Wrote Active Defender Code"

toc: true
toc_sticky: true

comments: true
---
## 1. Today's Subject

In the past few posts, I've primarily focused on the economy. A robust economy enables rapid growth, but to thrive, you must also defend your rooms effectively. This time, let's delve into room defense.

Basic room defense comprises three main components: strong enough towers, repairers, and active defenders. Among these, active defenders are the most challenging to manage. Today, I'll break down the process of implementing active defenders, focusing primarily on melee defenders.

To effectively use active defenders, we need to answer three key questions:

1. How much damage do we need?
2. How should we assign or spawn active defenders?
3. How can we move them smoothly?

Let’s tackle each of these questions one by one.

## 2. How Much Damage Do We Need?

The first question we need to answer is, "How many active defenders do we need?" To do this, we must first determine how much damage is necessary to deal effective damage to each enemy creep and how to group enemies.

### 2.1. Estimating Tower Damage

The first step is to understand how much damage we can expect from our towers. I use the minimum tower damage for all the outermost ramparts, as this is the most conservative estimate for enemy creeps. 

You can calculate this using `floodFill`. If you're unfamiliar with it, refer to the 5th paragraph of the "The Process" section in [this post](https://sy-harabi.github.io/Automating-base-planning-in-screeps/#the-process). Start from the room exits, collect the outermost ramparts, and then determine the minimum tower damage among them. This will serve as your tower damage expectation.

### 2.2. Calculating Required Damage

To calculate the required damage, I wrote a script based on the Screeps engine code, which you can view [here](https://github.com/sy-harabi/screeps-utils/blob/017105e9540ae5bfde34b0e307da1a588b85e9fb/utils.js#L7). The key point is that you must deal more than 100 damage to destroy one boosted tough part while considering the heal power of enemy creeps. Here’s how you can approach this:

1. **Calculate Total Heal:** For each enemy creep, calculate the total healing potential (including both regular and ranged heal). Let’s call this the total heal for that creep.

2. **Calculate Required Damage:** For each enemy creep, determine how much damage is needed to overcome a set amount of damage plus the total heal for that creep. I typically use 100 as the base amount, though this value is somewhat arbitrary.

3. **Adjust for Tower Damage:** Subtract the expected tower damage from the required damage calculated in step 2.

With this, we have determined the additional damage required beyond what the towers can deliver. We’ll use this information to decide how many active defenders to spawn later.

## 3. How Should We Assign/Spawn Active Defenders?

Spawning active defenders for each enemy creep is inefficient because enemies usually form groups like duos or quads. To manage this, we need to group them effectively. Here’s how I solved this problem:

### 3.1. Grouping Enemy Creeps

1. **Flow Field Setup:** Start by generating a flow field from our outermost ramparts (collected in step 2.1). This involves running a full Dijkstra and caching the results. This method allows us to quickly determine the closest outermost rampart for each enemy without performing costly pathfinding for every enemy.
  ![FloodFill Example](https://github.com/user-attachments/assets/cb17acb0-4e64-4d73-b78f-507d0d05e238)

1. **Initialize a Damage Map:** Create an empty object to store the expected damage for each outermost rampart position. You can use a two-dimensional object or calculate keys using the formula `50*y+x`.

1. **Assign Enemies to Ramparts:** For each enemy, determine the closest rampart (based on path length) using the flow field. Minimize diagonal moves, as these are less effective for blocking enemies from reaching the ramparts.
  ![Finding Closest Rampart](https://github.com/user-attachments/assets/dad9f6b4-d5ca-47bf-8947-9aa53e5ccf0a)
  This is good enough
  ![Prefer Straight Paths](https://github.com/user-attachments/assets/888e5b15-3c6f-43a0-8ad5-6e3d3c81f519)
  But this is better

1. **Collect Adjacent Ramparts:** Gather all ramparts adjacent to the closest one (using Manhattan distance). Typically, this involves three ramparts. For each position, add the enemy creep's attack power to the damage map.

1. **Group Enemies:** Identify the outermost rampart position with the highest expected damage. Group all the enemy creeps that contributed to this rampart. Repeat steps 3.1.3 through 3.1.4 until no enemy creeps remain ungrouped.

After grouping, you should see something like this:

![Grouping Enemies](https://github.com/user-attachments/assets/4350c389-f83c-4d84-8651-0b176af89366)

### 3.2. Assigning and Spawning Defenders

For each group, find the creep with the highest required damage calculated in step 2.2. Assign active defenders until their combined damage surpasses this requirement. If there aren’t enough defenders, spawn more. This method ensures that you efficiently assign and spawn enough active defenders.

## 4. How Should We Move Active Defenders Smoothly?

The final challenge is moving and allocating your defenders effectively. Movement is a complex aspect of Screeps, but I’ve found a solution that works well. Check out [this video](https://github.com/user-attachments/assets/fb17d13c-21de-4198-a15b-5dc030221dce) for a demonstration. Pretty elegant, right?

### 4.1. Movement Strategy

1. **Identify Key Positions:** For each enemy group, find the enemy closest to the outermost ramparts. Use the cached rampart positions to determine where your creeps should stand.

2. **Move Creeps:** The cyan squares in the video represent the positions defenders should occupy. Here’s how to move them:

    a. **Approach the Tiles:** For each creep more than one tile away from all the desired tiles, move it closer until it’s adjacent to one of the desired tiles.

    b. **Pathfinding:** For creeps adjacent to one of the desired tiles, find a path to an empty tile using only other defenders.

    c. **Chain Movement:** If a path is found, move each creep one step forward along the path, ultimately positioning the first creep inside the desired area.

This method allows you to allocate your defender creeps smoothly and effectively.

## Conclusion

We’ve covered how to manage active defenders in Screeps, from calculating damage needs to moving them efficiently. With these strategies, your established rooms should be well-defended against most threats. If you have any questions, feel free to ping me on the official Discord or leave a comment on this post.

Thank you for reading!
