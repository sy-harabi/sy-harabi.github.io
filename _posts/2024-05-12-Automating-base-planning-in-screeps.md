---
layout: single
title: "Automating Base Planning in Screeps – A Step-by-Step Guide"

toc: true
toc_sticky: true
---
## Why Base Planning?

Planning the layout of your base in Screeps might seem unusual at first, but it’s a crucial step in automating your bot. It’s also one of the most challenging tasks to tackle manually. While you don't necessarily need to fully understand the algorithms involved, you should at least know how to utilize them effectively. So, here’s a step-by-step guide on how I automate my base planning. Let’s dive into it.

## The Process

1. **Utilize Distance Transform**: Start by employing the distance transform algorithm to identify open spaces in the room. Typically, my bot requires a 5x5 square to begin, so I target positions with a distance of 3 or larger. Some bots, especially those employing bunker-style base planners, may require squares larger than 10x10. Plan ahead for the desired size of your base.
![image](https://drive.google.com/uc?export=view&id=1qGS8pEW4k8g_r9Ey-WOI0qneZ--NdBH7)
2. **Select Starting Position**: Choose an open area close to both the controller and energy sources. Proximity to the controller is critical, especially if significant energy upgrades are anticipated. Distance to energy sources is less crucial since transportation from source to storage is capped at 10 energy per tick.

3. **Place Core Structures**: Begin by placing core structures such as spawn, storage, and terminal. I surround the starting position with core structures using a 5x5 stamp for efficiency. Some bots utilize a stamp called "fast filler" to quickly fill extensions. The choice of design is up to you!

4. **Designate Controller Upgrade Area**: Allocate a 3x3 area near the core structures for upgrading the controller.

5. **Implement Floodfill Algorithm**: Utilize the floodfill algorithm to categorize tiles by their accessibility. Lower numbers indicate proximity to core structures.

6. **Allocate Tile Usage**: Assign tiles for essential structures such as extensions, labs, towers, factory, nuker, and observer. I personally prefer a grid layout similar to the "commie bot."

7. **Position Labs**: Identify 10 suitable tiles meeting specific criteria, ensuring that there are two source labs within range 2 of every other lab.

8. **Establish Infrastructure**: Lay down roads from storage to energy sources and minerals. Place a container and a link at each source, and a container and an extractor at the mineral.

9. **Optimize Ramparts**: Utilize the minimum cut algorithm to determine optimal rampart positions, then connect them with roads.

10. **Strategic Tower Placement**: Position towers to maximize their coverage. Personally, I prioritize maximizing the minimum damage of towers over base boundaries.

11. **Place Additional Structures**: Situate the factory, nuker, observer, and extensions. Placement for the observer is less critical, so anywhere works.

## Examples and Visuals

[Include relevant examples or images here.]

Automating base planning streamlines your development process and lays a solid foundation for your Screeps bot to thrive.

