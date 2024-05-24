---
layout: single
title: "Journey to Solving the Traffic Management Problem"

toc: true
toc_sticky: true

comments: true
---
## What is traffic management in Screeps?
Movement is one of the significant challenges in Screeps. In fact, the official Discord has a dedicated pathfinding channel. Traffic management, a critical aspect of movement control, refers to the efficient control and movement of creeps to prevent congestion and collisions. Without proper management, creeps can get stuck or move slowly, severely hindering their efficiency.
![1  Traffic](https://github.com/sy-harabi/screeps_harabi/assets/71678452/c84e2ae4-bd76-4e51-ae51-67eae6e6e6dd)
## Common methods to manage traffic
The most intuitive solution is swapping and pushing. This means if a creep is blocking another creep's path, you swap their positions or push the blocking creep out of the way. While effective in many cases, this method can fail in crowded areas, creating edge cases. Advanced techniques to handle these scenarios include prioritizing creeps and recursive shoving, which involves pushing a chain of creeps until one is moved to an empty tile.
## What’s different about my solution?
I found the common methods unsatisfactory due to the numerous edge cases they created, especially in extreme situations. Each new edge case required additional handling, complicating the solution. I aimed to develop a general solution to this problem. Through algorithmic research, I discovered that the Bipartite Matching Problem (BMP) could help solve this issue. Essentially, traffic management involves assigning each creep to a specific position for the next tick. By tweaking the Ford-Fulkerson method, I successfully addressed the traffic management problem.
## Basic Idea
This solution leverages graph theory. Imagine two sets of vertices: one representing creeps and the other representing room positions. We connect these vertices with edges, indicating a creep's potential move or stay. There are two types of creeps: those intending to move to an adjacent tile and those without such an intent. Our goal is to maximize the number of fulfilled move intents.
## How the algorithm works
We start by connecting each creep to its current position with edges. For each creep with a move intent, we search for augmenting paths in our graph. If a path increases the number of fulfilled intents, we send flow along that path. We use depth-first search (DFS) to explore these paths, scoring each path as follows: +1 if it connects a creep to its intended position, and -1 if it cancels an existing connection. After iterating through all creeps, the results indicate where each creep should be in the next tick.
## Pseudocode
```
Declare movementMap
Declare visitedCreeps

Function run(room):
        For each creep in creepsInRoom:
            Match creep to its current coordinate
            If creep has intended move, add to creepsWithMovementIntent

        For each creep in creepsWithMovementIntent:
            If creep is matched to its intended coordinates, continue
            Clear visitedCreeps
            Remove creep from movementMap
            If dfs(creep) > 0, continue
            Reassign creep to its current coordinate

        For each creep in creepsInRoom:
            Move creep towards matched coordinate

Function dfs(creep, score = 0):
    Mark creep as visited
    For each possible position:
        If creep is matched to its intended coordinate, increase score
        If position is unoccupied:
            If score > 0, match creep to position
            Return score
        If position is occupied by anotherCreep and anotherCreep is not visited
            If dfs(anotherCreep, score) > 0:
                match creep to position
                return the result of dfs
    Return -Infinity


Function getPossiblePositions(creep):
    Initialize possiblePositions with current position
    If intended coordinate exists:
        add to possiblePositions
        return possiblePositions
    Add valid adjacent positions to possiblePositions
    Return possiblePositions
```

