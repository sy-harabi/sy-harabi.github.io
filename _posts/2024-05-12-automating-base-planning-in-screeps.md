---
layout: post
title: "Automating Base Planning in Screeps"
---
Planning the layout of your base in Screeps might seem odd at first, but it’s a crucial step in optimizing your energy economy and overall bot efficiency. Let’s delve into why base planning is essential and how you can automate the process.

## Why Base Planning?

1. **Understanding the Blueprint**: Before diving into coding your bot’s energy management, it’s essential to visualize what your base will look like. Base planning provides this blueprint.

2. **Modularity**: Base planning is a standalone module, separate from your bot's core functionality. This independence allows for easier modifications and updates without affecting the entire bot.

3. **Organizational Benefits**: If your current base planning code resembles a tangled mess (like mine did), automating it can bring much-needed clarity and organization to your project.

## The Process

1. **Utilize Distance Transform**: Start by using the distance transform algorithm to identify open spaces in the room. Typically, my bot requires a 5x5 square to commence.

2. **Select Starting Position**: Opt for an open area close to both the controller and energy sources. Proximity to the controller is crucial, especially if you anticipate significant energy upgrades.

3. **Place Core Structures**: Surround the starting position with core structures using a 5x5 stamp for efficiency.

4. **Designate Controller Upgrade Area**: Allocate a 3x3 area near the core structures for upgrading the controller.

5. **Implement Floodfill Algorithm**: Employ the floodfill algorithm to categorize tiles by their accessibility. Lower numbers signify proximity to core structures.

6. **Allocate Tile Usage**: Choose 79 tiles for essential structures such as extensions, labs, towers, factory, nuker, and observer. I personally prefer a grid layout similar to the commie bot.

7. **Position Labs**: Identify 10 suitable tiles meeting specific criteria, ensuring each lab is within range 2 of every other lab.

8. **Establish Infrastructure**: Lay down roads from core structures to energy sources and minerals. Place containers, links, and an extractor strategically.

9. **Optimize Ramparts**: Utilize the minimum cut algorithm to determine optimal rampart positions, then connect them with roads.

10. **Strategic Tower Placement**: Position towers to maximize their coverage and minimize damage to base boundaries (ramparts).

11. **Place Additional Structures**: Situate the factory, nuker, observer, and extensions. For the observer, placement is less critical, so anywhere works.

## Examples and Visuals

[Include relevant examples or images here.]

By automating base planning, you streamline your development process and lay a solid
