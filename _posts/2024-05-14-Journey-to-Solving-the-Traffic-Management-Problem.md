---
layout: single
title: "Automating Base Planning in Screeps – A Step-by-Step Guide"

toc: true
toc_sticky: true

comments: true
---
Journey to Solving Traffic Management Problem

What is traffic management in screeps?
Movement is one of the big issues in screeps. Official discord has pathfinding channel. traffic management is one of the part of movement control, refers to the efficient control and movement of creeps to prevent congestion and collisions, which makes creeps to stuck each other and make them completely stop or slow.

Common methods to manage traffic.
Common and most intuitive solution would be swapping and pushing, meaning that if a creep is on another creep’s path, just swap their position or push that creep out of the path. This kind of solution works well in most cases, but can make edge cases, specially when there are many creeps in small area. More advanced techniques to avoid these edge cases includes prioritizing creeps and so called recursive shoving, which pushes chain of creeps until it pushes a creep to an empty tile.

What’s the difference that my solution has?
I had dissatisfaction with common methods since they made tons of edge cases especially in extreme cases. Every time I met edge cases, I should add some features to handle those edge cases. So I wanted to make general solution to this problem. With some algorithmic research, I concluded that Bipartite Matching Problem, shortly BMP, can help me to solve this problem since traffic management is basically assigning each creep to a room position that they will be stand on next tick. With some tweak to Ford-Fulkerson method, I successfully solved traffic management problem.

Basic Idea
So we’re in the world of graph theory. Assume that we have two set of vertices, one is set of creeps and one is set of room positions. We’ll connect them with edges, which mean a creep will move to or stay that position. There are two kind of creeps. One is creeps which have move intent to an adjacent tile, and the other one is creeps which don’t have move intent. Our goal is to maximize the number of fulfilled intents.

How the algorithm works
We start with edges that connects each creep to its current position. Iterate through all the creeps which has move intent. Search and check all the augment paths. (Not a screeps path. Path in our graph) if that path can increase the number of fulfilled intents, we’ll send flow along the paths. We’ll using dfs to search paths and check one path’s score as : +1 when it makes edge which connects creep to its intended position, -1 when it cancels edge which connects creep to its intended position. After iteration ends, result tells us what position each creeps should be at the next tick.

PseudoCode

Code Example && Link for Module

Revised version

Journey to Solving the Traffic Management Problem
What is traffic management in Screeps?
Movement is one of the significant challenges in Screeps. In fact, the official Discord has a dedicated pathfinding channel. Traffic management, a critical aspect of movement control, refers to the efficient control and movement of creeps to prevent congestion and collisions. Without proper management, creeps can get stuck or move slowly, severely hindering their efficiency.
Common methods to manage traffic
The most intuitive solution is swapping and pushing. This means if a creep is blocking another creep's path, you swap their positions or push the blocking creep out of the way. While effective in many cases, this method can fail in crowded areas, creating edge cases. Advanced techniques to handle these scenarios include prioritizing creeps and recursive shoving, which involves pushing a chain of creeps until one is moved to an empty tile.
What’s different about my solution?
I found the common methods unsatisfactory due to the numerous edge cases they created, especially in extreme situations. Each new edge case required additional handling, complicating the solution. I aimed to develop a general solution to this problem. Through algorithmic research, I discovered that the Bipartite Matching Problem (BMP) could help solve this issue. Essentially, traffic management involves assigning each creep to a specific position for the next tick. By tweaking the Ford-Fulkerson method, I successfully addressed the traffic management problem.
Basic Idea
This solution leverages graph theory. Imagine two sets of vertices: one representing creeps and the other representing room positions. We connect these vertices with edges, indicating a creep's potential move or stay. There are two types of creeps: those intending to move to an adjacent tile and those without such an intent. Our goal is to maximize the number of fulfilled move intents.
How the algorithm works
We start by connecting each creep to its current position with edges. For each creep with a move intent, we search for augmenting paths in our graph. If a path increases the number of fulfilled intents, we send flow along that path. We use depth-first search (DFS) to explore these paths, scoring each path as follows: +1 if it connects a creep to its intended position, and -1 if it cancels an existing connection. After iterating through all creeps, the results indicate where each creep should be in the next tick.
Pseudocode
Code Example & Module Link
[Provide your code example and a link to your module here]
Markdown

# Journey to Solving the Traffic Management Problem
## What is traffic management in Screeps?
Movement is one of the significant challenges in Screeps. In fact, the official Discord has a dedicated pathfinding channel. Traffic management, a critical aspect of movement control, refers to the efficient control and movement of creeps to prevent congestion and collisions. Without proper management, creeps can get stuck or move slowly, severely hindering their efficiency.
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

